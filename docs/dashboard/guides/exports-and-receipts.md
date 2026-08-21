# Exports and receipts

Two ways to get data out of the dashboard: a **CSV** for a spreadsheet, and a
**receipt** for a single payment.

## CSV export

Export a transaction list to CSV from the Transactions page. The file downloads
straight to your machine.

### It opens correctly in Excel

Small details, deliberately handled:

**Numbers are numbers.** Amounts are written unquoted, so your spreadsheet reads
them as numeric and `SUM` works without cleaning the column first.

**Commas in text cannot break the file.** Every text field is quoted and inner
quotes are doubled, per RFC 4180. A recipient name containing a comma, a quote,
or a newline stays in its own column instead of shifting everything to the right.

**Accents survive.** The file carries a UTF-8 byte order mark, which is what
Excel needs to read it as UTF-8 rather than guessing at a legacy encoding.
Without it, "Nguyễn" arrives mangled.

### Working with the export

Amounts are in the currency shown in the column header. Timestamps are UTC.

If you are reconciling against your own ledger, join on the **recipient
reference** or the **transaction ID** — both are in the export and both are
stable.

## Receipts

Open a payment and download a receipt: a plain-text document with the payment's
details, suitable for filing, emailing, or passing to a customer.

It covers what was sent, what the recipient received, the corridor, the reference,
the state, and the timestamps.

## Printing

Printable views are laid out for paper — proper page breaks, and table headers
repeated on each page, so a long list stays readable after page one.

Use your browser's print dialog. "Save as PDF" from there produces a clean file.

## Demo artefacts are marked

Anything generated in [demo mode](../getting-started/demo-mode.md) says so, in
the document and in the filename.

A demo receipt carries an unmistakable banner. A demo export is named as a demo
export.

!!! info

    This is not adjustable, and that is the point. A document that could pass for a
    record of a real payment is a document that will eventually be mistaken for one —
    in an audit, an expense claim, or a customer dispute.

## What exports are not

An export is a **snapshot** of what the dashboard showed at that moment.

For payments still in flight, the state in the file is the state at export time.
For anything that has to be exactly right, re-export once the payments have
reached **Reconciled** — that is when the final amounts are settled.

Exports are also not a substitute for your own books. They are a convenience for
reading and reconciling, not a system of record.
