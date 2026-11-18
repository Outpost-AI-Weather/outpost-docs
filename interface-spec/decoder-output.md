# Decoder Output Format

**Version:** 1.0 (draft)
**Owner:** Ayaan
**Used by:** packet decoder (produces), base station ingest (consumes), mock data script (produces)

## Purpose

Defines the JSON the packet decoder outputs for each LoRa packet. The decoder and the base station must both follow this format exactly.

## Transport

- **JSON Lines:** one JSON object per line, one line per packet, UTF-8
- The mock data script outputs the same format to a `.jsonl` file
- Packets that fail the checksum or have an unknown protocol version are logged and skipped, never output

## Fields

| Field | Type | Source | Notes |
|---|---|---|---|
| `protocol_version` | integer | packet | |
| `node_id` | integer | packet | |
| `sequence` | integer | packet | Used to detect lost and duplicate packets |
| `node_timestamp` | integer or null | packet | Unix time in UTC from the node |
| `received_at` | string | decoder | ISO 8601 UTC, from the base station's clock. Always present. |
| `temperature_c` | number or null | packet ÷ 100 | Signed |
| `humidity_pct` | number or null | packet ÷ 100 | |
| `pressure_pa` | integer or null | packet | |
| `wind_speed_ms` | number or null | packet ÷ 100 | |
| `wind_direction_deg` | number or null | packet ÷ 10 | Null if the sensor isn't installed |
| `battery_mv` | integer or null | packet | |
| `status_flags` | integer | packet | Raw byte, kept for debugging |
| `faults` | list of strings | decoder | Derived from status flags |
| `rssi_dbm` | number or null | receiver | Null until confirmed by hardware |
| `snr_db` | number or null | receiver | Null until confirmed by hardware |
| `raw_payload_hex` | string | decoder | The original 23-byte payload as hex |

Units match the packet specification. Conversion to display units (hPa, km/h) happens in the base station.

## Missing readings

A field holding its type's reserved value is output as `null`.

| Type | Reserved value |
|---|---|
| `int16` | `0x7FFF` |
| `uint16` | `0xFFFF` |
| `uint32` | `0xFFFFFFFF` |

## Status flags

| Bit | Meaning | `faults` value |
|---|---|---|
| 0 | Temperature sensor fault | `temperature` |
| 1 | Humidity sensor fault | `humidity` |
| 2 | Pressure sensor fault | `pressure` |
| 3 | Wind speed sensor fault | `wind_speed` |
| 4 | Wind direction fault or not installed | `wind_direction` |
| 5 | Low battery | `low_battery` |
| 6 | Node clock not synchronized | `clock_not_synced` |
| 7 | Reserved | |

When `clock_not_synced` is present, the base station uses `received_at` instead of `node_timestamp`.

## Example

```json
{"protocol_version": 1, "node_id": 1, "sequence": 1042, "node_timestamp": 1793836800, "received_at": "2026-11-20T12:00:00Z", "temperature_c": -4.25, "humidity_pct": 65.2, "pressure_pa": 101325, "wind_speed_ms": 5.25, "wind_direction_deg": null, "battery_mv": 12500, "status_flags": 16, "faults": ["wind_direction"], "rssi_dbm": null, "snr_db": null, "raw_payload_hex": "0101120400..."}
```

## Changes

Changes to this format follow the packet format process in the Development Workflow Guide: update this document first, then the decoder and base station.
