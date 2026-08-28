---
name: search-sex-offender-registry-by-name
description: >-
  Look up registered sex offender records in CrimeoMeter's US registry data by name, alias,
  birthdate or zip code, and read the result set correctly (pagination, exact-vs-fuzzy
  matching, string-encoded booleans).
api: sex-offenders-api:sex-offenders-api-sex-offenders-api
operations:
  - getSexOffenderRecords
generated: '2026-08-28'
method: generated
source: >-
  Grounded in openapi/sex-offenders-api-sex-offenders-api-openapi.yml, which transcribes
  CrimeoMeter's public Postman collection (published 2024-07-26). Every parameter, field and
  error below appears in a CrimeoMeter-published source.
---

# Search the sex offender registry by name

## Before you start

- **You need a key you cannot get yourself.** Authentication is a single `x-api-key`
  request header, and CrimeoMeter issues it manually through the Contact Us form at
  <https://www.crimeometer.com/#contactus>. There is no self-service signup, so an agent
  cannot onboard unattended.
- **This data is about named real people.** Every record carries a birthdate, a home
  address, a photograph and a physical description. Use is bound by CrimeoMeter's Terms of
  Service (<https://www.crimeometer.com/tos>) and the disclaimer linked from the product
  page. Do not cache, redistribute or act on a record without checking those terms.
- Base URL: `https://api.crimeometer.com/v5`. The product page still shows `v3`; the
  published Postman collection shows `v5` with the richer parameter set. Prefer `v5`.

## Steps

1. **Pick the narrowest filter you have.** `getSexOffenderRecords` accepts
   `zipcode`, `exact_zipcode`, `first_name`, `last_name`, `birthdate` (`YYYY-MM-DD`),
   `alias`, `exact_alias`, `last_updated_after` and `last_updated_before`. Combine them —
   `last_name` + `first_name` + `birthdate` is the tightest identity match published, and
   `zipcode` + `last_name` is the tightest geographic one.

2. **Choose exact or fuzzy deliberately.** CrimeoMeter publishes paired parameters:
   `zipcode` / `exact_zipcode` and `alias` / `exact_alias`. The unprefixed form is the
   loose match. If you are confirming an identity rather than exploring, use the `exact_`
   form — the loose form will return neighbours you did not ask for.

3. **Call it.**

   ```
   GET https://api.crimeometer.com/v5/sex-offenders/records?last_name=ANDREWS&first_name=CHRISTOPHER&birthdate=1975-02-11
   x-api-key: <your key>
   Content-Type: application/json
   ```

4. **Page through the results.** The envelope returns `sex_offenders_count` (total matches)
   and `pages_count` (pages available). Pass `page=<n>` to advance. There is **no page-size
   parameter and no published default**, so never assume the first page is the whole answer
   — always compare the length of `sex_offenders[]` against `sex_offenders_count`.

5. **Read the record carefully.** The published shape has three traps:
   - `sex_offender_is_predator` and `sex_offender_is_absconder` are the **strings**
     `"true"` / `"false"`, while `sex_offender_is_synced` is a real boolean. Never test
     these three the same way.
   - `sex_offender_aliases`, `sex_offender_marks` and `sex_offender_charges` are
     **semicolon-delimited strings**, not arrays. Split on `;` yourself.
   - `sex_offender_height` is free text (`"6ft 00in"`) and `sex_offender_weight` is a
     string. Neither is a number with a unit.

6. **Check freshness before you rely on it.** `sex_offender_last_updated`,
   `sex_offender_last_synced` and `sex_offender_is_synced` tell you when CrimeoMeter last
   reconciled the record against the source state registry. A record with
   `sex_offender_is_synced: false` has drifted from its source — say so rather than
   presenting it as current.

## Errors

There is one error envelope and it is not RFC 9457. Every failure is
`403` with `{"message":"Forbidden"}` (or `{"message":"Missing Authentication Token"}` for
an unrecognised path). **A 403 does not distinguish "bad key" from "wrong path"** — the AWS
API Gateway edge returns it for both, and for paths that do not exist at all. If you get a
403, verify the path and version prefix before concluding the key is wrong.

## What you cannot do

The API is read-only — there is no write, no correction, no takedown path. There are no
webhooks and no event stream: to detect change, re-query with `last_updated_after`.
CrimeoMeter's live MCP endpoint (`https://www.crimeometer.com/_api/mcp`) is a Wix website
MCP and **cannot** reach this API; there is no agent-native path to this data.
