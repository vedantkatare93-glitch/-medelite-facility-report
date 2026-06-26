# Facility Assessment Report Generator

A lightweight web app built for the Medelite technical case study. You enter a nursing home's CCN, the app pulls that facility's public data from the CMS Provider Data Catalog, combines it with a few manual operational fields, and exports a clean, branded report as a PDF or a Word document.

## Live demo and code

- Live app: https://dapper-zabaione-03ff6b.netlify.app/ 
- Repository: https://github.com/vedantkatare93-glitch/-medelite-facility-report 

## Working
The whole app is a single static file (`index.html`) with no framework and no build step. I kept it deliberately light because it's a single purpose tool, and reaching for something like React would have added complexity for no real benefit at this scope.

For data, it queries the CMS Provider Data Catalog "Provider Information" dataset for nursing homes (dataset id `4pq5-n9py`) through its datastore query endpoint, filtered on the `cms_certification_number_ccn` field. The PDF is generated entirely in the browser with jsPDF and the autotable plugin. The exported report also includes a clickable Medicare Care Compare link that is built dynamically from the entered CCN.

## Field mapping

I used the official CMS Data Dictionary to map each machine readable API field to the clean label on the report:

| Report field | Source | API field |
|---|---|---|
| Name of Facility | CMS API, with manual override | `provider_name` |
| Location | CMS API | `provider_address`, `citytown`, `state`, `zip_code` |
| Census Capacity | CMS API | `number_of_certified_beds` |
| Current Census | CMS API | `average_number_of_residents_per_day` |
| Overall Star Rating | CMS API | `overall_rating` |
| Health Inspection | CMS API | `health_inspection_rating` |
| Staffing | CMS API | `staffing_rating` |
| Quality of Resident Care | CMS API | `qm_rating` |
| EMR, Type of Patient, Previous Coverage, Previous Performance, Medical Coverage | Manual input | none |

## How I validated the data

To make sure the raw government data mapped cleanly without corruption, I pulled a real facility (CCN 015010) and compared the values the app displayed against that facility's actual Medicare Care Compare page, and they matched. The app also uses a small reader function that returns a clean placeholder whenever a value is empty or missing, so blank government fields never render as broken or junk text.

## Bonus features

- A visual five star rating band for the Overall, Health Inspection, Staffing, and Quality ratings, shown as colour coded meters (red for low, amber for middle, green for high) both on screen and in the PDF.
- Current Census is auto filled from the facility's average daily residents, which is what the template's Current Census field represents, and it stays editable.
- A Compliance and Analytics panel that surfaces ownership type, staff turnover, nurse staffing hours, fines, payment denials, special focus status, and the certification date. Rows only appear when data is present, so the panel never fills up with blanks.
- A Word export in addition to the PDF.
- Graceful loading and error states, plus a multi route data fetch that degrades cleanly instead of failing hard.

## Assumptions made

The sample CCN in the brief, 686123, does not return anything from the live CMS API, because it does not match the format of a real certification number. So instead of hardcoding the sample facility, I built the lookup to fail gracefully. If no record comes back, the app shows a clear message, still lets you complete and export the report by hand, and always builds the Care Compare link from whatever CCN was entered.

I also treated the template's Current Census as the CMS value for average number of residents per day, since that is the field it corresponds to. The branding banner is hardcoded as INFINITE, Managed by MEDELITE, and is never overwritten by the facility name. The facility name only appears in the report body.

## On the 12 hospitalization and ED metrics

I chose not to map the 12 claims based hospitalization and ED metrics. Those do not live in the Provider Information dataset. They sit in the separate Medicare Claims Quality Measures and State and US Averages datasets, each with its own measure codes, and I could not fully verify them in the time I had. Since this is a quality focused role, I did not want to put unverified numbers on a report, so I scoped them out and noted them here as a clear next step. Adding them would mean querying the claims dataset by CCN for the short stay and long stay rehospitalization and ED visit measures, and the averages dataset for the matching state and national rows.

## The main technical hurdle

The biggest obstacle was that the CMS API cannot be called directly from the browser, because it blocks cross origin requests. It first showed up as a "Failed to fetch" error, and a public relay I tried as a quick workaround timed out, so it was not reliable. I solved it by routing the request through a same origin Netlify proxy that fetches the data server side and avoids the cross origin problem entirely, with a multi route fallback left in place for resilience.

## Running it

This app is served from a real URL so the proxy can handle the CMS request. Opening the bare file directly from your computer will block the data call, which is expected. Deploy it (Netlify or similar) and the lookup works.
