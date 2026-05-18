# intellekt-statuspage

Public uptime monitoring for intellekt APIs at [`status.intellekt.fyi`](https://status.intellekt.fyi).

Forked from the [Upptime](https://upptime.js.org) template. Probes run as GitHub Actions on a 5-minute cron; results commit back to this repo; incidents auto-open as Issues; the static site rebuilds daily and deploys to GitHub Pages.

## Add or remove a service

Edit [`.upptimerc.yml`](./.upptimerc.yml) under `sites:`. Push to `master`. The `Setup CI` workflow regenerates the probe workflows and the new service is picked up on the next 5-min tick.

## Status, design, and operational notes

See [`charters/statuspage.md`](./charters/statuspage.md) — authoritative source for what's probed, why it's probed that way, branding decisions, and what's deferred.

## Incidents

Open incidents appear as [GitHub Issues](../../issues) with the `incident` label. Auto-closed on recovery.

## License

MIT. Per-service history data under [Open Data Commons ODL 1.0](https://opendatacommons.org/licenses/odbl/1-0/), inherited from the Upptime template.
