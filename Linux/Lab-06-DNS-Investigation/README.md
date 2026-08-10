# Lab 06: DNS Investigation with nslookup

## Objective

Use `nslookup` to investigate how DNS translates domain names into IP addresses.

## Tool Used

- Ubuntu on WSL
- `nslookup` from the `dnsutils` package

## DNS Lookup: google.com

Command used:

```bash
nslookup google.com

## DNS Lookup: github.com

Command used:

```bash
nslookup github.com

## MX Record Lookup: gmail.com

Command used:

```bash
nslookup -type=mx gmail.com