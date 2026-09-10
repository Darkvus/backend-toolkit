---
name: backend-encoding
description: "Text encoding standards for backend services: country, language, currency, timezone, and coordinate formats. Use when configuring encoding or handling data formats."
---

# Encoding

All encoding usage must follow the standards below. If you encounter a field that requires encoding and is not present in the following table, it must be agreed upon with the rest of the team.

| **Magnitude** | **Standard** |
| --- | --- |
| Country | Alpha-2 Code → [ISO 3166](https://www.iso.org/iso-3166-country-codes.html) |
| Language | 639-1 Code → [ISO 639](https://www.iso.org/iso-639-language-codes.html) |
| Currency | [ISO 4217](https://www.iso.org/iso-4217-currency-codes.html) |
| Timezone | IANA Time Zone Database |
| Latitude/Longitude | Float |
