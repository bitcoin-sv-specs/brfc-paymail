---
brfc: true
title: Simplified Payment Protocol (BIP 270) Paymail Integration
authors:
  - Ryan X. Charles (Money Button)
version: 1
---
# Simplified Payment Protocol (BIP 270) Paymail Integration

{{yfm}}

Hash: 34440c4668b3

This capability allows delivering Simplified Payment Protocol (BIP 270) endpoints. Also included is an extension to the PayTo protocol to support BIP 270.

## Motivation

It is important for scalability to move away from "pay-to-address", where the recipient of a transaction (or someone on their behalf) is expected to scan the blockchain to find their transaction, to "pay-to-paymail", where a transaction is delivered directly to the recipient (or someone on their behalf), eliminating the need to scan the blockchain to find the received transaction.

Simplified Payment Protocol (BIP 270) is a fork of the original Payment Protocol (BIP 70) and updated to be easier to implement while retaining the key desirable features.

The features of BIP 270 are:

- The recipient specifies a payment request that can include multiple outputs, arbitrary scripts, values, and fee rate information
- The sender gives the transaction to the endpoint specified by the payment request (and does not broadcast it to miners)
- The recipient broadcasts the transaction to miners
- JSON is used as the data format instead of protocol buffers
- x.509 certificates are removed; it is expected that authentication happens at a different layer (usually HTTPS)

While it is expected that most implementations of BIP 270 are also paired with paymail, that is not a requirement of BIP 270 itself. The specifications relevant to implementing BIP 270 independently of paymail are available here:

- https://github.com/moneybutton/bips/blob/master/bip-0270.mediawiki
- https://github.com/moneybutton/bips/blob/master/bip-0271.mediawiki
- https://github.com/moneybutton/bips/blob/master/bip-0272.mediawiki
- https://github.com/moneybutton/bips/blob/master/bip-0273.mediawiki
- https://github.com/moneybutton/bips/blob/master/bip-0274.mediawiki

The strategy of merging paymail and BIP 270 is sometimes referred to as "modern pay-to-IP", because it is similar to the pay-to-IP feature built into the original Bitcoin source code, but without the man-in-the-middle (MITM) attack issue and with some additional features like script templates and multiple outputs.

## Flow

The usual flow will be one of two possibilities. Either the sender wishes to send a payment of a certain amount to the recipient (such as in many real-life payment scenarios where no explicit payment request is provided, or where the recipient is the sender using a different wallet), or the recipient has already generated a payment request and the sender wishes to pay the payment request.

In the first case, the sender knows the paymail of the recipient and the amount they wish to send. The sender will query the paymail endpoint for the BIP 270 endpoint. Next, the sender's wallet software will query the BIP 270 paymail endpoint with a query parameter for the amount. The sender will receive a payment request which they will then pay by building, signing, and sending a transaction to the <code>paymentUrl</code> specified in the payment request (following the BIP 270 protocol).

In the second case, the sender has received a payment request ID embedded in a URL or in a QR code using the PayTo protocol with an extra `{?prid}` URI parameter. The sender either clicks the URL or scans the QR code, which will activate their wallet software. The wallet software will query the paymail endpoint to find the BIP 270 endpoint. The wallet software will then query the BIP 270 paymail endpoint with a query parameter for the payment request ID. The sender will then pay the payment request by building, signing, and sending a transaction to the <code>paymentUrl</code> specified in the payment request (following the BIP 270 protocol).

## Capability discovery

The `.well-known/bsvalias` document is updated to include a BIP 270 endpoint:

```json
{
  "bsvalias": "1.0",
  "capabilities": {
    "{{fm:brfc}}": "https://example.bsvalias.tld/api/{alias}@{domain.tld}/{?amount,purpose,prid}"
  }
}
```

The `capabilities.{{fm:brfc}}` is a template URL for where to retrieve payment requests.

## Client Request

The `capabilities.{{fm:brfc}}` path returns a URI template. Senders _MUST_ replace `{alias}`, `{domain.tld}` placeholders with a valid paymail handle.

The `{?amount}` query parameter _MAY_ be any valid number of satoshis representing the amount to be specified in the output(s) of the payment request to be returned by the server. This is the equivalent of the statement "please give me an invoice for X amount".

The `{?purpose}` query parameter _MAY_ be any URL-encoded string, including the empty string, representing the purpose of the payment. This value _MAY_ be used by the recipient to describe the payment.

The `{?prid}` query parameter _MAY_ be any URL-encoded string, including the empty string, representing the ID of a payment request to be retrieved and returned. The recipient _MUST_ deliver the same payment request each time the same payment request is requested.

Note that both `{amount}` and `{prid}` are optional, but the sender _MUST_ specify at least one of them.

## Server Response

The server will respond to the request with a BIP 270 payment request.

The server _MUST_ use the MIME types specified by BIP 271. For instance, the MIME type for the initial payment request is given by:

<pre>
Content-Type: application/json
</pre>

## PayTo Protocol Extension

The PayTo protocol is useful for QR codes or URLs on web pages. The PayTo protocol looks like this:

```
payto:{receiver}{?amount,purpose}
```

We extend the PayTo protocol with one optional addition, the invoice ID:

```
payto:{receiver}{?amount,purpose,invoice}
```

| Token | Required | Description |
|-|-|-|
| `receiver` | ✓ | `<alias>@<domain>.<tld>` formatted paymail handle, for example `payments@example.org` |
| `amount` | | Optional integer number of satoshis to be paid to be placed in the client request URI. |
| `purpose` | | Optional human-readable description of the purpose of the payment  |
| `invoice` | | Optional invoice ID to be placed in the client request URI |

Note that every single one of these parameters have meaning in the BIP 270 paymail integration.
