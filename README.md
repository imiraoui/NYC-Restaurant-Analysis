# Analyzing New York City's Sanitary Inspection

1. [Constructing and manipulating the dataset](#part1)
2. [Visualizing the dataset and attempting to predict a restaurant's health grade](#part2)
3. [Guiding Policy Going Forward](#part3)


## Creating our Dataset <a name="part1"></a>

### An Easy to Access Dataset

Sanitary inspection records are available on **New York City's open data initiative** (https://data.cityofnewyork.us/Health/DOHMH-New-York-City-Restaurant-Inspection-Results/43nn-pn8j) and we pull the most historical records using the API available.

### Matching the Restaurants with Yelp's Information

In "DOH and Yelp Scraping.ipynb", we leverage the **Yelp API** huge dataset to match the various restaurants with Yelp's various informations (hours of operation, review count, average rating,etc.)

_Note - Yelp limits access to it's API to 5,000 requests/day._

### Verifying the Accuracy of the Yelp Data

We use Seatgeek's very own **Fuzzywuzzy** (https://github.com/seatgeek/fuzzywuzzy) to verify that the Yelp businesses matched accurately with the Sanitary inspection data.

### Accounting for the Frequency of Inspections and Keeping a Single Record per Restaurant

Inspections are a lot more frequent for restaurants that have had lower initial scores. Hence, the frequency of inspection gives an indication of a restaurant's health score record.

To account for such a record, we look at the number of inspections a restaurant has had in the past 15 months.

![Timeline of an Inspection Cycle](https://2.bp.blogspot.com/-U3VR9ypH5tA/XCVaUKSgz4I/AAAAAAAAld8/Dft_vBOI_CoeQmnvBeKSkxj0rId346K3ACLcBGAs/s320/inspection%2Bcycle.PNG)

_Graph as per the DOH official documentation available at (https://www1.nyc.gov/assets/doh/downloads/pdf/rii/inspection-cycle-overview.pdf)_

Given that we already account for a restaurant's inspection record and that we are interested by their current grade, we also only keep the restaurant's latest inspection score for the purpose of our analysis.

### Accounting for Chains

We postulate as well that chains may have standardized processes and more stringent sanitary regulations which could have a positive impact on their grades. For simplicity, we count the number of restaurants with the same names and aggregate the number in a variable.

## Visualizing the Data <a name="part2"></a>

### By Boro

### Trend Visualizations

### Cuisine Types

### Review Count

### Inspection Count


![Proportion of Restaurants Graded A By Inspection Count](https://2.bp.blogspot.com/-VPDJRCydfho/XCVZoAI6sqI/AAAAAAAAld0/iBHNhqD6AD005dQkJL3BXqjKh4hRML7RACLcBGAs/s320/A%2Bby%2Binspection%2Bcount.png)

## Guiding Policy <a name="part3"></a>
