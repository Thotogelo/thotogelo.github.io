---
layout: post
title:  "The reversibility problem: converting telecom MSISDNs from 16 digits to 10"
date:   2026-03-21 21:06:21 +0200
categories: telecom data privacy
---

A number can look like a simple string of digits. In a telecom system, it is also an address used to reach a subscriber. That makes changing how it is represented more consequential than a routine database cleanup.

Suppose a system stores a 16-digit value and a new interface expects a 10-digit number. Can we convert the old value and later recover it? The answer depends on what those 16 digits mean. If the conversion drops information, it is not reversible by itself. If the conversion is meant to hide the subscriber's number, it should not be treated as a privacy control unless it is designed and protected as one.

## First, what do the digits represent?

An MSISDN is the public number used to address a mobile subscriber. It is distinct from identifiers such as the IMSI, which identifies a mobile subscription inside network systems. The international form follows the E.164 numbering plan, which allows at most 15 digits, excluding the leading `+` sign. So a 16-digit value should not be assumed to be a globally dialable MSISDN: it may include a country or system prefix, a trunk digit, a routing value, padding, or another identifier altogether. Confirm the source field's definition before transforming it. ([ITU-T E.164](https://www.itu.int/rec/T-REC-E.164))

Ten digits can be a valid national format in a particular country, but it is not a universal format. For example, South Africa's national numbers are generally ten digits; the international `+27` form omits the national trunk `0`. Those are two representations of a number under a known numbering plan, not a rule that any long digit string can safely be shortened to ten digits. ([ICASA numbering](https://www.icasa.org.za/pages/numbering))

## Where reversibility breaks

If the conversion is simply “keep the last ten digits,” it discards the preceding six. Those digits may distinguish countries, number ranges, networks, or internal namespaces. Two different source values can therefore produce the same ten-digit output. Once that happens, the output alone cannot tell you which source value it came from.

Even a conversion that appears reversible on today's sample can fail later. A leading zero may be dropped if a number is stored as an integer instead of text. A country code may be removed without recording which country it belonged to. A prefix rule may change, or a number may be ported to another operator. Formatting the value back to 16 digits with zero-padding does not restore missing information; it only creates a string of the expected length.

This is the reversibility problem: a many-to-one transformation has no unique inverse. You can only recover the original reliably if you retained the discarded information, or if the transformation itself is an explicitly reversible encoding with a protected key and a documented format.

## Risks of getting the migration wrong

- **Misdelivery and service failures.** A collision or incorrect reconstruction can send calls, messages, or verification codes to the wrong destination, or make a valid number unreachable.
- **Account takeover and privacy harm.** Phone numbers are often used for password recovery and one-time codes. A mistaken mapping can expose another person's messages or give access to an account. Numbers can also be reassigned over time, so a once-correct mapping may become stale.
- **Broken joins and duplicate identities.** Customer records, billing, consent, fraud checks, and call-detail records may stop matching—or, worse, match the wrong subscriber—when systems use different formats.
- **Weak anonymisation.** Truncating or masking a number does not make it anonymous. A ten-digit value may still be directly callable or guessable, and its remaining digits may link records across datasets.
- **Hard-to-audit corrections.** If the source value, conversion rule, and migration version are not retained, support teams cannot explain or reliably undo a bad conversion.

## Make the change safely

Treat the number as a string, not a number: preserve leading zeroes and the exact source value. Define one canonical international representation for matching and routing, and keep national display formats as presentation choices. Parse according to an explicit country and numbering plan; reject values that do not fit the expected source format instead of silently trimming digits.

Before migration, test the mapping for collisions, invalid lengths, missing prefixes, and ambiguous country codes. Keep the original value and a versioned mapping during the transition, with access limited to the systems and people that need it. Compare old and new records, route test traffic, monitor failed delivery and verification rates, and have a rollback path before retiring the old field.

If the goal is to make a stable pseudonymous join key, use a keyed HMAC over a canonicalised number and keep the secret key separate from the data. If an authorised system must recover the original number, use authenticated encryption and manage its keys accordingly. A plain hash or a shortened number is not a safe substitute: phone-number ranges are small enough to enumerate, especially when the country and prefix are known.

The central question is not “How do we fit 16 digits into 10?” It is “Which information must remain available, to whom, and for how long?” Decide that first, then choose a mapping that preserves the required routing and recovery behaviour—or explicitly accept that the old value cannot be reconstructed.
