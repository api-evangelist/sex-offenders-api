---
name: search-sex-offender-registry-near-a-location
description: >-
  Find registered sex offender records within a radius of a latitude/longitude point using
  CrimeoMeter's Sex Offenders API, and handle the geographic result set correctly.
api: sex-offenders-api:sex-offenders-api-sex-offenders-api
operations:
  - getSexOffendersByLocation
generated: '2026-08-28'
method: generated
source: >-
  Grounded in openapi/sex-offenders-api-sex-offenders-api-openapi.yml, which transcribes
  CrimeoMeter's public Postman collection (published 2024-07-26).
---

# Search the sex offender registry around a point

## Before you start

Same two gates as the name search: an `x-api-key` issued by hand through
<https://www.crimeometer.com/#contactus>, and personally identifying registry data about
named individuals governed by <https://www.crimeometer.com/tos>. Base URL
`https://api.crimeometer.com/v5`.

Note that this operation **does not appear on CrimeoMeter's product page at all** — it is
published only in their Postman collection. Do not expect the marketing page to describe it.

## Steps

1. **Geocode first.** `getSexOffendersByLocation` takes `lat` and `lon` as decimal degrees.
   It accepts no address, city or zip parameter — if you have an address, resolve it to a
   point before calling. If you only have a zip code, use `getSexOffenderRecords` with
   `zipcode`/`exact_zipcode` instead.

2. **Set the radius in CrimeoMeter's published form.** `distance` is a string of the form
   `<number>mi` — `5mi`, `10mi`, `.25mi` all appear in CrimeoMeter's own published requests.
   It is miles, not kilometres, and there is no unit parameter.

3. **Call it.**

   ```
   GET https://api.crimeometer.com/v5/sex-offenders/location?lat=41.9777476245164&lon=-87.6472903440513&distance=10mi
   x-api-key: <your key>
   ```

4. **Narrow by recency when you are polling.** `last_updated_after` and
   `last_updated_before` accept `YYYY-MM-DD HH:MM:SS` (space-separated — note that
   CrimeoMeter's Crime Data and 911 Calls operations use ISO-8601 with a `T` and `Z`
   instead; the two formats are not interchangeable). Re-querying with
   `last_updated_after` set to your last run is the only change-detection mechanism
   available — there are no webhooks.

5. **Page and parse exactly as in the name search.** Same `sex_offenders_count` /
   `pages_count` envelope, same `page` parameter, same string-encoded booleans and
   semicolon-delimited lists. Use `sex_offender_lat` / `sex_offender_lon` on each record to
   plot or to compute real distances — the API returns matches inside the radius but does
   not return a distance value.

## Errors

`403` `{"message":"Forbidden"}` for a missing, invalid or unentitled key **and** for an
unrecognised path. No rate-limit headers are returned and no limits are published, so back
off conservatively on your own schedule rather than waiting for a `Retry-After` that will
not arrive.
