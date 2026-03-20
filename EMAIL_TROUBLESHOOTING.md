# Email Delivery Troubleshooting Guide

## Issue

Emails sent to `support@github.com` are being blocked. The bounce-back message references [Google Support Article 172179](https://support.google.com/a/answer/172179), which relates to email authentication failures.

## Cause

Google (which handles GitHub's email) rejects incoming messages when the sender's domain fails email authentication checks. These checks verify that the email actually comes from the domain it claims to come from, and include:

- **SPF (Sender Policy Framework)** — Specifies which mail servers are authorized to send email on behalf of your domain.
- **DKIM (DomainKeys Identified Mail)** — Adds a digital signature to verify the email was not altered in transit.
- **DMARC (Domain-based Message Authentication, Reporting, and Conformance)** — Tells receiving servers what to do when SPF or DKIM checks fail.

If any of these records are missing or misconfigured in your domain's DNS, Google may block the email.

## How to Fix

### 1. Set Up SPF

Add a single TXT record to your domain's DNS:

```
Type:  TXT
Host:  @
Value: v=spf1 include:_spf.google.com ~all
```

> **Note:** Replace or extend the `include:` value to match your email provider. For example, if you use Outlook/Microsoft 365, use `include:spf.protection.outlook.com`. You can include multiple providers in one record.

**Common SPF mistakes to avoid:**
- Having more than one SPF record (there must be exactly one).
- Exceeding the 10 DNS lookup limit.
- Typos or syntax errors in the record.

### 2. Set Up DKIM

DKIM setup depends on your email provider. General steps:

1. Generate a DKIM key pair through your email provider's admin console.
2. Publish the public key as a TXT record in your DNS. Your provider will give you the exact record name and value.
3. Enable DKIM signing in your email provider settings.

**Common DKIM mistakes to avoid:**
- Incorrect or truncated DNS TXT record value.
- Mismatched DKIM selectors between provider settings and DNS.
- Not waiting for DNS propagation (allow 24–48 hours).

### 3. Set Up DMARC

Add a TXT record to your domain's DNS:

```
Type:  TXT
Host:  _dmarc
Value: v=DMARC1; p=none; rua=mailto:your-email@yourdomain.com
```

> **Tip:** Start with `p=none` (monitoring only) and move to `p=quarantine` or `p=reject` after confirming that SPF and DKIM are passing for all legitimate mail sources.

### 4. Verify Your Records

Use free tools to check that your DNS records are correct and propagated:

- [Google Admin Toolbox — Check MX](https://toolbox.googleapps.com/apps/checkmx/)
- [MXToolbox](https://mxtoolbox.com/)
- [DMARC Analyzer](https://www.dmarcanalyzer.com/)

### 5. Wait and Re-test

DNS changes can take up to 48 hours to propagate globally. After making changes, wait and then resend your email to `support@github.com`.

## References

- [Google Support — Troubleshoot SPF Issues (Article 172179)](https://support.google.com/a/answer/172179)
- [Google Workspace — Set Up SPF](https://support.google.com/a/answer/33786)
- [Google Workspace — Set Up DKIM](https://support.google.com/a/answer/174124)
- [Google Workspace — Set Up DMARC](https://support.google.com/a/answer/2466580)
