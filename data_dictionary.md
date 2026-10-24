# Data Dictionary

All three source files cover a single day, **July 15, 2026**.

## Source fields

| File | Field | Type | Format / Unit | Observed values | Description |
|---|---|---|---|---|---|
| `feed102.xml` | `Station` `id` attribute | string | text | `WS-01` | ID of the weather station that produced the feed. |
| `feed102.xml` | `observation` `date` attribute | date | `DD/MM/YYYY` | `15/07/2026` | Date of the reading (day first). |
| `feed102.xml` | `observation` `time` attribute | time | `HH:MM`, 24-hour | `00:00` to `23:50`, every 10 min (144 readings) | Time of the reading. |
| `feed102.xml` | `airTemperature` | decimal | °C (`unit="C"`) | 23.4 to 34.7 | Air temperature. |
| `feed102.xml` | `humidity` | integer | percent (`unit="percent"`) | 44 to 93 | Relative humidity (assumed relative; the file only says "humidity"). |
| `feed102.xml` | `windSpeed` | decimal | mph (`unit="mph"`) | 1.4 to 9.6 | Wind speed. |
| `feed102.xml` | `sunlight` | decimal | W/m² (`unit="W/m2"`) | **Empty in all 144 readings** | Solar radiation. No values were recorded. |
| `batch102.csv` | `Date` | date | `M/D/YYYY` | `7/15/2026` | Date of the reading (month first). |
| `batch102.csv` | `Time` | time | `h:mm AM/PM` | `12:00 AM` to `10:00 PM`, every 2 hours | Time of the reading. |
| `batch102.csv` | `SensorID` | string | `SN-####` | 10 sensors, 12 readings each | Soil sensor ID. |
| `batch102.csv` | `BedID` | string | `BED-#` | `BED-1` to `BED-5`, 2 sensors per bed | Growing bed the sensor is in. |
| `batch102.csv` | `SoilMoisture_VWC` | decimal | volumetric water content, % (unit assumed from the name) | 24.7 to 36.4 | Soil moisture. |
| `batch102.csv` | `SoilTemp_F` | decimal | °F | 70.6 to 92.1 | Soil temperature. |
| `batch102.csv` | `SoilBattery_L` | decimal | unknown (name suggests a battery level) | **Empty in all 120 rows** | Sensor battery level. No values were recorded; the meaning of `_L` is not documented. |
| `stream102.json` | `Log[].date` | date | `MM/DD/YYYY` | `07/15/2026` | Date of the log entry (month first). |
| `stream102.json` | `Log[].time` | time | `h:mm AM/PM` | `5:00 AM` to `5:15 PM`, every 5 min during runs | Time of the log entry. |
| `stream102.json` | `Log[].zone` | integer | zone number | 1, 2, 3 | Irrigation zone. Not linked to `BedID` in any file. |
| `stream102.json` | `Log[].flowRateGpm` | decimal | gallons per minute | 0.0 to 4.35 | Water flow. 0.0 on "off" entries. |
| `stream102.json` | `Log[].pumpStatus` | string | `on` / `off` | `on` (31 entries), `off` (7 entries) | Pump state. One `off` entry closes each of the 7 runs. |
| `stream102.json` | `Log[].activationMode` | string | not documented | **`null` in all 38 entries** | How the pump was started (for example scheduled vs manual). No values were recorded. |

## Fields added in `integrated_102.xlsx`

Sheet `Integrated_10min` has one row per weather reading (144 rows).

| Field | Type | Unit | Derived from | Description |
|---|---|---|---|---|
| `Date` | text | `M/D/YYYY` | all files | Date, in the same style as `batch102.csv`. |
| `Time` | text | `h:mm AM/PM` | all files | Start of the 10-minute slot. |
| `air_temp_C` | decimal | °C | `airTemperature` | Copied as-is. |
| `air_temp_F` | decimal | °F | `airTemperature` | `air_temp_C × 9/5 + 32`, rounded to 1 decimal. Added so it can be compared with soil temperature. |
| `humidity_pct` | integer | % | `humidity` | Copied as-is. |
| `wind_mph` | decimal | mph | `windSpeed` | Copied as-is. |
| `sunlight_Wm2` | decimal | W/m² | `sunlight` | Copied as-is (blank). |
| `irrigating` | text | Yes / No | `pumpStatus` | `Yes` if any `on` log entry falls in the 10-minute slot. |
| `irrigation_zone` | integer | zone | `zone` | Zone of the `on` entries in the slot. Blank when not irrigating. |
| `avg_flow_gpm` | decimal | gpm | `flowRateGpm` | Average of the `on` entries in the slot, rounded to 2 decimals. Blank when not irrigating. |
| `soil_reading_time` | text | `h:mm AM/PM` | `Time` (CSV) | Time of the soil reading used in this row (the most recent one at or before the slot). |
| `moisture_vwc_pct_BED-1` … `BED-5` | decimal | % | `SoilMoisture_VWC` | Average of the 2 sensors in the bed at `soil_reading_time`. |
| `temp_F_BED-1` … `BED-5` | decimal | °F | `SoilTemp_F` | Average of the 2 sensors in the bed at `soil_reading_time`. |

The `Weather`, `Soil`, and `Irrigation` sheets hold every source row with the same dates and times reformatted. Their column names are lower-case versions of the source fields.
