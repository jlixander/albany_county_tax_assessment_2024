# Analysis of Property Assessment Change in Albany, New York

## Introduction/Objective

In May 2024, Albany County residents received a letter stating that a new property assessment had been conducted. The letter outlined changes in the assessed value of the property and the impact it will have on property taxes for the upcoming fiscal year. While some residents have been fortunate to pay the same or lower tax amounts, many have been left with jaw-dropping increases for their tax bills. 

In particular, this analysis aims to better understand the hidden nuances that are not direclty revealed by looking at individual properties. This study will aim to aggregate data across many different categories. These include, but is not limited to: zip code, neighborhoods, and school district. The study will also compare newly assessed property values with nearby recently sold homes data. Ultimately, the goal of this study is to verify if the new property assessments have been conducted fairly and equitably.

This study is significant because it can economically impact Albany Residents. For homeowners, this can mean 1) overpaying property and school taxes on an overvalued home, 2) Increases in home insurance premiums and 3) impact the actual market value of the property. For renters, this can potentially mean increased rent prices through __cost-passthrough__.

## Description

#### Research Questions

There are two questions to formulate hypothesis' from:


1. Did property assessments increase at the same percent rate by neighborhood? By Zip Code? By School District? Other segmentations?
    - Null Hypothesis (H0): There is no significant difference in property tax increase by geographical area.
    - Alternative Hypothesis (H1): There is significant difference in property tax increase by geographical area.

2. If property taxes did not increase at the same rate, was it different for low-income communities? _(Low-Income Communities to be identified using Census Data)_
    - Null Hypothesis (H0): There is no significant difference in property tax increase.
    - Alternative Hypothesis (H1): There is significant difference in property tax increase.   

3. If the null hypothesis is accepted for question 1 and 2. How does the city of albany property tax changes look like compared to surrounding towns in the County of Albany? 



#### About The Data

The main data used for this study are the 2023 and 2024 assessment rolls, which are readily available to the public in the Albany County webiste. In addition, recently sold property data will be used to compare against. Lastly, Census data will be used to identify high-poverty areas. 

* 2023: https://www.albanyny.gov/DocumentCenter/View/8661/2023-Final-Assessment-Roll
* 2023: https://www.albanycountyny.gov/home/showpublisheddocument/47859/638568204642200000
* Residential Sales: https://www.albanyny.gov/DocumentCenter/View/8452/Residential-Sales
* Census: https://www.census.gov/programs-surveys/acs/data/data-via-api.html

__Limitations of the data:__
The parsing method used in this study failed to capture every individual parcel in the tax assesment rolls. Consequently, only detectable parcels that appear in both the 2023 and 2024 results are included in the analysis. In addition, data exploration revealed that some detected parcels did not return other field values such as *Full Market Value* and *School Tax Value*. The total amount of parcels yielded by the parsing script is 28,963. Of these, 159 observations were removed due to lack of missing values. 
  
This approach ensures a consistent basis for comparing data across different segments and years, allowing for more accurate and reliable comparisons between populations and time periods. However, it is unknown whether the missing observations are due to an inherent property characteristic which makes it difficult for the parsing methodology to capture. As such, there may be some bias in this analysis.

The analyis will be updated if tabular datasets for both 2023 and 2024 property assessement rolls are made public.

#### Background Information: High Poverty Areas

The United States Bureau defines high-poverty areas as geographically concentrated zones where 20% or more of the population is living at or under poverty thresholds. Poverty thresholds are defined based on a combination of household income and family size. As of 2023, the thresholds are as follows:

| Size of family unit                | None   | One    | Two    | Three  | Four   | Five   | Six    | Seven  | Eight or more |
|------------------------------------|--------|--------|--------|--------|--------|--------|--------|--------|---------------|
| One person (unrelated individual): |        |        |        |        |        |        |        |        |               |
| - Under 65 years                   | 15,852 |        |        |        |        |        |        |        |               |
| - 65 years and over                | 14,614 |        |        |        |        |        |        |        |               |
| Two people:                        |        |        |        |        |        |        |        |        |               |
| - Householder under 65 years       | 20,404 | 21,002 |        |        |        |        |        |        |               |
| - Householder 65 years and over    | 18,418 | 20,923 |        |        |        |        |        |        |               |
| Three people                       | 23,834 | 24,526 | 24,549 |        |        |        |        |        |               |
| Four people                        | 31,428 | 31,742 | 30,900 | 31,008 |        |        |        |        |               |
| Five people                        | 37,091 | 38,452 | 37,275 | 36,363 | 35,807 |        |        |        |               |
| Six people                         | 43,593 | 43,766 | 42,743 | 41,999 | 40,714 | 39,952 |        |        |               |
| Seven people                       | 50,159 | 50,472 | 49,393 | 48,604 | 47,238 | 45,602 | 43,808 |        |               |
| Eight people                       | 56,099 | 56,594 | 55,575 | 54,683 | 53,416 | 51,809 | 50,136 | 49,710 |               |
| Nine people or more                | 67,483 | 67,810 | 66,908 | 66,151 | 64,908 | 63,198 | 61,651 | 61,268 | 58,907        |

Source: [U.S. Census Bureau, 2024](https://www2.census.gov/programs-surveys/cps/tables/time-series/historical-poverty-thresholds/thresh23.xlsx)


#### Methods Used

* Inferential Statistics
* Data Visualization
* Mapping

#### Technologies

* Python
* Pandas/GeoPandas
* Ploty
* Matplotlib

## Data Transformation and Calculations

#### PDF Data Parsing

To extract information from the assessment roll PDF documents, regular expressions were employed. To start, each PDF document was loaded and converted to a single string text. Once in text format, each page header was removed, followed by the delimiting of each parcel record. The optimal delimiter was identified to be the parcel number embedded in a full text line of asterisks.

From there, each piece of information was extracted using the following regular expression patterns:
| Pattern Name               | Regular Expression Pattern                                                                                         | Description                      |
|----------------------------|--------------------------------------------------------------------------------------------------------------------|----------------------------------|
| `parcelnum_pattern`        | `(?:\d{1,2}\.\d{1,2}-\d*?-)(?:(?:\d{1,2}\.\d{1,2})\|(?:\d{1,2}))`                                                   | Finds unique parcel identifier       |
| `address_pattern`          | `.+\s+(?=(?:HOMESTEAD PARCEL\|NON-HOMESTEAD PARCEL))`                                                               | Finds Property Address            |
| `property_type_pattern`    | `(?:(?:\d{1,2}\.\d{1,2}-\d*?-)(?:(?:\d{1,2}\.\d{1,2})\|(?:\d{1,2})))(?: \\\\n \\\\n +)(\d{3} )`                     | Finds Property Type Code (e.g.: 1 Family Res)      |
| `county_tax_pattern`       | `(?:COUNTY TAXABLE VALUE\s+)(\b(\d{1,4}(,\d{3})+)\b\|[1-9]\d+\|0)`                                                   | Finds the assigned County Taxable Value   |
| `city_tax_pattern`         | `(?:CITY TAXABLE VALUE\s+)(\b(\d{1,4}(,\d{3})+)\b\|[1-9]\d+\|0)`                                                     | Finds the assigned City Taxable Value     |
| `full_market_value_pattern`| `(?:FULL MARKET VALUE\s+)(\b(\d{1,4}(,\d{3})+)\b\|[1-9]\d+)`                                                        | Finds the assigned Full Market Property Value  |
| `school_tax_pattern`       | `(?:SCHOOL TAXABLE VALUE\s+)(\b(\d{1,4}(,\d{3})+)\b\|[1-9]\d+\|0)`                                                   | Finds the assigned County Taxable Value   |

Lastly, both 2023 and 2024 were merged together using an inner join on the parcel number. Essentially, an __inner join__ yields a combined dataset where only records found in both 2023 and 2024 are returned. See the *About The Data* section for an explanation of possible limitations of this approach.

#### Handling Missing Values

For data analysis to be accurate, missing values have to be handled. This allows for appropriate calculations of derived values and for a fair comparison across time periods. No missing values were found, however, around 540 observations where found to have one or more empty string column values. A fraction of these observations were dropped, leaving behind 352 observations with missing 'Property Type Code', which identifies the type of parcel (E.g.: 1 Family Res).

#### Computing Percent Change In Property Values

To better understand the differences in the property assessment rolls, it is prudent to understand magnitude of change for *City Tax*, *County Tax*, *Schol Tax*, and *Full Market Value*. Discrete magnitudes (Thousands of dollar Increase/Decrease) does not provide a clear comparison relative to other properties. As such, the percent change is calculated. This provides a standard scale to compare across parcels in an intuitive way. For example, if the initial home assessment was $100k and is now $140k, this means a 40% increase. If we compare it to a $600K property, a $40k increase would only mean a 6.67% increase. This clearly shows that the former had a much higher assessment hike!

Percent Change is calculated using the formula below:

$$
\text{Percentage Change} = \left( \frac{\text{Value in 2024} - \text{Value in 2023}}{\text{Value in 2023}} \right) \times 100
$$

## Findings


## Conclusion