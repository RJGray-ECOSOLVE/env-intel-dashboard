# 🌍 Environmental Intelligence Dashboard

> A real-time open-source intelligence platform aggregating global environmental feeds — deforestation, illegal wildlife trade, climate anomalies, disaster events, and AI-generated threat briefings — into a single operational view.

**Live:** [rjgray-ecosolve.github.io/env-intel-dashboard](https://rjgray-ecosolve.github.io/env-intel-dashboard/)

---

![Dashboard Screenshot](screenshot.png)

---

## Inspiration

This project was inspired by **[World Monitor](https://worldmonitor.org)** — a really cool security intelligence platform. I wanted to explore whether if I could replicate the core concept using only public APIs, open data, and a static HTML file and create something for environmental intelligrence. The answer turned out to be yes!

---

## What It Does

The dashboard pulls live data from 13+ sources and renders them in a single scrollable view with a persistent map panel at the top. No backend, no database, no build step — just one HTML file served statically from GitHub Pages.

### 🗺️ Map Tabs

| Tab | Source | Description |
|-----|--------|-------------|
| 🌳 Terrestrial | [Global Forest Watch](https://www.globalforestwatch.org) | Real-time deforestation & forest cover |
| 🌊 Marine | [Global Fishing Watch](https://globalfishingwatch.org) | Illegal & unregulated fishing activity |
| 🔍 Cyber / IWT | [Ecosolve](https://www.ecosolve.eco) | Cyber-enabled wildlife trafficking intelligence |
| 🦏 Wildeye IWT | [Wildeye / Oxpeckers](https://global.wildeye.oxpeckers.org) | Africa-focused wildlife crime mapping |

### 📊 Intelligence Panels

| Panel | Source | Data |
|-------|--------|------|
| 📰 Intelligence News Feeds | [Mongabay](https://news.mongabay.com), [Grist](https://grist.org), [The Guardian](https://theguardian.com/environment) | 7 topic feeds: wildlife crime, illegal logging, deforestation, mining, indigenous, climate policy |
| 🤖 AI Intel Briefings | [Groq](https://groq.com) / Meta Llama 3.3-70b | Date-aware AI briefings across Threat, Hotspots, Species, and Policy tabs |
| 🌍 Environmental Stability Index | [World Bank Open Data](https://data.worldbank.org), [ReliefWeb](https://reliefweb.int) | Composite 0–100 stability score for 20 countries across 5 weighted dimensions |
| 🔥 NASA EONET | [NASA EONET v3](https://eonet.gsfc.nasa.gov) | Live open natural events: wildfires (48hr), volcanoes, floods, storms, droughts |
| 🌡️ ERA5 Temp Anomaly | [Open-Meteo Archive](https://archive-api.open-meteo.com) | 14-day mean temperature vs. same period last year across 8 ecological zones |
| 🌊 Sea Surface Temp | [Open-Meteo Marine](https://marine-api.open-meteo.com) | SST anomaly vs. prior year across 8 ocean regions — coral bleaching risk indicator |
| 🚨 Wildlife Justice Commission | [WJC RSS](https://wildlifejustice.org/feed/) | Live enforcement operations and arrests |
| 🌪️ ReliefWeb Disasters | [UN OCHA ReliefWeb](https://reliefweb.int) | Active floods, droughts, wildfires, cyclones |
| 📈 Keyword Spike Monitor | [Google Trends](https://trends.google.com) | Flags when environmental crime terms hit daily trending searches |
| 📡 NOAA CO₂ | [NOAA GML Mauna Loa](https://gml.noaa.gov/ccgg/trends/) | Weekly atmospheric CO₂ ppm + year-on-year delta |

---

## Environmental Stability Index (ESI)

The ESI is a custom composite index built from public data, computed client-side on page load.

| Dimension | Source | Weight |
|-----------|--------|--------|
| 🌳 Forest Cover | World Bank `AG.LND.FRST.ZS` | 30% |
| 🦜 Protected Areas | World Bank `ER.PTD.TOTL.ZS` | 20% |
| 🌪️ Disaster Events | ReliefWeb (12-month count) | 20% |
| 💨 CO₂ Emissions | World Bank `EN.ATM.CO2E.PC` | 15% |
| 🏭 Air Quality | World Bank `EN.ATM.PM25.MC.M3` | 15% |

Click any country in the list to expand a full signal breakdown with individual dimension bars and status badges.

> **Note:** World Bank data carries a ~1–2 year publication lag. Missing values default to a neutral score of 50.

---

## AI Briefings

The AI panel uses **Meta's Llama 3.3-70b** model via the [Groq](https://groq.com) inference API. It generates four date-aware intelligence briefings on load:

- **⚠ Threat** — current global environmental crime vectors
- **📍 Hotspots** — highest-priority geographic areas
- **🦏 Species** — species under active trafficking pressure
- **📜 Policy** — recent enforcement and policy developments

Because API keys should never be exposed in client-side code, requests are routed through a **Cloudflare Worker** acting as a secure server-side proxy.

---

## Architecture

```
Browser (static HTML)
    │
    ├── Public APIs (no key needed)
    │     ├── NASA EONET
    │     ├── Open-Meteo (ERA5 + Marine)
    │     ├── World Bank
    │     └── ReliefWeb
    │
    ├── RSS Feeds (via CORS proxy chain)
    │     ├── corsproxy.io
    │     ├── allorigins.win
    │     ├── thingproxy.freeboard.io
    │     └── codetabs.com
    │
    └── Groq AI (via Cloudflare Worker proxy)
          └── GROQ_API_KEY stored as encrypted secret
```

Everything runs in the browser. There is no server, no database, no Node.js process, and no build pipeline. The Cloudflare Worker is the only server-side component — it exists solely to keep the Groq API key out of the client.

---

## Setup

### 1. Deploy the static dashboard

Fork or clone this repo. Enable GitHub Pages on the `main` branch. That's it — the dashboard is live.

### 2. Enable AI briefings (optional)

The AI Intel panel requires a free [Groq API key](https://console.groq.com/keys) and a [Cloudflare](https://cloudflare.com) account (free tier).

**Deploy the Worker:**

```bash
npm install -g wrangler
wrangler login
cd groq-proxy-worker
wrangler secret put GROQ_API_KEY   # paste your Groq key when prompted
wrangler deploy
```

**Connect to the dashboard:**

Open `env_intel_dashboard.html` and set your Worker URL:

```js
const AI_PROXY_URL = "https://your-worker.your-subdomain.workers.dev/ai";
```

**Optional — lock to your domain:**

```bash
wrangler secret put ALLOWED_ORIGIN   # e.g. https://yourusername.github.io
```

---

## Data Sources & Credits

This dashboard is an aggregation layer only. All data, maps, and intelligence belong to their respective providers:

| Source | Website |
|--------|---------|
| Global Forest Watch | [globalforestwatch.org](https://www.globalforestwatch.org) |
| Global Fishing Watch | [globalfishingwatch.org](https://globalfishingwatch.org) |
| Ecosolve | [ecosolve.eco](https://www.ecosolve.eco) |
| Wildeye / Oxpeckers | [global.wildeye.oxpeckers.org](https://global.wildeye.oxpeckers.org) |
| NASA EONET | [eonet.gsfc.nasa.gov](https://eonet.gsfc.nasa.gov) |
| NOAA Global Monitoring Laboratory | [gml.noaa.gov](https://gml.noaa.gov/ccgg/trends/) |
| Open-Meteo | [open-meteo.com](https://open-meteo.com) |
| World Bank Open Data | [data.worldbank.org](https://data.worldbank.org) |
| ReliefWeb (UN OCHA) | [reliefweb.int](https://reliefweb.int) |
| Wildlife Justice Commission | [wildlifejustice.org](https://wildlifejustice.org) |
| Mongabay | [news.mongabay.com](https://news.mongabay.com) |
| Grist | [grist.org](https://grist.org) |
| The Guardian Environment | [theguardian.com/environment](https://theguardian.com/environment) |
| Groq / Meta Llama 3.3 | [groq.com](https://groq.com) |
| World Monitor *(inspiration)* | [worldmonitor.org](https://worldmonitor.org) |

---

## Disclaimer

This project is an independent personal tool built for research and awareness purposes. It is **not affiliated with, endorsed by, or representative of** any of the data providers listed above. All trademarks and data remain the property of their respective owners.

---

## License

MIT — free to use, fork, and adapt. If you build on it, a credit or link back would be appreciated.

---

*Built by [RJ Gray](https://github.com/rjgray-ecosolve) · [Ecosolve](https://www.ecosolve.eco)*
