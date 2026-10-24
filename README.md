# IoT Data Integration: Greenhouse Scenario

This repository integrates three data files from one deployment into a single workbook, and documents what each file contains.

All data covers one day: **July 15, 2026**.

## What's here

| Path | What it is |
|---|---|
| `source-data/feed102.xml` | Weather station feed (`WS-01`): air temperature, humidity, wind speed, sunlight. Every 10 minutes. |
| `source-data/batch102.csv` | Soil sensor batch export: moisture, temperature, and battery for 10 sensors in 5 beds. Every 2 hours. |
| `source-data/stream102.json` | Irrigation pump log: zone, flow rate, and pump status. Every 5 minutes while a zone is running. |
| `integrated_102.xlsx` | The integrated dataset (see below). |
| `DATA_DICTIONARY.md` | Every field in the three source files and in the workbook. |
| `REPORT.md` | How the files were integrated, what they show, and their limits. |

## The integrated workbook

`integrated_102.xlsx` has five sheets:

- **`Integrated_10min`**: one row per weather reading (144 rows) with irrigation and soil data joined on. This is the main sheet.
- **`Weather`**, **`Soil`**, **`Irrigation`**: every row of each source file, with dates and times reformatted to one style.
- **`Notes`**: a short summary of the integration rules.

## How the files differ

The files use different conventions, so they cannot be combined as-is:

| | `feed102.xml` | `batch102.csv` | `stream102.json` |
|---|---|---|---|
| Format | XML | CSV | JSON |
| Date | `DD/MM/YYYY` | `M/D/YYYY` | `MM/DD/YYYY` |
| Time | 24-hour | 12-hour AM/PM | 12-hour AM/PM |
| Temperature unit | °C | °F | n/a |
| Reading interval | 10 min | 2 hours | 5 min while running |

## Key points to know

- Irrigation zones (1 to 3) and soil beds (1 to 5) cannot be linked, because no file says which zone waters which bed.
- Three fields are empty in the source files: `sunlight`, `SoilBattery_L`, and `activationMode`.
- Soil readings are only taken every 2 hours, so in `Integrated_10min` each reading is carried forward until the next one (`soil_reading_time` shows which one).

See `REPORT.md` for details and `DATA_DICTIONARY.md` for field definitions.

## How to use

Open `integrated_102.xlsx` in Excel or any spreadsheet program. No installation is needed.
