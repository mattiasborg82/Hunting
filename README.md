# Hunting – Kusto External Data Enrichment

## Overview

**Hunting** provides a set of enrichment datasets designed to be consumed in Kusto using the `externaldata()` function.

## Core Idea

Traditional enrichment in Kusto often requires:
- Ingesting lookup tables
- Maintaining update pipelines
- Managing storage and retention

This repo avoids all of that.

Instead, it uses:

~~~kql
externaldata(...)
~~~

to fetch enrichment datasets directly from remote sources.



## How It Works

At query time, Kusto:
1. Downloads the file (CSV, JSON, etc.)
2. Applies the defined schema
3. Makes it available as a virtual table
4. Lets you join or enrich your data

## Example

### Enrich Data from GitHub


Result message
~~~kql
let EntraMessages = externaldata (ResultType:int, Message:string)
[
@"https://raw.githubusercontent.com/mattiasborg82/Hunting/refs/heads/main/General/EntraResultCodes.csv"
]
with(format="csv",ignoreFirstRecord=true);
EntraIdSignInEvents
| take 100
| join kind=leftouter EntraMessages on $left.ErrorCode == $right.ResultType
~~~

Country Code sample
~~~kql
let CountryCodes = externaldata (Country:string, Code:string)
[
@"https://raw.githubusercontent.com/mattiasborg82/Hunting/refs/heads/main/General/cc.csv"
]
with(format="csv",ignoreFirstRecord=true);
EntraIdSignInEvents
| take 100
| join kind=leftouter CountryCodes on $left.Country == $right.Code
~~~


## What This Repo Contains

The repository includes:

- Static enrichment datasets
  - Ports
  - Country codes to country name
  - EventIds
  - Entra resulttypes
- Files optimized for:
  - `externaldata()`
  - Fast joins in KQL
- Lightweight, portable enrichment sources



## Use Cases exampes

### Port info Enrichment
- Known ports
- Might contain errors (AI)

### Country Code
- Maps list Country Codes (Alpha-2) to name
- [Map Country Code → Country Names](https://www.iban.com/country-codes)
- To be used with any log source containing the short code

and more

### Event Descriptions, Logonstatus and logon types
- Event Ids and description (short list)


### No Ingestion Required
No pipelines, no ingestion jobs, no maintenance overhead.

### Always Up-to-Date
Update the file → instantly reflected in queries.

### Portable
Works across:
- Azure Data Explorer
- Log Analytics
- Microsoft Defender XDR / Sentinel


## Things You Should Know

### Performance Considerations
- `externaldata()` fetches data at query time
- Large files = slower queries
- Best suited for small to medium datasets

## Requirements

- Azure Data Explorer (ADX) or Azure Log Analytics
- Ability to run KQL queries
- Internet access from your Kusto environment

## Contributing

Add new datasets if they are:

- Small and efficient
- Clearly structured
- Useful for enrichment scenarios

Keep things:
- Clean
- Practical
- Fast to use in queries


## Disclaimer
- Validate external sources
- Be aware of query-time data loading
- Don’t rely on this for high-volume production pipelines

## Credit
Fabian Bader for Entra ID table ([Blog post](https://cloudbrothers.info/en/entra-id-azure-ad-signin-errors/))

## Author
Created by Mattias Borg  
Focused on threat hunting, DFIR, and making data actually useful.
