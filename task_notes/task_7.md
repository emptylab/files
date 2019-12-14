# One Time Password Approaches

Based on my current, limited reading, TOTP wins because each one-time password is time-limited by definion.

Comparison of HOTP with TOTP

- See https://crypto.stackexchange.com/a/2221/75183

HMAC-based One-Time Password (HOTP)

- See https://tools.ietf.org/html/rfc4226

Time-based One-Time Password (TOTP)

- See https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm
- See also https://tools.ietf.org/html/rfc6238

# SMTP Email Provider

Proton Mail

- (+) Keep all email-related MyLocalFarm services under one provider.
- (-) Might require installing Bridge on Ubuntu Server.
- See https://protonmail.com/support/knowledge-base/imap-smtp-and-pop3-setup/

SendGrid

- (+) Supports 100 emails/day for free.
- See https://sendgrid.com/pricing/

Amazon SES

- (-) Need to use Amazon.

Mailgun

- (-) Has "gun" in its name.
- (+) Supports 10,000 emails/month for free.
- (+) Specializes in "transactional emails", which are all that we need for log in.
- See https://www.mailgun.com/pricing/

Self-Hosted Email Server

- See https://news.ycombinator.com/item?id=14201562

# OpenID Connect

https://openid.net/specs/openid-connect-core-1_0.html

# API Contract

**Log In - Initiate** Sends an email with a one-time password link.

```
POST ~/api/login
{ email }
```

**Log In - Complete** Gets an `access_token` and `id_tokden` from the server.

```
GET ~/api/login/${one-time-password}
```

To log out, the browser clears the `access_token` and `id_token` from browser storage.

# Notes from our chat conversation

We have had conversations about this in Telegram starting around 06 December. These notes capture some of the ideas from the conversation.

## General Ideas

- Hash the email.
- Use a one-time password that's associated with the email.
- The link could include 1. the encrypted email and 2. the one-time password.

## Authentication Alternatives

Email only, receive a link at the email (link acts as the password)

- (+) No database to support
- (+) Hard code the email addrses
- (+) Probably easier for end-users (no password)

## Email Storage Alternatives

Store authorizated email in a database.

- (-) Requirement to support a database.
- (-) Requirement to support a backup.

Store authorizated email in a text file that's included in deployment but not included in the code base.

- (-) Requirement to persist the text file across depeloyments.
- (-) Requirement to support a backup.

Hard code authorized emails in the code base.

- (-) Privacy leak because we are an open source repository.

Hard code a hash of authorized emails in the code base.

- (+) Less risk of a privacy breach (though still possible to brute force the hash).
- (+) Easiest approach for the prototype; we can adapt this to more flexible and private storage later.

## Client-side Session Storage Alternatives

Store user session in a coookie.

- (-) Possible legal requirement to have a GDPR warning.

Store user session in a local storage.

Store user session in a session storage.
