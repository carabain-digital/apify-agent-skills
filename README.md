# Apify Agent Skills by Carabain Digital

Agent skills that let AI agents (Claude Code, Codex, Cursor, Gemini CLI and any agent that reads `SKILL.md`) get
public web data through [our Apify Actors](https://apify.com/carabain-digital): car listings, real estate, jobs, software
reviews, market data and more, from sites across Europe and the US. Each skill tells the agent when to use the
Actor, how to call it (Apify API, Python client or the Apify MCP server), every input field with examples, the
output fields and the price.

## Install

With the [skills CLI](https://skills.sh) (picks the right folder for your agent):

```bash
npx skills add carabain-digital/apify-agent-skills                            # choose from the list
npx skills add carabain-digital/apify-agent-skills --skill mobile-de-listings # one skill
```

Or copy a folder from `skills/` into your agent's skills directory (Claude Code: `~/.claude/skills/`).

The Actors run on [Apify](https://apify.com). You need an Apify account and an API token (`APIFY_TOKEN`):
[sign up free](https://console.apify.com/sign-up), no credit card needed; the Free plan includes $5 of credit every month. Runs are
billed per result to your own Apify account.

## Car listings

| Skill | What it does | `--skill` | Actor |
|---|---|---|---|
| [AutoScout24 Car Listings Scraper](skills/autoscout24-listings/SKILL.md) | Scrape autoscout24.de, .nl, .be, .at, .it, .fr, .es car listings: make, model, price, mileage, year, fuel, price rating, dealer phone. JSON/CSV. | `autoscout24-listings` | [Apify Store](https://apify.com/carabain-digital/autoscout24-listings) |
| [Autotrader Car Listings Scraper](skills/autotrader-listings/SKILL.md) | Scrape autotrader.com car listings by ZIP: make, model, price, mileage, VIN, KBB fair price, deal rating, dealer phone and address. Export JSON/CSV. | `autotrader-listings` | [Apify Store](https://apify.com/carabain-digital/autotrader-listings) |
| [Belgium Car Listings Scraper](skills/belgium-cars/SKILL.md) | Scrape used and new cars in Belgium from AutoScout24, Gocar and 2dehands in one run: price, km, year, fuel, dealer. Deduplicated. JSON/CSV/Excel. | `belgium-cars` | [Apify Store](https://apify.com/carabain-digital/belgium-cars) |
| [CarGurus Car Listings Scraper](skills/cargurus-listings/SKILL.md) | Scrape cargurus.com used and new car listings by ZIP, make and model: price, CarGurus deal rating, market value (IMV), VIN, mileage, dealer phone. | `cargurus-listings` | [Apify Store](https://apify.com/carabain-digital/cargurus-listings) |
| [Gaspedaal Car Listings Scraper](skills/gaspedaal-listings/SKILL.md) | Scrape gaspedaal.nl used cars: make, model, price, km, year, fuel, power, city and source site. Filter by make, price, year, postcode. JSON/CSV. | `gaspedaal-listings` | [Apify Store](https://apify.com/carabain-digital/gaspedaal-listings) |
| [Germany Cars Scraper](skills/germany-cars/SKILL.md) | Scrape German car listings from mobile.de, AutoScout24, Kleinanzeigen and AutoUncle in one run: price, km, year, fuel, seller, deduplicated. JSON/CSV/Excel. | `germany-cars` | [Apify Store](https://apify.com/carabain-digital/germany-cars) |
| [Marktplaats Car Listings Scraper](skills/marktplaats-cars/SKILL.md) | Scrape car listings from marktplaats.nl: price, mileage, year, fuel, gearbox, seller, photos. Filter by make, model, price. Export JSON/CSV. | `marktplaats-cars` | [Apify Store](https://apify.com/carabain-digital/marktplaats-cars) |
| [mobile.de Car Listings Scraper](skills/mobile-de-listings/SKILL.md) | Scrape mobile.de car listings: make, model, price, km, year, fuel, power, dealer phone and price rating. Filter by make, price, year. Export JSON/CSV. | `mobile-de-listings` | [Apify Store](https://apify.com/carabain-digital/mobile-de-listings) |
| [Netherlands Car Listings Scraper](skills/netherlands-cars/SKILL.md) | Scrape Dutch car listings from AutoScout24, Marktplaats, AutoTrack, AutoWereld, Gaspedaal and AutoUncle in one run: occasions, deduplicated, newest first. JSON/CSV/Excel. | `netherlands-cars` | [Apify Store](https://apify.com/carabain-digital/netherlands-cars) |
| [USA Car Listings Scraper](skills/usa-cars/SKILL.md) | Scrape US car listings from CarGurus, Autotrader, Cars.com, TrueCar, CarMax, Edmunds, Carvana and Craigslist in one run, deduped on VIN. JSON/CSV/Excel. | `usa-cars` | [Apify Store](https://apify.com/carabain-digital/usa-cars) |

## Real estate

| Skill | What it does | `--skill` | Actor |
|---|---|---|---|
| [Belgium Real Estate Scraper](skills/belgium-real-estate/SKILL.md) | Scrape Belgian vastgoed from Zimmo, Immoweb and Immovlan in one run: huizen en appartementen te koop of te huur, deduplicated. JSON/CSV/Excel. | `belgium-real-estate` | [Apify Store](https://apify.com/carabain-digital/belgium-real-estate) |
| [Bien'ici Scraper](skills/bienici-listings/SKILL.md) | Scrape bienici.com property listings for sale and rent: filter by city, price, rooms and area. Price, DPE, photos, agency. Export JSON/CSV. Price history. | `bienici-listings` | [Apify Store](https://apify.com/carabain-digital/bienici-listings) |
| [France Real Estate Scraper](skills/france-real-estate/SKILL.md) | Scrape annonces immobilières from SeLoger, Bien'ici and PAP in one run: appartements et maisons, location ou vente, newest first. JSON/CSV/Excel. | `france-real-estate` | [Apify Store](https://apify.com/carabain-digital/france-real-estate) |
| [Germany Real Estate Scraper](skills/germany-real-estate/SKILL.md) | Scrape Immobilien from ImmoScout24 and Immowelt in one run: Wohnungen und Häuser zur Miete oder zum Kauf, deduplicated. JSON/CSV/Excel. | `germany-real-estate` | [Apify Store](https://apify.com/carabain-digital/germany-real-estate) |
| [Huislijn Listings Scraper](skills/huislijn-listings/SKILL.md) | Scrape Huislijn.nl listings for sale and rent in the Netherlands: price, address, area, rooms, status, agent. Track price changes. Export JSON, CSV, Excel. | `huislijn-listings` | [Apify Store](https://apify.com/carabain-digital/huislijn-listings) |
| [ImmoScout24 Scraper](skills/immoscout24-listings/SKILL.md) | Scrape immobilienscout24.de listings for rent and sale: filter by city, price, rooms and area. Price, address, energy class, agent. Export JSON/CSV. | `immoscout24-listings` | [Apify Store](https://apify.com/carabain-digital/immoscout24-listings) |
| [Immovlan Listings Scraper](skills/immovlan-listings/SKILL.md) | Scrape Immovlan.be listings for sale and rent in Belgium: price, location, EPC, area, bedrooms, agent. Track price changes. Export JSON, CSV, Excel. | `immovlan-listings` | [Apify Store](https://apify.com/carabain-digital/immovlan-listings) |
| [Immoweb Scraper](skills/immoweb-listings/SKILL.md) | Scrape immoweb.be property listings for sale and rent: filter by city, price, bedrooms and area. Price, address, EPC, agent. Export JSON/CSV. Price history. | `immoweb-listings` | [Apify Store](https://apify.com/carabain-digital/immoweb-listings) |
| [Immowelt Scraper](skills/immowelt-listings/SKILL.md) | Scrape immowelt.de property listings for sale and rent: filter by city, price, rooms, area. Price, address, energy label, agent phone. Export JSON/CSV. | `immowelt-listings` | [Apify Store](https://apify.com/carabain-digital/immowelt-listings) |
| [Netherlands Real Estate Scraper](skills/netherlands-real-estate/SKILL.md) | Scrape Dutch huurwoningen en koopwoningen from Huislijn and Pararius in one run, deduplicated. JSON/CSV/Excel. | `netherlands-real-estate` | [Apify Store](https://apify.com/carabain-digital/netherlands-real-estate) |
| [PAP.fr Scraper](skills/pap-listings/SKILL.md) | Scrape pap.fr listings from private sellers, for sale and rent: filter by city, price, rooms and area. Price, DPE, photos. JSON/CSV. Price history. | `pap-listings` | [Apify Store](https://apify.com/carabain-digital/pap-listings) |
| [Pararius Listings Scraper](skills/pararius-listings/SKILL.md) | Scrape Pararius rental and sale listings: price, address, m2, rooms, agent, photos. Filter by city, price and size. Export JSON/CSV/Excel. | `pararius-listings` | [Apify Store](https://apify.com/carabain-digital/pararius-listings) |
| [SeLoger Scraper](skills/seloger-listings/SKILL.md) | Scrape seloger.com property listings (annonces immobilières) for sale and rent: filter by city, price, rooms, area. Price, DPE, agent phone. Export JSON/CSV. | `seloger-listings` | [Apify Store](https://apify.com/carabain-digital/seloger-listings) |
| [Zimmo Listings Scraper](skills/zimmo-listings/SKILL.md) | Scrape zimmo.be property listings for sale and rent: filter by city, price, bedrooms and area. Price, address, EPC, agent. Export JSON/CSV. Price history. | `zimmo-listings` | [Apify Store](https://apify.com/carabain-digital/zimmo-listings) |

## Jobs

| Skill | What it does | `--skill` | Actor |
|---|---|---|---|
| [Arbeitsagentur Jobs Scraper](skills/arbeitsagentur-jobs/SKILL.md) | Scrape jobs from the Bundesagentur für Arbeit Jobbörse: filter by keyword, city, radius and date. Salary, coordinates, full text. Export JSON/CSV. | `arbeitsagentur-jobs` | [Apify Store](https://apify.com/carabain-digital/arbeitsagentur-jobs) |
| [France Jobs Scraper](skills/france-jobs/SKILL.md) | Scrape offres d'emploi from Welcome to the Jungle and France Travail in one run: CDI, CDD, alternance, deduplicated, newest first. JSON/CSV/Excel. | `france-jobs` | [Apify Store](https://apify.com/carabain-digital/france-jobs) |
| [France Travail Jobs Scraper](skills/francetravail-jobs/SKILL.md) | Scrape France Travail offres d'emploi: filter by keyword, city, radius, CDI/CDD/intérim/alternance and date. Employer, contract, salary. Export JSON/CSV. | `francetravail-jobs` | [Apify Store](https://apify.com/carabain-digital/francetravail-jobs) |
| [Germany Jobs Scraper](skills/germany-jobs/SKILL.md) | Scrape Stellenangebote from Arbeitsagentur and StepStone in one run: Jobs in Deutschland, deduplicated, newest first. JSON, CSV, Excel. | `germany-jobs` | [Apify Store](https://apify.com/carabain-digital/germany-jobs) |
| [Intermediair Jobs Scraper](skills/intermediair-jobs/SKILL.md) | Scrape Intermediair job postings: title, company, salary, hours, location, full description and apply link. Search by keyword, place and date. | `intermediair-jobs` | [Apify Store](https://apify.com/carabain-digital/intermediair-jobs) |
| [Jobbird Jobs Scraper](skills/jobbird-jobs/SKILL.md) | Scrape Dutch job postings from Jobbird.com: title, company, city, date, salary, hours. Filter by keyword, location and age. No proxy needed. JSON, CSV, Excel. | `jobbird-jobs` | [Apify Store](https://apify.com/carabain-digital/jobbird-jobs) |
| [Nationale Vacaturebank Jobs Scraper](skills/nationale-vacaturebank-jobs/SKILL.md) | Scrape Nationale Vacaturebank job postings: title, company, salary, hours, location, full description and apply link. Search by keyword, place and date. | `nationale-vacaturebank-jobs` | [Apify Store](https://apify.com/carabain-digital/nationale-vacaturebank-jobs) |
| [Netherlands Jobs Scraper](skills/netherlands-jobs/SKILL.md) | Scrape vacatures from Nationale Vacaturebank, Intermediair and Jobbird in one run, deduplicated, newest first. JSON, CSV, Excel. | `netherlands-jobs` | [Apify Store](https://apify.com/carabain-digital/netherlands-jobs) |
| [StepStone Jobs Scraper](skills/stepstone-jobs/SKILL.md) | Scrape German job postings from StepStone.de: title, company, location, date, home office, description. Filter by keyword, city and age. JSON, CSV, Excel. | `stepstone-jobs` | [Apify Store](https://apify.com/carabain-digital/stepstone-jobs) |
| [VDAB Jobs Scraper](skills/vdab-jobs/SKILL.md) | Scrape jobs from VDAB (Flanders, Belgium): filter by keyword, location, radius and date. Contract type, experience, full text. Export JSON/CSV. | `vdab-jobs` | [Apify Store](https://apify.com/carabain-digital/vdab-jobs) |
| [Welcome to the Jungle Jobs Scraper](skills/welcometothejungle-jobs/SKILL.md) | Scrape Welcome to the Jungle jobs in France: filter by keyword, city, radius, CDI/alternance/stage and date. Salary, remote, text. Export JSON/CSV. | `welcometothejungle-jobs` | [Apify Store](https://apify.com/carabain-digital/welcometothejungle-jobs) |

## Software reviews and business leads

| Skill | What it does | `--skill` | Actor |
|---|---|---|---|
| [BBB Scraper](skills/bbb-scraper/SKILL.md) | Scrape BBB.org businesses by keyword and city: rating, accreditation, phone, address. Add-ons: owners, email, website, reviews, complaints. | `bbb-scraper` | [Apify Store](https://apify.com/carabain-digital/bbb-scraper) |
| [Capterra Scraper](skills/capterra-scraper/SKILL.md) | Scrape Capterra software per category: vendor, ratings, review counts, features. Add-ons: pricing plans, AI pros and cons, user reviews. | `capterra-scraper` | [Apify Store](https://apify.com/carabain-digital/capterra-scraper) |
| [G2 Scraper](skills/g2-scraper/SKILL.md) | Scrape G2.com software per category: vendor, rating, review count, pros, cons, prices. Add-ons: website, pricing plans, user reviews. | `g2-scraper` | [Apify Store](https://apify.com/carabain-digital/g2-scraper) |

## EU market data

| Skill | What it does | `--skill` | Actor |
|---|---|---|---|
| [EU Dividends & Splits by ISIN](skills/eu-dividends/SKILL.md) | Dividend history, calendar and splits per ISIN for European and UK shares: ex-date, pay date, type, declared amount. For portfolio trackers. JSON/CSV. | `eu-dividends` | [Apify Store](https://apify.com/carabain-digital/eu-dividends) |
| [EU Short Positions Register](skills/eu-short-positions/SKILL.md) | Disclosed net short positions in European and UK shares from 9 regulators: holder, issuer, ISIN, net short %, date. Current or full history. JSON/CSV. | `eu-short-positions` | [Apify Store](https://apify.com/carabain-digital/eu-short-positions) |

## Events and tickets

| Skill | What it does | `--skill` | Actor |
|---|---|---|---|
| [Bandsintown Events Scraper](skills/bandsintown-events/SKILL.md) | Scrape bandsintown.com concert and tour dates by artist: date, venue, city, country, coordinates, tickets, status. Export JSON/CSV. No account needed. | `bandsintown-events` | [Apify Store](https://apify.com/carabain-digital/bandsintown-events) |
| [TicketSwap Listings Scraper](skills/ticketswap-listings/SKILL.md) | Scrape TicketSwap resale listings: price per ticket with fees, face value, quantity, seats, ticket type, event, venue. Sold prices too. Export JSON/CSV/Excel. | `ticketswap-listings` | [Apify Store](https://apify.com/carabain-digital/ticketswap-listings) |

## More tools

| Skill | What it does | `--skill` | Actor |
|---|---|---|---|
| [Discogs Marketplace Scraper](skills/discogs-marketplace/SKILL.md) | Scrape the Discogs marketplace: every copy for sale with price, shipping, media and sleeve condition, seller rating, last sold and median price. JSON/CSV. | `discogs-marketplace` | [Apify Store](https://apify.com/carabain-digital/discogs-marketplace) |
| [Google Sheets Export](skills/google-sheets-export/SKILL.md) | Export any Apify dataset to Google Sheets: replace, append, or add only new rows by key. Reliable service account, big datasets, schedules. | `google-sheets-export` | [Apify Store](https://apify.com/carabain-digital/google-sheets-export) |
| [Google Trends Scraper](skills/google-trends/SKILL.md) | Google Trends by keyword or URL: interest over time, by country, region, city, related queries, Trending now. Drop-in for Apify's scraper, no 429s. | `google-trends` | [Apify Store](https://apify.com/carabain-digital/google-trends) |
| [Reddit Scraper](skills/reddit-scraper/SKILL.md) | Scrape Reddit posts, comments, subreddits, users and search without login or API key. Score, text, replies, media. Drop-in for Reddit Scraper Lite. | `reddit-scraper` | [Apify Store](https://apify.com/carabain-digital/reddit-scraper) |
| [Website Screenshot & PDF](skills/website-screenshot-pdf/SKILL.md) | Full-page PNG/JPEG screenshots and real text PDFs of any URL, also behind Cloudflare. Cookie banners removed. Residential proxy built in. | `website-screenshot-pdf` | [Apify Store](https://apify.com/carabain-digital/website-screenshot-pdf) |

Generated from the Actors' own input and output schemas, so fields and prices match the live Actors.
Questions or a site you need? Open an issue.

## License

[MIT](LICENSE)
