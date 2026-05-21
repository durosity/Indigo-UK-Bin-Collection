# UK Bin Collection Plugin for Indigo

[![Indigo](https://img.shields.io/badge/Indigo-2025.1+-blue.svg)](https://www.indigodomo.com/)
[![Version](https://img.shields.io/badge/version-1.2.0-green.svg)](https://github.com/Durosity/indigo-uk-bin-collection/releases)
[![License](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)

An [Indigo](https://www.indigodomo.com/) plugin that brings UK bin collection schedules into your smart home. Built on top of the excellent [UKBinCollectionData](https://github.com/robbrad/UKBinCollectionData) project by robbrad, it supports **300+ UK councils** and gives you everything you need to build reminders, control page widgets, and automations around your bin days.

---

## Features

- **300+ UK councils supported** via the bundled UKBinCollectionData library
- **Configurable update frequency** — hourly, every 2/6/12 hours, or daily
- **Rich device states** for triggers and conditions:
  - Next collection date and type
  - Days until next collection
  - Boolean flags for "collection today" and "collection tomorrow"
  - Combined types when multiple bins go out on the same day (e.g. *Recycling & Garden*)
  - Full JSON of all upcoming collections
- **Control page friendly** — three pre-formatted lines (e.g. *"Next Collection:" / "Tomorrow" / "Put Out Recycling at 7pm"*) plus a status state designed for icon-cycling
- **Manual bin status tracking** — mark a bin as Put Out → Collected → Put Away with built-in actions, perfect for a single button on a control page
- **Smart auto-reset** — when a new collection rolls round, the manual status clears itself
- **Selenium support** for councils whose websites require JavaScript rendering
- **All Python dependencies bundled** — no `pip install` required for most councils
- **Clean logging** with optional debug mode

---

## Installation

1. Download the latest `.indigoplugin` file from the [Releases](../../releases) page
2. Double-click the file — Indigo will install it
3. Enable the plugin in Indigo
4. Create a new **Bin Collection Schedule** device
5. Fill in your council details (see below)

> **Requires Indigo 2025.1 or later.**

---

## Configuration

### Required

| Field | Description |
|---|---|
| **Council Module Name** | The exact module name from the UKBinCollectionData project, e.g. `SheffieldCityCouncil`. Case-sensitive, no spaces. |
| **Council URL** | The URL to your council's bin collection page. |

### Optional (council-dependent)

| Field | Description |
|---|---|
| **Postcode** | Your postcode with a space, e.g. `LS1 2JG` |
| **House Number** | Your house/building number |
| **UPRN** | Unique Property Reference Number — find yours at [uprn.uk](https://uprn.uk) |
| **Selenium Server URL** | Only needed for councils with JavaScript-heavy sites. See [Selenium Support](#selenium-support) below. |

### Update Frequency

Choose how often the plugin polls your council: every hour, 2 hours, 6 hours (default), 12 hours, or once daily.

### Finding Your Council Details

The UKBinCollectionData wiki has the full list of supported councils, the exact module name, and which parameters each one needs:

👉 [**UKBinCollectionData Wiki**](https://github.com/robbrad/UKBinCollectionData/wiki)

You can jump there at any time via the plugin menu: **Plugins → UK Bin Collection → View Available Councils Wiki…**

---

## Device States

| State | Type | Description |
|---|---|---|
| `lastUpdate` | String | Timestamp of last successful update |
| `nextCollection` | String | Date of next collection (`YYYY-MM-DD`) |
| `nextCollectionType` | String | Type of next collection (e.g. `General Waste`, or `Recycling & Garden` when multiple bins are out on the same day) |
| `nextCollectionDays` | Integer | Days until next collection |
| `collectionToday` | Boolean | True if a collection is today |
| `collectionTomorrow` | Boolean | True if a collection is tomorrow |
| `collectionCP` | String | Compact status for control page icons — combines timing and bin category (e.g. `tomorrowRecycling`, `todayBin`, `futureGarden`), or a manual status (`putOut`, `collected`) |
| `manualBinStatus` | String | Current manual override (`""`, `putOut`, or `collected`) |
| `lastTrackedCollection` | String | Internal key used to detect when a new collection has rolled round |
| `controlPageLine1` | String | Always `Next Collection:` |
| `controlPageLine2` | String | `Today`, `Tomorrow`, or a friendly date like `13th Jan` |
| `controlPageLine3` | String | Context-aware action line, e.g. `Put Out Recycling at 7pm`, `Bin Put Out`, `Put Bin Away` |
| `allCollections` | String | JSON array of all upcoming collections |

---

## Actions

These actions are available in Indigo's action editor and can be assigned to control page buttons:

| Action | What it does |
|---|---|
| **Cycle Bin Status** | One-button cycle through: *Automatic → Put Out → Collected → Automatic*. Ideal for a single control page button. |
| **Mark Bin as Put Out** | Set manual status to *Put Out* |
| **Mark Bin as Collected** | Set manual status to *Collected* |
| **Mark Bin as Put Away** | Reset back to automatic tracking |
| **Reset Bin Status to Automatic** | Clear any manual override |

When a new collection date arrives, manual status resets itself automatically — you only ever need to touch this for the current week's collection.

---

## Automation Examples

**Notify the night before**
```
If device "My Bins" state collectionTomorrow becomes true
  Then send notification "Bins out tonight — " + state nextCollectionType
```

**Flash a lamp on collection morning**
```
If device "My Bins" state collectionToday is true
  And time is 07:00
  Then flash light "Hallway" blue 3 times
```

**Different action per bin type**
```
If device "My Bins" state nextCollectionType contains "Recycling"
  And state collectionTomorrow is true
  Then set variable "WhichBin" to "Blue recycling bin"
```

**Single-button control page widget**

Use the **Cycle Bin Status** action on a button, then bind the button's image to the `collectionCP` state. The state cleanly maps to icons like:
- `todayBin`, `todayRecycling`, `todayGarden`
- `tomorrowBin`, `tomorrowRecycling`, `tomorrowGarden`
- `futureBin`, `futureRecycling`, `futureGarden`
- `putOut`, `collected`

Combined types (e.g. `todayRecycling & Garden`) are also supported.

---

## Selenium Support

A handful of councils use heavily JavaScript-driven sites that need a real browser to scrape. For these, run Selenium in Docker:

```bash
docker run -d -p 4444:4444 --restart=always selenium/standalone-chrome
```

Then in the device config, set the **Selenium Server URL** to `http://localhost:4444`.

If your council doesn't need Selenium (most don't), leave this field blank.

---

## Menu Items

Found under **Plugins → UK Bin Collection**:

- **Update All Bin Collections** — forces an immediate refresh of every device
- **View Available Councils Wiki…** — opens the council list in your browser

---

## Troubleshooting

**"No bin data returned"**
- Double-check the council module name (case-sensitive, no spaces)
- Verify your council URL is correct
- Make sure any required parameters (postcode, UPRN, house number) are set
- Enable debug logging in plugin config and re-check the log

**"Council module not found"**
- The module name must match exactly. Check the [wiki](https://github.com/robbrad/UKBinCollectionData/wiki)
- Some councils have been renamed in the upstream project

**Connection errors**
- Confirm your internet connection
- Some councils block automated access — slowing the update frequency can help
- Council websites change occasionally; check the upstream project for fixes

**Selenium errors**
- Confirm the container is running: `docker ps`
- Test the endpoint: `http://localhost:4444`
- Double-check the URL in the device config

### Debug Logging

Enable **Show debug information in log** in plugin config to see raw JSON responses, date-parsing details, and full error traces.

---

## How it Works

The plugin wraps the [UKBinCollectionData](https://github.com/robbrad/UKBinCollectionData) Python library, which contains scrapers for hundreds of UK council websites. On each update cycle:

1. The plugin builds the appropriate command-line arguments from your device config
2. It runs the council's scraper and receives a JSON response
3. Results are parsed, sorted, filtered to future dates, and stored in device states
4. The control page lines and status state are computed
5. Manual bin status is preserved within a collection and auto-reset when a new one arrives

Updates run on a background thread; the polling interval is checked every 60 seconds against each device's configured frequency.

---

## Limitations

- Only councils supported by the upstream UKBinCollectionData project will work
- Selenium-dependent councils require an external Docker container
- Update frequency is intentionally capped at hourly to avoid hammering council servers
- Councils occasionally redesign their websites, which can break scraping until the upstream project catches up

---

## Support

- **Plugin issues** (Indigo behaviour, states, actions, control pages) — please [open an issue](../../issues) on this repo
- **Council-specific issues** (scraper not working, council not in the list) — these belong upstream at [UKBinCollectionData/issues](https://github.com/robbrad/UKBinCollectionData/issues)

---

## Credits

- **[UKBinCollectionData](https://github.com/robbrad/UKBinCollectionData)** by robbrad — the heavy lifting behind every council scraper
- Plugin development by **Durosity** for Indigo 2025.1
- [Beautiful Soup 4](https://www.crummy.com/software/BeautifulSoup/) for HTML parsing
- [Requests](https://requests.readthedocs.io/) for HTTP

---

## License

MIT — see [LICENSE](LICENSE).

The bundled UKBinCollectionData library is also MIT-licensed by its respective author.

---

## Version History

### 1.2.0 — First public release (2026-05-21)
- Initial public/GitHub release
- 300+ UK councils supported via bundled UKBinCollectionData
- Selenium support for JavaScript-heavy council sites
- Manual bin status tracking with one-button cycle action
- Three control-page line states with friendly date formatting (`13th Jan`, `Tomorrow`, etc.)
- Combined bin-type detection (e.g. *Recycling & Garden*)
- Smart bin-type mapping into three icon-friendly categories
- Auto-reset of manual status on new collection
- All Python dependencies bundled

