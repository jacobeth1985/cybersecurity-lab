 # Lab 06: DNS Investigation with nslookup

## Objective

Use `nslookup` to investigate how DNS translates domain names into IP addresses.

## Tool Used

- Ubuntu on WSL
- `nslookup` from the `dnsutils` package

## DNS Lookup: google.com

Command used:

    nslookup google.com

## What I Learned

DNS stands for Domain Name System. It translates a human-readable domain name, such as `google.com`, into IP addresses that computers use to communicate.

The result was marked **Non-authoritative answer**. This means the DNS server provided a cached response rather than answering directly as the authoritative DNS server for Google.

## DNS Lookup: github.com

Command used:

    nslookup github.com

## Result

GitHub resolved to this IPv4 address:

    20.87.245.0

## Observation

A single domain can resolve to one or more IP addresses. These addresses can change over time because large services use distributed infrastructure.

## MX Record Lookup: gmail.com

Command used:

    nslookup -type=mx gmail.com

## Result

Gmail returned these mail exchangers:

- Priority 5: `gmail-smtp-in.l.google.com`
- Priority 10: `alt1.gmail-smtp-in.l.google.com`
- Priority 20: `alt2.gmail-smtp-in.l.google.com`
- Priority 30: `alt3.gmail-smtp-in.l.google.com`
- Priority 40: `alt4.gmail-smtp-in.l.google.com`

## What I Learned

MX means **Mail Exchanger**. MX records tell other mail systems which servers should receive email for a domain.

The priority number matters: lower numbers have higher priority. Gmail’s primary mail server has priority 5, while the `alt` servers provide backup options.