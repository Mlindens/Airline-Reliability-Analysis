# US Aviation Operational Reliability Analysis

The columns of interest in the dataset are:

* Year, Month, Day of week = Date of flight
* Depdelay15 = Departure delay indicator, 15 minutes or more (1=Yes, 0=No)
* Depdelaynew = Difference in minutes between scheduled and actual departure time. Early departures set to 0
* Op_unique_carrier = An airline code used by the U.S. DOT to identify a carrier
* Arrdelay15 = Arrival delay indicator, 15 minutes or more (1=Yes, 0=No)
* Arrdelaynew = Difference in minutes between scheduled and actual arrival time. Early arrivals set to 0
* Cancelled = Cancelled flight indicator (1=Yes, 0=No)
* Cancellation_code = 	Specifies the reason for cancellation
* Carrierdelay = Carrier delay, in minutes
* Weatherdelay = Weather delay, in minutes
* Securitydelay = Security delay, in minutes
* NASdelay = National air system delay, in minutes
* Lateaircraftdelay = Late aircraft delay, in minutes

It's worth noting that BTS states that the reason a flight is delayed (such as weather, security, etc) is only tracked for a late arrival flight.
If a flight departs late but arrives within 15 minutes of its scheduled arrival time, there will be no information as to why the flight was delayed at departure.

## Cleaning and modeling

<img width="711" height="881" alt="1 Python" src="https://github.com/user-attachments/assets/1aec4482-247b-45b4-b095-605ffdc0464e" />

All data on the Bureau of Transportation Statistics website is provided on a monthly basis and since I have 8 years of data, that is 96 zip files.
After downloading all the ZIP files from the Bureau of Transportation Statistics website, I noticed that all CSV files in the ZIP files had the same name.
I needed a way to merge them all into one .csv file, so I wrote a Python script to do so.
This Python script takes the folder that was entered in the code (which is full of .zip files), opens each .zip file and reads the CSV inside.
It combines all the CSVs into one temporary dataframe and saves the combined data to a single .csv file.

<br>
<br>


| Table/Column Creation | Import Data From .CSV |
|---|---|
| <img width="356" height="744" alt="2 create_table_flightdata" src="https://github.com/user-attachments/assets/0c35a8f2-e6d4-423c-82ad-0d99ce4eaa50" /> | <img width="547" height="115" alt="3 import_data_csv" src="https://github.com/user-attachments/assets/00378a67-14fa-47cc-87d0-ba237acc8dfd" /> |


Once the Python script finished, I had a ~9.7GB .csv file on my hands. I knew that Excel would have issues with a file of this size so I decided to import it to my local PostgreSQL database.
For this, I had to create the table and columns so that I could import the .csv data into my database.

<br>
<br>

| Checking Total Row Count | Top 10 Rows of Data |
|---|---|
| <img width="220" height="74" alt="4 count_query" src="https://github.com/user-attachments/assets/321abb4d-3480-4b30-b5cf-4f8aa3509805" /> | <img width="203" height="97" alt="5 select_query" src="https://github.com/user-attachments/assets/f3340d4c-e03a-4b0b-af4f-cb8e7e3fab38" /> |
<img width="126" height="74" alt="6 results" src="https://github.com/user-attachments/assets/b6f169c5-e3ca-407c-89e0-dd4b4af0233f" />

Time to check our data to make sure the import worked as intended.
 * There are over 52 million rows of data, this should shrink a bit once I filter to only include the major U.S. airlines.
<br>

<img width="1954" height="66" alt="7 results" src="https://github.com/user-attachments/assets/4d842c31-6fd1-4446-be2e-14bde2f7a4d2" />
<img width="968" height="68" alt="8 results" src="https://github.com/user-attachments/assets/7c8bcd9c-4013-4279-aa5c-00f321138a16" />
<img width="1221" height="67" alt="9 results" src="https://github.com/user-attachments/assets/2ab2a02a-21c7-4458-9785-99ba054bc0a5" />

Sample row of data. 
 * I noticed that op_unique_carrier and cancellation_code columns will need to be decoded to know what they represent.

<br>

<img width="773" height="232" alt="10 query" src="https://github.com/user-attachments/assets/1832053e-cfc7-446c-b225-e09269e449a5" />
<img width="1110" height="71" alt="11 results" src="https://github.com/user-attachments/assets/e20e2698-566d-458a-8f88-0026a91c3ea3" />

Since a cancelled flight will have a null value for departure delay and arrival delay (as per BTS instructions), I can check if this lines up with the cancellations.
 * Missing departure delays are roughly consistent with total cancellations, which is expected because many cancelled flights never receive a departure delay record.
 * Missing arrival delays are higher than total cancellations, which suggests that some of these cases may be diverted flights or other flights that never completed arrival as planned.
 * All numbers are within an expected tolerance.

<br>

<img width="769" height="289" alt="12 query" src="https://github.com/user-attachments/assets/a45f8e47-e526-4742-9bf4-ef6571c3ce88" />
<img width="1327" height="71" alt="13 results" src="https://github.com/user-attachments/assets/150639cd-96be-4b97-b5f4-a6a2dbbe6cfa" />

Checking for null values in the remaining columns of interest. A lot of null values in cancellation_code column, time to investigate.

<br>

| Checking combination of cancellation status and nulls | Results |
|---|---|
| <img width="382" height="176" alt="14 query" src="https://github.com/user-attachments/assets/95461889-13e5-4a26-935c-03ed1b53ab55" /> | <img width="406" height="172" alt="15 results" src="https://github.com/user-attachments/assets/bd9828b6-08f4-4ac8-af5e-913c7dc5799e" /> |

Since BTS states that they use codes for cancellation_code to flag why the flight was cancelled, I can use that to get a summary showing every unique combination of cancellation status and code, along with nulls.
* Based on the result, the dataset looks consistent: cancelled flights have a code, and non-cancelled flights do not and instead have a null value.
* We know from a previous query that the total row count is 52,969,707.
  * If we add up all the counts from the above results, 51,865,729 + 276,199 + 466,435 + 121,952 + 239,392 = 52,969,707, we are not missing any rows here.

<br>

| Table/Column Creation | Import Data From .CSV |
|---|---|
| <img width="303" height="120" alt="16 create_table" src="https://github.com/user-attachments/assets/21db8267-a70b-42d0-82fc-e7cf2906adb5" /> | <img width="531" height="116" alt="17 copy_csv" src="https://github.com/user-attachments/assets/09326c87-86c9-4501-b0fc-2645e1676e74" /> |

It's time to decode the op_unique_carrier column. I noticed that BTS has a lookup table that I can use to decode the airlines.
* I'll start by first creating a new table that I can import the lookup table to, and then import the .CSV file into the table.

<br>

| Select Rows of Newly Created Table | 10 Sample Rows |
|---|---|
| <img width="232" height="96" alt="18 query" src="https://github.com/user-attachments/assets/42f34508-1ace-4f4d-b760-4154f71e3ce9" /> | <img width="481" height="292" alt="19 results" src="https://github.com/user-attachments/assets/f32d1b1d-e876-4199-b073-fb6652751e5b" /> |

A sample 10 rows of the carrier lookup table.

<br>

| Checking For Null Values | No Nulls Found |
|---|---|
| <img width="630" height="111" alt="20 null_values" src="https://github.com/user-attachments/assets/63f384e3-2ffd-49af-a98a-f9e226640f55" /> | <img width="257" height="75" alt="21 null_results" src="https://github.com/user-attachments/assets/2840f18d-204d-476f-8738-c95158a7ac4a" /> |

Checking the carrier lookup table for nulls to make sure nothing is missing. 

<br>

| Checking Airline Codes | Airline Code Results |
|---|---|
| <img width="659" height="97" alt="22 airline_codes" src="https://github.com/user-attachments/assets/bd0333c1-a568-4ba4-953e-959c7452aab3" /> | <img width="351" height="268" alt="23 airline_codes_resultgs" src="https://github.com/user-attachments/assets/ac1de3d3-1dd4-4f89-bcdd-8a13e4b25a70" /> |

I checked the airline codes for the 10 major U.S. airlines, but I don't want to assume that BTS uses the same airline codes.
* Checking if BTS uses the same codes as the ones found on Google. Looks good so far!

<br>

<img width="779" height="484" alt="24 creat_view1" src="https://github.com/user-attachments/assets/3fbd30f7-3063-4793-9290-3a23d983e2ee" />

It's time to create a few reference tables that Tableau can use with relevant data that will be needed for the analysis.
* The first reference table contains:
  * Dates (year, month, day of week).
    * I'll be using this to create a risk heatmap of worst months/days for delays.
  * Carrier name.
    * This will be used to create a filter for the heatmap by airline and also a stacked bar chart for airline ranking.
  * Total flights.
    * Will be needed for calculations such as total cancellations / total flights.
  * Total cancelled.
    * This will be used to calculate total cancellation rate for all flights/airlines and per airline.
    * Since cancellation is either a 1 for yes or 0 for no, I can use SUM here to get a total amount of cancelled flights.
  * Departure Delay 15/Arrival Delay 15.
    * Will be used to calculate the total flights delayed by 15 minutes or more.
    * Since both columns are either a 1 for yes or 0 for no, I can use SUM here to get a total amount of delayed flights of 15mins or more.
  * Departure delay new / Arrival delay new.
    * Shows the total flight delay count, regardless of how late the flight was.
    * I used SUM here with CASE WHEN > 0 to get a total count.
    * This will be used in calculations to see how many flights were delayed (and not just by 15mins or more).
  * Departure delay minutes average / Arrival delay minutes average.
    * The average delay for departure and arrival flights.
    * Will be used to show average delays for all airlines.
* A LEFT JOIN was used to connect with the carrier_lookup table to filter for only the major U.S. airlines.

<br>

<img width="599" height="88" alt="25 copy_to_csv" src="https://github.com/user-attachments/assets/0333deb6-f937-46d9-886c-de8d35ff407d" />

Now that the table is complete, I need to export the table to .csv so I can import it in Tableau.

<br>

| Table/Column Creation | Import Data From .CSV |
|---|---|
| <img width="349" height="115" alt="26 create_table" src="https://github.com/user-attachments/assets/9587b5d2-77d1-44bc-b132-0ce537a24685" /> | <img width="506" height="115" alt="27 copy_from_csv" src="https://github.com/user-attachments/assets/5671bcdd-a750-45e8-95e3-1b9ac4cb7346" /> |

Before I create the next reference table, I need to decode the cancellation_code column. BTS have a lookup table for this that I can use to decode.
* Just like the op_unique_carrier_code decoding, I will create a table that can hold the data, and import the look up .csv into that table.

<br>

| Select Rows of Newly Created Table | List of Rows |
|---|---|
| <img width="292" height="78" alt="28 query" src="https://github.com/user-attachments/assets/501e1019-e44d-4e45-a9af-4dc7d593613c" /> | <img width="342" height="148" alt="29 query_r esult" src="https://github.com/user-attachments/assets/29f776f6-5d92-4c1c-a714-91ce63c55827" /> |

We can see that there are 4 different codes as the reasons for the cancellations. 
* This can be used to figure out how many flights were cancelled due to weather, or other reasons.
* This will be applied per airline later to see airline reliability (i.e, how many cancellations due to carrier issues).

<br>

<img width="746" height="545" alt="30 creat_view2" src="https://github.com/user-attachments/assets/5ab1b411-91a4-4ef0-aa0c-6861d0141233" />

Now that the cancellation lookup table is complete, it's time to create another reference table.
This table will be used for the tooltip to provide more information on delay profiles.
Do note that reasons are only tracked for cancelled flights and delayed arrivals, and not delayed departures.
* The cause_breakdown reference table contains:
  * Year.
    * I included year in case I want to be able to filter it by year.
  * Carrier name.
    * Will be used to identify the carrier and see the delay profile of each one individually.
  * Cancellation code.
    * Cancellation reason. This helps identify the most common reasons flights were cancelled.
  * Total reason delay mins.
    * Includes the reasons for the delays, the breakdown is as follows. 
      * Late Aircraft: The cascading ripple effect. A plane was delayed in its previous city, causing it to depart late for its current flight.
      * Carrier Issue: The airline's direct fault (maintenance, crew scheduling, aircraft cleaning).
      * Air System (ATC): Delays caused by the National Aviation System (heavy runway traffic, air traffic control limits, standard non-extreme weather slowing down the flow).
      * Weather: Extreme, localized weather events (blizzards, hurricanes) that prevent safe flying.
      * Security: Delays caused by terminal evacuations, TSA equipment failures, or security breaches.
  * A LEFT JOIN was used to join with the carrier_lookup table to filter for only the major U.S. airlines.
    * I also used a LEFT JOIN on cancellation_code table to get the reasons for the cancellations.
    * Besides filtering for the major U.S. airlines, I also filtered to only include cancelled flights or delayed flights.

<br>

<img width="592" height="96" alt="31 copy_to_csv" src="https://github.com/user-attachments/assets/c7f71a48-0c5f-45f5-aa70-5d87460bbeb9" />

Exporting the reference table to .csv so I can import it to Tableau.

<br>

Now that I have everything I need, I can start importing the data to Tableau to visualize the findings!
