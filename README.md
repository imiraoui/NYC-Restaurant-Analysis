# Analyzing New York City Restaurants' Sanitary Inspections

### Abstract

Using New York City's DOH data and Yelp API, we attempt to predict a restaurant's health grade. As most websites do not provide free access to written reviews, we focus on objective facts around the businesses such as the average rating, the number of reviews, the hours of operations, etc.

Our analysis shows that most variables are uncorrelated with a restaurant's sanitary conditions. However, the chain size and the number of recent inspections seem to play an important role in predicting a restaurant's health grade.

Using a Random Forest, we are able to significantly better predict a restaurant's health grade. The model could be used by the city to customize its inspection schedule and improve its ROI.

We conclude by suggesting policy improvements and future areas of research to further the analysis.

1. [Constructing and Manipulating the Dataset](#part1)
2. [Visualizing the Dataset and Attempting to Predict a Restaurant's Health Grade](#part2)
3. [Guiding Policy and Future Analysis](#part3)


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

<span style="display:block;text-align:center">![Timeline of an Inspection Cycle](https://2.bp.blogspot.com/-U3VR9ypH5tA/XCVaUKSgz4I/AAAAAAAAld8/Dft_vBOI_CoeQmnvBeKSkxj0rId346K3ACLcBGAs/s320/inspection%2Bcycle.PNG)

_Graph as per the DOH official documentation available at (https://www1.nyc.gov/assets/doh/downloads/pdf/rii/inspection-cycle-overview.pdf)_</span>

Given that we already account for a restaurant's inspection record and that we are interested by their current grade, we also only keep the restaurant's latest inspection score for the purpose of our analysis.

### Accounting for Chains

We postulate as well that chains may have standardized processes and more stringent sanitary regulations which could have a positive impact on their grades. For simplicity, we count the number of restaurants with the same names and aggregate the number in a variable.

### Is the Business Primarily Serving Food?

We also hypothesize that restaurants that focus on the preparation of food may be more likely to have unsanitary conditions compared to other types of shops (donuts, coffee shops, etc.). We create a new variable to account for whether the restaurant prepares food and aggregate the different cuisine types into it.

### Handling the Hours of Operations

The way a restaurant remains open may give information about the sanitary conditions of a business (overnight restaurants may be less sanitary as they "never close", restaurants that close between lunch and dinner may clean more, restaurants that do not disclose their hours of operations on Yelp may be run with less attention to details, etc.).

We thus aggregate the result of our parsing into 5 different categories:
1. Open Overnight ("overnight")
2. Closes between Lunch and Dinner ("Closes for Lunch")
3. Open for Long Hours ("Long Hours")
4. Open for Short Hours ("Short Hours")
5. Data not available - not the usual format or not disclosed on Yelp ("N/A")

**After cleaning the dataset, we study ~17,000 restaurants**

## Visualizing the Data and Creating a Model <a name="part2"></a>

### Geographical Location is For the Most Part Irrelevant
<span style="display:block;text-align:center">![Proportion of Restaurants Graded A by Borough](https://3.bp.blogspot.com/-jmUrpE6DCA0/XCYNX0n-cdI/AAAAAAAAlfk/ZgNmbYkCk7kxDR_XHwwGbxanh2FjmikaQCLcBGAs/s320/By%2BBorough.png)</span>

The geographic area does not seem to indicate too much with respect to the sanitary conditions of Restaurants' within a certain area.

<span style="display:block;text-align:center">![Proportion of Restaurants Graded A by Neighborhood](https://4.bp.blogspot.com/-jPyG79mBr-E/XCYmPM6AHZI/AAAAAAAAlgc/aT5Drif0U4siAZ2M9n2yIkiCQUaPm0OlACLcBGAs/s320/Restaurants%2BGraded%2BA%2Bby%2BNeighborhood.png)</span>

While most neighborhoods have generally the same sanitary conditions, we note slight differences depending on the neighborhoods.

**A chloropleth map is also available under "proportion_map.html"**

### An Increasing Number of Violations within a growing Dataset

<span style="display:block;text-align:center">![Violations Overtime](https://4.bp.blogspot.com/-68Fd-cpVNvU/XCYCAWLFpnI/AAAAAAAAles/it_aMNerlSgjq88JLTEQRbHeQu_HjhKmwCLcBGAs/s320/Violations%2Bby%2BYear.png)</span>

The number of violations recorded has increased dramatically since 2014 and the inception of the dataset.

<span style="display:block;text-align:center">![JoyPlot by Year](https://2.bp.blogspot.com/-OxiNKoN1-LY/XCX_Lr_UVoI/AAAAAAAAleM/hUGjX_Eh4rc4kuWVK1rnPB_ulpjo8sh3QCLcBGAs/s320/JoyPlot%2BBy%2BYear.png)</span>

However, the distribution of inspection scores has remained globally the same since 2014 with the median remaining stable at 14.


<span style="display:block;text-align:center">![JoyPlot by Year](https://1.bp.blogspot.com/-p2zq79YbSjs/XCYCZzz9URI/AAAAAAAAle0/ldyynvkcYQ4UL4S9B911xUonheqo5DxHACLcBGAs/s1600/Violations%2Bby%2BMonth.png)</span>

Looking at seasonality effect, we also seem to find a much lower amount of violations in the winter months.

<span style="display:block;text-align:center">![JoyPlot by Year](https://3.bp.blogspot.com/-k-xQfEZwE_4/XCYCaU7cZoI/AAAAAAAAle4/d2V6eh-U8b8fbTEnalBd5tj8BSf4qmYBwCLcBGAs/s320/Score%2Bby%2BMonth.png)</span>

However, distributions intra-months are very similar as well. It does not seem to reveal much about a seasonal difference in restaurants' sanitary conditions but rather the fact that inspections may be less frequent.

### The Type of Business Highlights Relevant Sanitary Differences

<span style="display:block;text-align:center">![Is it food?](https://4.bp.blogspot.com/-UZNJyomqcyY/XCYDRhxouaI/AAAAAAAAlfM/ojBwLAIpQoILDwijqCFTW20O-z6wYLhEQCLcBGAs/s320/Food%2Bvs%2Bno%2BFood.png )</span>

Whether a restaurant serves food or not seems to be a relevant differentiation factor in determining a business' sanitary conditions.

<span style="display:block;text-align:center">![Type of Business](https://3.bp.blogspot.com/-6MVHXQKhCPk/XCYDRg_YLhI/AAAAAAAAlfI/qdnalN0iV6M-WCWwb_kWt1zAgl0QmRcIQCLcBGAs/s1600/Restaurants%2BGraded%2BA%2Bby%2BType.png)</span>

As highlighted by the graph above, we could even refine the analysis as we notice that there is a significant variability in businesses' sanitary conditions depending on their main source of activity more.


| Type of Business                                        | Count  |
|---------------------------------------------------------|--------|
| Bakery                                                  | 506    |
| Bottled beverages, including water, sodas, juices, etc. | 55     |
| Coffee/Tea                                              | 1,236  |
| Delicatessen                                            | 185    |
| Donuts                                                  | 488    |
| Food                                                    | 14,016 |
| Ice Cream, Gelato, Yogurt, Ices                         | 272    |
| Juice, Smoothies, Fruit Salads                          | 223    |
| Other                                                   | 149    |

While some categories only have a few businesses and may be too small to provide a very thorough picture, we can nonetheless infer relevant differences depending on the businesses' sources of activity.

### The Number of Reviews Does Not Inform Much on Inspection scores

<span style="display:block;text-align:center">![Proportion of Restaurants Graded A By Number of Reviews](https://4.bp.blogspot.com/-UvIrOcuBZhc/XCYr0zsFHLI/AAAAAAAAlgo/XzzenToNTuIotZW1UO6h5EpDTPXqB-tkACLcBGAs/s1600/A%2Bby%2Blog%2Bof%2BReview%2BCount.pngg)</span>

When adjusted for the scale, the number of reviews does not seem to indicate much with respect to the sanitary grade of a restaurant.

### The Average Rating on Yelp is Uncorrelated with Inspection Scores

<span style="display:block;text-align:center">![Proportion of Restaurants Graded A By Average Rating](https://4.bp.blogspot.com/-CQrTaOlKR-0/XCYQIk_wetI/AAAAAAAAlgA/Tz71nHxvh0Iechu1HHipCGihDQHFvT9mACLcBGAs/s320/Score%2Bby%2BRating.png)</span>

The average rating on Yelp seems for the most part uncorrelated with inspection results.


### The Price Level Does Not Indicate Much

<span style="display:block;text-align:center">![Proportion of Restaurants Graded A By Price Point](https://2.bp.blogspot.com/-THTbIIZ4pu0/XCYQIJDO8hI/AAAAAAAAlf8/JdP-eBYC-AgSvuUwyAr9lWOw0MwYx_pKgCLcBGAs/s1600/Proportion%2BA%2Bby%2BPrice.png)</span>

The Price level does not seem to indicate any relationship with sanitary conditions aside from the highest price point, where as expected, restaurants tend to have slightly better health grades.

### Hours of Operations Are Mostly Irrelevant

<span style="display:block;text-align:center">![Proportion of Restaurants Graded A By Hours of Operations](https://4.bp.blogspot.com/-mDsw1ghHlvM/XCYQHkqDj_I/AAAAAAAAlfw/BspQvIatTWcf8S073UEiG7o61ZFU1TwggCLcBGAs/s1600/A%2Bby%2Bhours%2Bof%2Boperation.png)</span>

Hours of operations do not seem to matter much as only the restaurants open overnight tend to have a lower likelihood of being graded A.

### Chain size is Positively Correlated with Sanitary Conditions

<span style="display:block;text-align:center">![Proportion of Restaurants Graded A By Chain Size](https://3.bp.blogspot.com/-d4JYvh9ju2Y/XCYQHpi82CI/AAAAAAAAlf4/q9QbVkjLksE_6R1POKbvse7zfImo9JwHgCLcBGAs/s320/A%2Bby%2Bchain%2Bsize.png)</span>

As we had hypothesized, there appears to be some positive correlation between the chain size of the restaurant and the likelihood of the restaurant being rated A.

### Restaurants' Sanitary Issues Tend to be Very Recurrent

<span style="display:block;text-align:center">![Proportion of Restaurants Graded A By Inspection Count](https://2.bp.blogspot.com/-VPDJRCydfho/XCVZoAI6sqI/AAAAAAAAld0/iBHNhqD6AD005dQkJL3BXqjKh4hRML7RACLcBGAs/s320/A%2Bby%2Binspection%2Bcount.png)</span>

The more often a restaurant gets inspected, the less likely the business is to receive an "A" grade. The relationship seems to be much more signifcant as well.

## Guiding Policy And Future Analysis <a name="part3"></a>

### Splitting and Setting Up our Dataset

We split our data 60/20/20 between the Training/Test/Validation set in order to create and test our models.

We standardize the variable related to the size of the chain and the number of reviews.

For the model, we keep the variables relating to the inspection count, the chain size, the type of cuisine, the price point, the neighborhood and whether the restaurant is open overnight.

We set 1 to refer to the restaurant not being graded A and 0 for restaurants graded A.

### A Logistic Regression Does Not Improve the Model Much

<span style="display:block;text-align:center">![Logistic Regression Test Set AUC Curve](https://3.bp.blogspot.com/-eozPCUeDsNY/XCYsyGh8UuI/AAAAAAAAlgw/xlVq1f775f4mb0r5eL3qEA8bJC0zPIYmgCLcBGAs/s320/Log_ROC.png)</span>


### A Random Forest Provides with the Best Results

<span style="display:block;text-align:center">**On the Test Set**

![Random Forest Regression Test Set AUC Curve](https://2.bp.blogspot.com/-D6xz9Fo6Z0Q/XCYsyMZUI9I/AAAAAAAAlg0/l9cz-EHztSw2QeQouc-zypNqDUFv_obxwCLcBGAs/s320/RF_test_ROC.png)</span>

<span style="display:block;text-align:center">**On the Validation Set**

![Random Forest Regression Validation Set AUC Curve](https://4.bp.blogspot.com/-yQlKW4Zu_Lg/XCYtVXVvOhI/AAAAAAAAlhA/NR3ZgKKvg8sHDCvePxY4g4UZac4cO8JsQCLcBGAs/s320/RF_validation_ROC.png)</span>

### Potential Policy Implications

* As the model allows to predict sanitary issues more accurately the City could refine its inspections criteria and reduce the inspection frequency on the low-risk restaurants and focus on the businesses that yield higher risks. It could allow the city to greatly improve its ROI as the DOH could target more efficiently its inspections
  * For Instance, using a threshold of 0.001 on our model, we are able to correctly predict that **~38-39%** of the restaurants will be graded "A" while missing only **~2-3%** of the businesses that would have otherwise received a lower score.
  * Using a threhsold of 0.02 our model, we are able to correctly predict that **~53-55%** of the restaurants will be graded "A" while missing only **~10-12%** of the businesses that would have otherwise received a lower score.

<span style="display:block;text-align:center">![Random Forest Regression Test Set Precision Recall Curves](https://4.bp.blogspot.com/--g5DkETOU3w/XCZCAxBiRII/AAAAAAAAlhY/YOwKqg-MXSgTybstFeorEiDBe_VaX0s7wCLcBGAs/s320/precision-recall.png)</span>

* Aside from the predictive abilities of this model, we saw that restaurants' sanitary conditions tend to be relatively "sticky". as of now, the DOH reserves the right to close a restaurant for scores above 50 or to "to correct a public health hazard that cannot be corrected immediately or when a restaurant is operating without a valid permit". However, many restaurants that have low grades (with scores below 50 but higher than 13) do not correct their scores easily and may need additional incentives/penalties. While the current inspection system pushes restaurants into correcting their sanitary conditions as they will be subject to inspections increasingly frequently (cf. image below), we could suggest alternative mechanisms to remedy better to this issue.
 1. Restaurants could be forced to pay for Food Safety Consultants after a certain amount of inspections within a short period of time to be able to learn and adopt the right behaviors.
 2. An additional penalty could be added after a certain amount of inspections in order to add a financial burden to restaurants that have not improved their sanitary behaviors and give them an additional incentives to change their habits.
    * Alternatively, restaurants could have to pay a fine that grows with the time they have spent below "A".
 3. Make mandatory the use of an external "taskforce" to resolve the sanitary issues after a certain number of inspections.
 4. Restaurants could be forced to close if they have not been able to (re-)establish an A grade following a certain number of inspections.





<span style="display:block;text-align:center">![Inspection Schedule Depending on the Restaurant's Grade](https://1.bp.blogspot.com/-m6OL9KyTegs/XCYwNlgflPI/AAAAAAAAlhM/NwgjCRQVdVQCWIvBi85b_jDmYjurfz0bgCLcBGAs/s320/grade%2Btable.PNG)

_Graph as per the DOH official documentation available at (https://www1.nyc.gov/assets/doh/downloads/pdf/rii/inspection-cycle-overview.pdf)_</span>



### Areas for Future Research

- Mapping the cost of an inspection and the potential revenue that could be derived from restaurants' violations (https://www1.nyc.gov/assets/doh/downloads/pdf/rii/ri-violation-penalty.pdf) in order to set the threshold optimally to maximize revenues and set customized inspection timelines

- Quantifying and optimizing the savings that could be made from the use of the new model in determining the timeframe for each inspections

- While Yelp does not provide free access to the restaurants' reviews, using NLP analysis on written reviews may allow to refine the analysis further and could provide significant improvement to the current model

- Leveraging the Google Places API and additional datapoints on the restaurants such as the presence of a website, the volume of visits,etc. could provide additional insights
