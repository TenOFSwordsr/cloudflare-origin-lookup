# cloud.rb - Cloudflare Origin IP Lookup via crimeflare.org

A short Ruby script from an old personal collection that takes a hostname fronted by Cloudflare and
tries to recover the real origin address behind it, then annotates the answer with geolocation and
hosting data. It does no probing of its own - the disclosure work is done by a third-party CGI, and
this is a thin client around it. A copy of the HatBrasil "hatcloud" tool rather than original work.

**Suggested repo name:** `cloudflare-origin-lookup`
**Stack:** Ruby stdlib only (`net/http`, `open-uri`, `json`, `socket`, `optparse`); depends on crimeflare.org and ipinfo.io
**Status:** archived
**Last modified:** 2019-12-04

## What it does

- Accepts one target: `-b` / `--byp <host>`; `-o/--out` is parsed but reserved for a "next release"
  mass mode that was never written, and `-h` prints the banner plus usage.
- POSTs the hostname as `cfS` to `http://www.crimeflare.org/cgi-bin/cfsearch.cgi`, then scans the
  response: a `No working nameservers are registered` string means the domain is not Cloudflare
  protected, and the first `d.d.d.d` match is taken as the origin IP.
- Resolves the input hostname locally with `IPSocket.getaddress` and prints that as the Cloudflare
  edge IP next to the claimed real IP.
- Fetches `http://ipinfo.io/<ip>/json` and prints hostname, city, region, location and organization
  for the origin.
- ASCII-art banner credits `fb.com/hatbashbr` and `github.com/hatbashbr`.

## Layout

```
cloud.rb   the whole script (~93 lines)
```

## Running it

```
ruby cloud.rb -b example.com
```

## Notes

- Non-functional today: crimeflare.org's search CGI is long gone, so the request now falls into the
  "No valid address" branch for every input.
- Origin disclosure is a real weakness in a CDN setup, but using it against infrastructure you do not
  own or are not authorised to test is unauthorised reconnaissance. Treat the tool as archival, and
  if you publish it say so plainly.
- Written before routine practice changed: plain HTTP for both services, no TLS, no timeouts, an
  unguarded `json['hostname']` access that will raise on ipinfo.io rate-limit responses, and a first
  IPv4 regex that can match an unrelated dotted number in the page body.
- Not your code - the banner attributes it to HatBrasil; keep that credit if it goes public.
