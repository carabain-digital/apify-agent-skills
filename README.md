# Apify Agent Skills by Carabain Digital

Agent skills that let AI agents (Claude Code, Codex, Cursor and any agent that reads `SKILL.md`) scrape public
web data through our [Apify Actors](https://apify.com/carabain-digital): car listings from Europe's and America's
biggest marketplaces, with more verticals coming. Each skill explains when to use the Actor, how to call it
(Apify API, Python client or the Apify MCP server), every input field with examples, the output fields and the
price.

## Install

Copy a skill folder into your agent's skills directory, for example for Claude Code:

```bash
git clone https://github.com/carabain-digital/apify-agent-skills
cp -r apify-agent-skills/skills/mobile-de-listings ~/.claude/skills/
```

The Actors run on [Apify](https://apify.com): you need an Apify account and an API token (`APIFY_TOKEN`).
Runs are billed per result to your Apify account; the Apify Free plan includes monthly credit to try them.

## Skills

| Skill | What it does | Actor |
|---|---|---|
| [AutoScout24 Car Listings Scraper](skills/autoscout24-listings/SKILL.md) | Scrape autoscout24.de, .nl, .be, .at, .it, .fr, .es car listings: make, model, price, mileage, year, fuel, price rating, dealer phone. JSON/CSV. | [Apify Store](https://apify.com/carabain-digital/autoscout24-listings) |
| [Autotrader Car Listings Scraper](skills/autotrader-listings/SKILL.md) | Scrape autotrader.com car listings by ZIP: make, model, price, mileage, VIN, KBB fair price, deal rating, dealer phone and address. Export JSON/CSV. | [Apify Store](https://apify.com/carabain-digital/autotrader-listings) |
| [Belgium Car Listings Scraper](skills/belgium-cars/SKILL.md) | Scrape used and new cars in Belgium from AutoScout24, Gocar and 2dehands in one run: price, km, year, fuel, dealer. Deduplicated. JSON/CSV/Excel. | [Apify Store](https://apify.com/carabain-digital/belgium-cars) |
| [CarGurus Car Listings Scraper](skills/cargurus-listings/SKILL.md) | Scrape cargurus.com used and new car listings by ZIP, make and model: price, CarGurus deal rating, market value (IMV), VIN, mileage, dealer phone. | [Apify Store](https://apify.com/carabain-digital/cargurus-listings) |
| [Gaspedaal Car Listings Scraper](skills/gaspedaal-listings/SKILL.md) | Scrape gaspedaal.nl used cars: make, model, price, km, year, fuel, power, city and source site. Filter by make, price, year, postcode. JSON/CSV. | [Apify Store](https://apify.com/carabain-digital/gaspedaal-listings) |
| [Germany Cars Scraper](skills/germany-cars/SKILL.md) | Scrape German car listings from mobile.de, AutoScout24, Kleinanzeigen and AutoUncle in one run: price, km, year, fuel, seller, deduplicated. JSON/CSV/Excel. | [Apify Store](https://apify.com/carabain-digital/germany-cars) |
| [Marktplaats Car Listings Scraper](skills/marktplaats-cars/SKILL.md) | Scrape car listings from marktplaats.nl: price, mileage, year, fuel, gearbox, seller, photos. Filter by make, model, price. Export JSON/CSV. | [Apify Store](https://apify.com/carabain-digital/marktplaats-cars) |
| [mobile.de Car Listings Scraper](skills/mobile-de-listings/SKILL.md) | Scrape mobile.de car listings: make, model, price, km, year, fuel, power, dealer phone and price rating. Filter by make, price, year. Export JSON/CSV. | [Apify Store](https://apify.com/carabain-digital/mobile-de-listings) |
| [Netherlands Car Listings Scraper](skills/netherlands-cars/SKILL.md) | Scrape Dutch car listings from AutoScout24, Marktplaats, AutoTrack, AutoWereld, Gaspedaal and AutoUncle in one run: occasions, deduplicated, newest first. JSON/CSV/Excel. | [Apify Store](https://apify.com/carabain-digital/netherlands-cars) |
| [USA Car Listings Scraper](skills/usa-cars/SKILL.md) | Scrape US car listings from CarGurus, Autotrader, Cars.com, TrueCar, CarMax, Edmunds, Carvana and Craigslist in one run, deduped on VIN. JSON/CSV/Excel. | [Apify Store](https://apify.com/carabain-digital/usa-cars) |

Generated from the Actors' own input and output schemas, so fields and prices match the live Actors.
