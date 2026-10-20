# vizio-dns-blocklist

A small, conservative DNS blocklist for **Vizio SmartCast TVs**. It cuts home-screen ads, ad measurement,
ACR viewing-data reporting (Inscape) and telemetry, and leaves the hosts SmartCast, the built-in apps,
Chromecast and firmware updates depend on.

SmartCast is a locked Linux platform: no ADB, no sideloading, no way to disable apps or replace the
launcher. Android TV debloat tools don't work on it, so filtering at the network is the main option you have.

Built for a 2021 V-Series (V655-H19, firmware group MG306) but the hosts are SmartCast-wide.

## Use it

Subscribe in **AdGuard Home** → *Filters* → *DNS blocklists* → *Add blocklist* → *Add a custom list*:

```
https://raw.githubusercontent.com/d2x/vizio-dns-blocklist/master/vizio-blocklist.txt
```

The file uses AdGuard/adblock syntax (`||host^`), which AdGuard Home, AdGuard DNS and uBlock-style
DNS tools read directly. To undo it, disable or remove the list.

## Why ads might still get through: the TV bypasses your DNS

SmartCast's Chromecast stack can fall back to Google DNS (`8.8.8.8` / `8.8.4.4`). If AdGuard Home's
query log shows few or no queries from the TV, the list is not in the path. On your router:

1. Redirect (DNAT) all TCP/UDP port 53 traffic from the TV to your AdGuard Home instance.
2. Block TCP/UDP 853 (DNS-over-TLS) from the TV.
3. Optionally block known DNS-over-HTTPS resolvers (e.g. `dns.google`) for the TV.

## What this list doesn't do

- It **doesn't remove app tiles or the WatchFree+ rows.** Some home-screen content is served from the same
  hosts as normal artwork, and blocking those (e.g. `image-api-prod.smartcasttv.com`) also breaks app icons.
- It **doesn't block YouTube or other in-app ads.** Those are served by the apps' own content hosts.
- It **doesn't replace the TV's own privacy settings.** Also set *Admin & Privacy → Viewing Data: Off* and
  *Advertising → Limit Ad Tracking: On*.

For a fully ad-free setup, use an external streaming box and block the TV from the internet.

## Deliberately not blocked

Blocking these breaks the TV. Don't add them unless you want it offline:

| Host | Needed for |
|---|---|
| `api.vizio.com`, `scfs.vizio.com` | SmartCast core, app catalogue/launch config |
| `osu.vudip.vizio.com` | firmware updates (security fixes; there is no rollback) |
| `vizio.pool.ntp.org` | clock (wrong time breaks TLS) |
| `*.plat.vizio.com` | device registration |
| `user-authorization-prod`, `video-api-prod`, `catalog-prod`, `watchfreeplus-epg-prod`, `voice-search-prod`, `image-api-prod` `.smartcasttv.com` | sign-in, playback, catalogue, EPG, voice, artwork |

## Sources and confidence

- The Sentry crash/trace project ID comes from the TV's own `/state/device/deviceinfo` (SmartCast API).
- Other hosts come from community Smart-TV blocklists (Okazakee/tv-dns-blocklist, Perflyst/PiHoleBlocklist
  SmartTV list) and were checked to resolve in public DNS on 2026-10-20. `ad-broker-prod`, `promotions-prod` and
  `rlog_sem` didn't resolve that day; they stay listed because blocking a missing name has no effect.
- Inclusion policy: any host a source flags as possible ad serving, ad measurement or tracking stays listed, even if
  its link to Vizio is weak (e.g. `admeasurement.com`) or it only matters to older sets (the Yahoo widget hosts).
  A host is removed only if blocking it breaks something.
- They have **not yet been checked against a live TV's query log**. Additions should come from real query logs.

## Contributing

Open an issue or PR with the hostname, the AdGuard Home / Pi-hole query-log evidence (model + firmware), and
what, if anything, broke when you blocked it. Remove your TV's serial number and ad ID from any logs you share.

## License

[Unlicense](https://unlicense.org/) (public domain).
