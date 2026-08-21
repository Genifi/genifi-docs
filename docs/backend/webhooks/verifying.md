# Verifying signatures

Every delivery is signed. Verify it before you trust it — an unverified webhook
endpoint is a public API that lets anyone tell your system that a payment
settled.

## The construction

```
signature = "sha256=" + HMAC_SHA256(secret, timestamp + "." + rawBody)
```

* `secret` — the `whsec_…` value returned when you registered the endpoint
* `timestamp` — the `X-Genifi-Timestamp` header, Unix **seconds**
* `rawBody` — the exact bytes of the request body, before parsing
* Result — lowercase hex, prefixed `sha256=`, sent as `X-Genifi-Signature`

## Why the timestamp is inside the signature

A signature over the body alone proves only that Genifi produced that payload at
*some* point — not that we produced it *now*.

Anyone who captures one delivery (a logging proxy, a misconfigured TLS
terminator, a compromised intermediary) could resend those exact bytes with that
exact signature forever, and a verifier checking only the body could not tell the
difference. For transaction events, that is a way to make your system believe a
payment settled repeatedly.

Binding the timestamp into the signed message means the timestamp cannot be
edited without invalidating the signature — which is what lets you reject
anything older than your tolerance window. It is the same construction Stripe and
GitHub use, for the same reason.

## Verify in three steps

1. **Recompute** the HMAC from the raw body and the header timestamp.
2. **Compare in constant time.** A plain `===` leaks, through timing, how many
   leading characters matched — enough to forge a signature byte by byte given
   enough attempts.
3. **Check the timestamp is recent.** Five minutes is a reasonable tolerance.

Step 3 is not optional. Steps 1 and 2 alone accept an infinitely-replayable
capture.

## Node.js

```ts
import { createHmac, timingSafeEqual } from "node:crypto";

const TOLERANCE_SECONDS = 300;

export function verify(
  rawBody: string,
  signatureHeader: string,
  timestampHeader: string,
  secret: string,
): boolean {
  const timestamp = Number(timestampHeader);
  if (!Number.isFinite(timestamp)) return false;

  // Reject anything outside the tolerance window, in both directions.
  const age = Math.abs(Math.floor(Date.now() / 1000) - timestamp);
  if (age > TOLERANCE_SECONDS) return false;

  const expected =
    "sha256=" +
    createHmac("sha256", secret)
      .update(`${timestamp}.${rawBody}`, "utf8")
      .digest("hex");

  const a = Buffer.from(signatureHeader, "utf8");
  const b = Buffer.from(expected, "utf8");
  return a.length === b.length && timingSafeEqual(a, b);
}
```

### With Express

```ts
import express from "express";

const app = express();

// The RAW body, not the parsed one. Re-serializing JSON changes bytes
// and breaks verification.
app.post(
  "/genifi",
  express.raw({ type: "application/json" }),
  (req, res) => {
    const raw = req.body.toString("utf8");

    const ok = verify(
      raw,
      String(req.header("x-genifi-signature")),
      String(req.header("x-genifi-timestamp")),
      process.env.GENIFI_WEBHOOK_SECRET!,
    );
    if (!ok) return res.sendStatus(401);

    const event = JSON.parse(raw);

    if (alreadyProcessed(event.deliveryId)) return res.sendStatus(200);
    recordDelivery(event.deliveryId);

    res.sendStatus(200);        // acknowledge first
    void handleAsync(event);    // then do the work
  },
);
```

## Python

```python
import hmac, hashlib, time

TOLERANCE_SECONDS = 300

def verify(raw_body: bytes, signature: str, timestamp: str, secret: str) -> bool:
    try:
        ts = int(timestamp)
    except (TypeError, ValueError):
        return False

    if abs(int(time.time()) - ts) > TOLERANCE_SECONDS:
        return False

    expected = "sha256=" + hmac.new(
        secret.encode(),
        f"{ts}.".encode() + raw_body,
        hashlib.sha256,
    ).hexdigest()

    return hmac.compare_digest(expected, signature)
```

## Go

```go
func Verify(rawBody []byte, signature, timestamp, secret string) bool {
	ts, err := strconv.ParseInt(timestamp, 10, 64)
	if err != nil {
		return false
	}
	if age := time.Now().Unix() - ts; age > 300 || age < -300 {
		return false
	}

	mac := hmac.New(sha256.New, []byte(secret))
	mac.Write([]byte(strconv.FormatInt(ts, 10) + "."))
	mac.Write(rawBody)
	expected := "sha256=" + hex.EncodeToString(mac.Sum(nil))

	return hmac.Equal([]byte(expected), []byte(signature))
}
```

## Common mistakes

**Verifying the parsed body.** Your JSON parser will re-serialize with different
key order or whitespace, and the bytes will not match. Capture the raw body
before any middleware touches it.

**Skipping the timestamp check.** Signature-valid and *fresh* are different
properties. Without the freshness check, a captured delivery replays forever.

**Comparing with `===`.** Use a constant-time comparison. Every language in this
page has one built in.

**Reusing a stale secret.** Re-registering an endpoint mints a fresh secret and
kills the old one. If verification suddenly fails everywhere, check that first.

**Rejecting duplicates as invalid.** A repeated `deliveryId` is expected
behaviour, not an attack. Return `200` and move on — see
[Webhooks](README.md#delivery-guarantees).

## Rotating the secret

Register the same URL again:

```bash
curl -X PUT "$GENIFI_API/webhooks/endpoint" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "url": "https://hooks.northwind.example/genifi" }'
```

A fresh secret comes back and the old one stops working immediately. There is no
overlap window, so deploy a receiver that accepts either secret before you
rotate, then drop the old one afterwards.
