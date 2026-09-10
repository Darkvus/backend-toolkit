---
name: backend-units-of-measurement
description: "Units of measurement standards: meters, cents, seconds, UTC datetime formats. Use when handling measurements or unit conversions in backend code."
---

# Units of Measurement

All usage of units of measurement must follow the data type or format below. If you encounter a unit of measurement that is not present in the following table, you must agree with the rest of the team on the most appropriate data type/format.

| **Magnitude** | **Unit** | **Data Type / Format** |
| --- | --- | --- |
| Distance | Meters (m) | Integer |
| Currency | Cents | Integer |
| Duration | Seconds | Integer |
| Date - Time | UTC | Datetime / **%Y-%m-%dT%H:%M:%SZ** |
| Date | UTC | Date / **%Y-%m-%d** |
| Time | UTC | Time / **%H:%M:%SZ** |
