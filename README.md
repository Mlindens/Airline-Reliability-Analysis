# US Aviation Operational Reliability Analysis

I had a flight cancelled on a recent trip. The airline sent me a text at 3AM and the flight was supposed to leave at 12PM the same day, with the replacement flight leaving the next day. This would not work for me so I had to scramble to find a different flight that day. This made me ask myself, how many flights are cancelled and what do the delays look like for the major airlines in the US?

After doing some research, I found that major airlines (ones with at least 0.5% of total domestic passenger revenue) are required to report flight performance data to the Bureau of Transportation Statistics (BTS). BTS in turn publishes monthly consumer reports which are available to the public.

My Tableau dashboard for this analysis can be found [here](https://public.tableau.com/views/USAviationAnalysis/Overview?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

## The Data
I downloaded the data for 2018-2025 from the Bureau of Transportation Statistics, which resulted in over 52 million rows of data (~36.8 million rows after filtering for only major U.S. airlines).

Columns of interest included:
* Airline Name
* Month
* Day of Week
* Departure delays
* Arrival delays caused by:
  * Weather Delays
  * Airline Issues Delays
  * Security Delays
  * Air System Delays
* Number of Cancellations

You can find the BTS website and dataset [here](https://www.transtats.bts.gov/DL_SelectFields.aspx?gnoyr_VQ=FGJ&QO_fu146_anzr=b0-gvzr).

For this project, I wanted to answer these questions:
1. What is the average departure delay for flights?
2. What are the reasons for the arrival delays?
3. How many flights got cancelled?
4. Which airline is the most reliable in terms of departure delays, arrival delays, and cancellations?
5. Which airline is the least reliable?
6. What do the departure and arrival delays look like on any given weekday for a specific month?

## Cleaning and modeling
Click [here](https://github.com/Mlindens/Airline_Reliability_Analysis/blob/main/Cleaning.md) to see how I cleaned, transformed, and modeled the data.

## Key Insights
* The average departure delay is **15 minutes** and the average arrival delay is **14 minutes**.
* The total delay rate is **34.95%** from over 36.8 million flights.
* The total cancellation rate is **1.94%** out of the 36.8 million flights.
* The most reliable airline is Delta Air Lines with an on time rate of **84%**.
* The least reliable airline is Frontier Airlines with an on time rate of **72%**.
* On average across all airlines, delays are longest in months June and July, Friday - Sunday.

## A Deeper Look

<img width="839" height="103" alt="KPI" src="https://github.com/user-attachments/assets/51c4d8f2-af71-4cd4-9c35-41ab84779769" />


Looking at our KPI chart, we can see that the average departure delay is 15 minutes and the average arrival delay is 14 minutes. 

We can also see that 34.95% of the total flight volume of 36,893,938 flights from 2018-2025 were delayed. That is ~1.6 million delayed flights a year!
If we look at cancellation rate, we can see that it is at 1.94%, which results in 715,742 or about 90,000 cancelled flights a year.
<br>
<br>

<img width="983" height="267" alt="Carrier_reliability" src="https://github.com/user-attachments/assets/10acdc0d-1c78-452d-b70e-30183a69aa90" />

As we can see from the stacked bar chart, Delta Air Lines is the most reliable airline with an ~84% on time ratio for all of their flights.
As for the least reliable airline, Frontier Airlines sits at 72% on time rate. 
<br>
<br>
<br>

Tooltip explanation:
* Late Aircraft: The cascading ripple effect. A plane was delayed in its previous city, causing it to depart late for its current flight.
* Carrier Issue: The airline's direct fault (maintenance, crew scheduling, aircraft cleaning).
* Air System (ATC): Delays caused by the National Aviation System (heavy runway traffic, air traffic control limits, standard non-extreme weather slowing down the flow).
* Weather: Extreme, localized weather events (blizzards, hurricanes) that prevent safe flying.
* Security: Delays caused by terminal evacuations, TSA equipment failures, or security breaches.

<img width="338" height="154" alt="Delta_profile" src="https://github.com/user-attachments/assets/8fe2f0ea-d38b-4f50-829b-6930a569cc1b" />

Note that BTS only tracks reasons for delayed flights (such as weather, security, etc) for late arrival flights, not departure delays.
We can see on the Delta Air Lines arrival delay profile that out of all of their arrival delays:

* Carrier issues made up 44.8%.
* Late aircraft accounted for 28.5% of delays.
* National Aviation System delays represented 21.0%.
* Weather caused 5.6% of delays.
* Security issues accounted for 0.1%.

<br>
<img width="339" height="154" alt="Frontier_profile" src="https://github.com/user-attachments/assets/4690e985-111f-4b04-ae3c-070a09f4d9a7" />

As for Frontier Airlines arrival delay profile:
* Late aircraft accounted for 48.6% of delays.
* Carrier issues made up 29.2%.
* National Aviation System delays represented 20.0%.
* Weather caused 2.2% of delays.

<br>
<br>

<img width="939" height="218" alt="image" src="https://github.com/user-attachments/assets/f8f48acd-c55c-406a-b7e6-5d0547db8309" />

The risk heatmaps show the average delay across all airlines, with filtering options for a specific airline and departure vs arrival delays. On average across all airlines, delays are longest in months June and July, Friday - Sunday.
If we filter this for the previously mentioned airlines, we can see a shift.
<br>
<br>
<br>
<img width="928" height="205" alt="Heatmap_Delta" src="https://github.com/user-attachments/assets/70bcd346-098a-4c60-b28b-f67577a28e89" />

For Delta, we can see that it is mostly the same as all the airline averages previously mentioned, but with a higher average delay in July. Friday delays have an average of 21 minutes, with the weekend going down to 20 and 19, respectively.
Delta has the lowest average delays during the months of September - November, with an average delay of 8 minutes.
<br>
<br>
<br>
<img width="929" height="204" alt="Heatmap_Frontier" src="https://github.com/user-attachments/assets/5ea4c26b-82fe-443d-aede-4e29089e959a" />

For Frontier Airlines, July is the month with the highest average delays, with Sunday having the highest average delay at 33 minutes, followed closely by Saturday at an average of 30 minutes delay.
Saturdays and Sundays in June also peak at a 30 minutes average delay. The lowest average delay month is November, with an average delay of 17 minutes.

Interestingly, for both airlines, the average delayed departure time is very similar to the average delayed arrival time.
This would make sense since if the flight leaves 15 minutes later, it most likely arrives 15 minutes later too.
What this shows is that there is only so much a captain/flight can do to speed up the arrival time if the flight already departed late.
<br>
<br>
<br>

## Conclusion
Thank you for taking the time to read through my analysis. I hope this project provides a clearer view of U.S. airline reliability and the factors that most often affect passengers.
