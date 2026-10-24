# IoT Data Integration Report: Greenhouse Scenario

## 1. Purpose

Three devices in one deployment each produced a data file for July 15, 2026. This report explains what each file contains, how the three were combined into one dataset (`integrated_102.xlsx`), and what the combined data shows.

## 2. The three sources

| File | Device | Content | Records |
|---|---|---|---|
| `feed102.xml` | Weather station `WS-01` | Air temperature, humidity, wind speed, sunlight | 144 (every 10 min, all day) |
| `batch102.csv` | 10 soil sensors in 5 beds | Soil moisture, soil temperature, battery | 120 (12 per sensor, every 2 h) |
| `stream102.json` | Irrigation controller | Zone, flow rate, pump status, activation mode | 38 (every 5 min during runs) |

Field-level detail is in `DATA_DICTIONARY.md`.

## 3. Integration approach

**Differences found.** The files disagree on three things:
- Date order: the XML is day/month/year; the CSV and JSON are month/day/year.
- Clock: the XML uses 24-hour time; the CSV and JSON use AM/PM.
- Temperature: the XML is in °C; the CSV is in °F.

**Steps.**
1. Parsed each file and converted every date and time to one timestamp.
2. Added `air_temp_F` so air and soil temperature can be compared. The original °C column is kept.
3. Used the weather feed as the timeline, because it has a reading every 10 minutes. That gives 144 rows.
4. Joined irrigation by grouping its 5-minute entries into the 10-minute slot they fall in. A slot is marked `irrigating = Yes` if it contains any pump-on entry, and the flow is the average of those entries.
5. Joined soil by averaging the 2 sensors in each bed, then carrying each 2-hourly reading forward until the next one. A `soil_reading_time` column records which reading each row uses.
6. Kept cleaned copies of all three sources as separate sheets so no original row is lost.

## 4. What the data shows

**Weather.** Air temperature ranged from 23.4 °C (5:40 AM) to 34.7 °C (3:30 PM), averaging 29.0 °C. Humidity ranged from 44% to 93%. Wind speed ranged from 1.4 to 9.6 mph.

**Irrigation.** The pump ran in 7 runs: zones 1, 2, 3 in turn from 5:00 to 7:00 AM; zone 2 alone from 1:05 to 1:25 PM; and zones 1, 2, 3 in turn from 4:00 to 5:15 PM. Average flow while running was about 4.04 gpm in zone 1, 3.88 gpm in zone 2, and 2.20 gpm in zone 3. Flow was lowest at the first entry of each run (for example 3.57 gpm for zone 1 at 5:00 AM) and then rose. Total water use is roughly 533 gallons, estimated by treating each pump-on entry as 5 minutes of flow (the log does not give a volume).

**Soil.** Average moisture across all sensors was:

| Time | 4 AM | 6 AM | 8 AM | 10 AM | 12 PM | 2 PM | 4 PM | 6 PM | 8 PM | 10 PM |
|---|---|---|---|---|---|---|---|---|---|---|
| Moisture (%) | 27.5 | 31.4 | 34.1 | 33.4 | 31.9 | 31.5 | 28.6 | 29.9 | 28.2 | 27.8 |

Moisture rose during the morning irrigation, peaked at 8 AM, and declined through the afternoon. It rose again between 4 PM and 6 PM, during the evening irrigation. Soil temperature ranged from 70.6 °F to 92.1 °F across all beds.

These patterns line up in time with irrigation, but the files alone do not prove cause: with only a reading every 2 hours, and no link between zones and beds, the effect of each zone on each bed cannot be measured.

## 5. Data quality issues and limits

- **Three empty fields.** `sunlight` (all 144 readings), `SoilBattery_L` (all 120 rows), and `activationMode` (all 38 entries) have no values. The reason is not given in the files, so they were left blank and not estimated.
- **No zone-to-bed mapping.** Zones 1 to 3 and beds 1 to 5 cannot be joined. Soil and irrigation are therefore matched by time only.
- **Different reading intervals.** Soil is only measured every 2 hours, so carried-forward values in `Integrated_10min` can be up to 2 hours old.
- **Idle periods are inferred.** The irrigation log has no entries between runs. Slots without an entry are shown as `irrigating = No`, on the assumption that the pump was off.
- **Unclear units.** The CSV gives `SoilMoisture_VWC` without a unit; it is assumed to be a percentage. The meaning of the `_L` suffix in `SoilBattery_L` is not documented.
- **One day only.** Nothing here shows day-to-day variation.

## 6. Conclusion

The three files describe the same day from three angles: outside conditions, soil conditions, and watering. Once dates, times, and units are aligned, they combine into one 10-minute timeline in `Integrated_10min`. The main gaps are the three empty fields and the missing zone-to-bed link; with those, the effect of watering on each bed could be analyzed directly.
