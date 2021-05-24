# MTA Turnstile Analysis - Interactive Dashboard for Resource Planning
David Wismer

## Abstract
The goal of this project is to provide insights related to Metropolitan Transportation Authority (MTA) data for turnstile activity. Specifically, my project aims to provide a tool that allows for digging into trends down to the station level, allowing for resource management. Examples might include planning for staffing, adding additional turnstiles at a station, closing down stations during hours of inactivity, etc. I pulled turnstile data from the MTA website, cleaned the data, and organized it in a way that supports analyzing hourly and daily trending. I created high level trending visualizations using Python visualization libraries, but the primary output is a set of interactive Tableau dashboards.

## Design
The key to this project is clean data and standardization. With more reliable data and convenient organization in dashboards, the MTA can answer a variety of questions that will help in resource planning. The format of the original data is inconsistent across different turnstiles and stations. The data also comes in a cumulative format that isn't well suited to analysis of activity over a given period. The crux of the project was converting the data from cumulative audit data taken over varying time periods to hourly entry and exit data for each and every station over the first 5 months of 2021. This standardization would allow for comparing various activity metrics across stations, understanding periods of high and low activity throughout the day, etc.

## Data
I pulled all MTA turnstile datasets containing dates in 2021 from the MTA website. The data was then stored in a SQLite database and queried using the SQL Alchemy Python library. Ultimately, the data included all audits of MTA turnstiles from 12/31/2020 through 5/15/2021. Each database record provided the **cumulative** entries and exits through a given turnstile as of a specified time and date. Each record provided the Control Area, Unit, Subunit Channel Position, and the Station. Each unique combinatioon of these four attributes represents a single turnstile.

## Algorithms

*Data Cleaning and Manipulation*
1. **Reversed Counters** - Some turnstiles are set up incorrectly and begin counting in reverse, as can be seen by cumulative entry and exit data declining over time.
2. **Reset Counters** - In some cases, a turnstile's cumulative count will make a large jump, generally downward, as the counter is reset. This leads to inaccurate calculations when solving for daily/hourly entries and exits.
3. **Large Numbers** - The data has certain records that are simply inaccurate, as show be wildly large entry and exit data relative to the normal level for a given turnstile.

Items 1-3 were resolved by creating a function to identify reverse counting, resets, and unreasonable jumps in the cumulative count for a turnstile.

4. **Cumulative to Periodic Data** - The entry and exit data were converted from cumulative data to periodic data using "shift" and "diff" from the pandas library.
5. **Conversion to Hourly Data** - I created a new datetime column that captured the time and date information for each turnstile audit, rounded to the nearest hour. I then calculated the span of time between audits for a given turnstile and divided the entries and exits for a given period between audits by the number of hours that had passed.
6. **Filling in the Gaps** - I used "product" from the itertools library to generate a dataframe with all possible combinatitons of turnstiles and hours in the period from 1/1/2021 through 5/15/2021 (e.g. 04/06/2021 8:00:00 AM). For all hours with no associated audit, the assumed entries/exits per hour were filled in using the rate from the audit that came next. For example, if I calculated 100 entries per hour for a particular turnstile over the period of 4AM to 8AM on a given date, the final dataframe would show 100 in the "entries per hour" column for the hours beginning at 4AM, 5AM, 6AM, and 7AM.
7. **Outliers** - I examined the histograms of the hourly data and daily data. I began by assuming an upper limit based on a theoretical maximum of 2 entrants to a turnstile per second. I then reduced the upper limit until the histograms appeared reasonable. The final histograms had very long right tails, which is reasonable given that specific events could result in significantly larger volume than normal.

*Tableau Preparation*
1. I removed all columns that would not serve as measures or filters in the Tableau dashboards.
2. I added date columns in readable format for use in filtering.
3. I computed "Net Flow" (entries - exits) and "Total Traffic" (entries + exits) as additional measures to be analyzed in Tableau.
4. I dropped the first two days from the dataset to avoid any calculation issues associated with "shift" and "diff."

## Tools
- SQLite and SQL Alchemy for data storage and querying.
- Numpy, Pandas, and Itertools for data manipulation.
- Matplotlib and Seaborn for plotting and visualization.
- Tableau for interactive visualizations.

## Communication
My high-level analysis will be presented in a slideshow using visualization created with Matplotlib and Seaborn. The set of Tableau dashboards I created for further analysis can be found [here](https://public.tableau.com/profile/david.wismer#!/vizhome/MTATurnstileAnalysis-MetisBootcamp/TrafficHeatmap).

Note that there are three dashboards worth exploring. This link takes you to the first dashboard, the Traffic Heatmap. Scroll to the bottom of the screen and click on "Station Comparison" and "Time Series" to explore further. You can also download the workbook and view it in the Tableau Public desktop app.

