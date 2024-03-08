# gds_assessment
Github repository for (GDS) ACE assessment 

Require: Your directory should look like this
```
gds_assessement
├── input
│   ├── 
│   │   ├── Country-Code.xlsx
│   │   ├── restaurant_data.json
│
├── notebooks
│   ├── analysis.ipynb
│
├── output
```

How to run the code

Prerequisites
Ensure Python is installed
If not, go to the following link to get the installer
https://www.python.org/downloads/

Ensure pandas to installed on your system
If not installed run the following command in your command terminal
```
pip install pandas
```
All required data and code files are in this repository

Data loader code
```python
import pandas as pd

#load the restaurant data from JSON
restaurant_data = pd.read_json(r'input\restaurant_data.json')

#display the first few rows to understand the structure
restaurant_data.head()

#load country code data
country_codes = pd.read_excel(r'input\Country-Code.xlsx')

#display the first few rows to understand the structure
country_codes.head()
```
Output
```
| Index | Results Found | Results Start | Results Shown | restaurants                                      |
|-------|---------------|---------------|---------------|--------------------------------------------------|
| 0     | 29287         | 1             | 20            | [{'restaurant': {'R': {'res_id': 18649486}, 'a...|
| 1     | 7625          | 1             | 20            | [{'restaurant': {'R': {'res_id': 18707652}, 'a...|
| 2     | 21776         | 1             | 20            | [{'restaurant': {'R': {'res_id': 18392725}, 'a...|
| 3     | 16762         | 1             | 20            | [{'restaurant': {'R': {'res_id': 58882}, 'apik...|
| 4     | 12026         | 1             | 20            | [{'restaurant': {'R': {'res_id': 18893197}, 'a...|

| Index | Country Code | Country    |
|-------|--------------|------------|
| 0     | 1            | India      |
| 1     | 14           | Australia  |
| 2     | 30           | Brazil     |
| 3     | 37           | Canada     |
| 4     | 94           | Indonesia  |

```
Task 1 Part 1
Extract the following fields and store the data as restaurants.csv.
* Restaurant Id
* Restaurant Name
* Country
* City
* User Rating Votes
* User Aggregate Rating (in float)
* Cuisines

Solution
```python
def normalise_restaurants(restaurants, country_codes):
    #to store data
    normalised_data = []
    #iterate through "restuarants"
    for restaurant_list in restaurants['restaurants']:
        #for each restaurant
        for restaurant in restaurant_list:
            restaurant_info = restaurant['restaurant']
            restaurant_json_country_code = restaurant_info['location']['country_id']
            #match restaurant_json_country_code with Country-Code.xlsx
            country_name = country_codes[country_codes['Country Code'] == restaurant_json_country_code]['Country'].values[0] if restaurant_json_country_code in country_codes['Country Code'].values else 'Unknown'

            normalised_data.append({
                'Restaurant Id': restaurant_info['R']['res_id'],
                'Restaurant Name': restaurant_info['name'],
                'Country': country_name,
                'City': restaurant_info['location']['city'],
                'User Rating Votes': restaurant_info['user_rating']['votes'],
                'User Aggregate Rating': float(restaurant_info['user_rating']['aggregate_rating']),
                'Cuisines': restaurant_info['cuisines']
            })

    return pd.DataFrame(normalised_data)
```
This will extract all the relevant data and format it to be saved to a csv

```python
normalised_restaurants = normalise_restaurants(restaurant_data, country_codes)
#check structure
normalised_restaurants.head()
```
Output
```
| Index | Restaurant Id | Restaurant Name         | Country | City      | User Rating Votes | User Aggregate Rating | Cuisines                                          |
|-------|---------------|-------------------------|---------|-----------|--------------------|-----------------------|--------------------------------------------------|
| 0     | 18649486      | The Drunken Botanist    | India   | Gurgaon   | 4765               | 4.4                   | Continental, Italian, North Indian, Chinese      |
| 1     | 308322        | Hauz Khas Social        | India   | New Delhi | 13627              | 4.6                   | Continental, American, Asian, North Indian, Ch...|
| 2     | 18856789      | AIR- An Ivory Region    | India   | New Delhi | 1819               | 4.1                   | North Indian, Chinese, Continental, Asian        |
| 3     | 307374        | AMA Cafe                | India   | New Delhi | 3252               | 4.4                   | Cafe, Juices                                     |
| 4     | 18238278      | Tamasha                 | India   | New Delhi | 8112               | 4.4                   | Finger Food, North Indian, Continental, Italian  |
```
Code to save the above output to csv
```python
# Save data to CSV
normalised_restaurants.to_csv(r'output\restaurants.csv', index=False)
```

Task 1 Part 2
Extract the list of restaurants that have past event in the month of April 2019 and store the data as restaurant_events.csv.
* Event Id
* Restaurant Id
* Restaurant Name
* Photo URL
* Event Title
* Event Start Date
* Event End Date

Solution
```python
from datetime import datetime
def normalise_events(restaurants):
    #store data
    events_data = []
    #make april in datetime
    start_april_2019 = datetime(2019, 4, 1)
    end_april_2019 = datetime(2019, 4, 30)
    #same iteration as above
    for restaurant_list in restaurants['restaurants']:
        for restaurant in restaurant_list:
            restaurant_info = restaurant['restaurant']
            if 'zomato_events' in restaurant_info:#check if there was an event
                for event in restaurant_info['zomato_events']:
                    event_info = event['event']
                    #format dates
                    start_date = pd.to_datetime(event_info['start_date'])
                    end_date = pd.to_datetime(event_info['end_date'])
                    #events that occurred in April 2019
                    #this mean so long as there was an event during april 2019, it counts
                    if start_date <= start_april_2019 and end_date >= end_april_2019:
                        events_data.append({
                            'Event Id': event_info['event_id'],
                            'Restaurant Id': restaurant_info['R']['res_id'],
                            'Restaurant Name': restaurant_info['name'],
                            'Photo URL': event_info['photos'][0]['photo']['url'] if event_info['photos'] else 'NA',
                            'Event Title': event_info['title'],
                            'Event Start Date': event_info['start_date'],
                            'Event End Date': event_info['end_date'],
                        })

    return pd.DataFrame(events_data)
```
Filters by events and if the event was occuring during April 2019

```python
events_normalised = normalise_events(restaurant_data)
events_normalised.head()
print(events_normalised)
```
Output
```
| Event Id | Restaurant Id | Restaurant Name       | Photo URL                                          | Event Title                                         | Event Start Date | Event End Date |
|----------|---------------|-----------------------|----------------------------------------------------|-----------------------------------------------------|------------------|----------------|
| 322331   | 18649486      | The Drunken Botanist  | ![Link](https://b.zmtcdn.com/data/zomato_events/...) | BackToBasic Wednesdays !!                           | 2019-03-06       | 2019-08-28     |
| 332812   | 308322        | Hauz Khas Social      | ![Link](https://b.zmtcdn.com/data/zomato_events/...) | Live 20/20 Match Screenings                         | 2019-03-29       | 2019-05-23     |
| 116096   | 18466951      | Jungle Jamboree       | ![Link](https://b.zmtcdn.com/data/zomato_events/...) | Festive Food Bonanza || Special Buffet Prices ...   | 2019-01-21       | 2019-05-31     |
| 161417   | 18466951      | Jungle Jamboree       | ![Link](https://b.zmtcdn.com/data/zomato_events/...) | 7 Course High Tea Birthday & Kitty Parties @ R...   | 2019-01-21       | 2019-05-31     |
| 332812   | 18246991      | Odeon Social          | ![Link](https://b.zmtcdn.com/data/zomato_events/...) | Live 20/20 Match Screenings                         | 2019-03-29       | 2019-05-23     |
```
Note, table above is truncated and is not an exact match to the actual output

Code to save output to csv
```python
# Save the events data to CSV
events_normalised.to_csv(r'output\restaurant_events.csv', index=False)
```

Task 1 Part 3
From the dataset (restaurant_data.json), determine the threshold for the different rating text based on aggregate rating. Return aggregates for the following ratings only:
* Excellent
* Very Good
* Good
* Average
* Poor

```python
def rating_thresholds(restaurants):
    #flatten the restaurant data to easily access user ratings
    flat_data = []
    for restaurant_list in restaurants['restaurants']:
        for rest in restaurant_list:
            rest_info = rest['restaurant']
            flat_data.append({
                'User Aggregate Rating': float(rest_info['user_rating']['aggregate_rating']),
                'Rating Text': rest_info['user_rating']['rating_text']
            })
    
    ratings_df = pd.DataFrame(flat_data)
    
    #filter for the specified ratings
    specified_ratings = ['Excellent', 'Very Good', 'Good', 'Average', 'Poor']
    filtered_ratings = ratings_df[ratings_df['Rating Text'].isin(specified_ratings)]
    
    #calculate thresholds
    groupings = filtered_ratings.groupby('Rating Text')
    thresholds = groupings['User Aggregate Rating'].agg(['min', 'max']).reset_index()
    rating_order = {rating: i for i, rating in enumerate(specified_ratings)} #to make it follow the order of the task
    thresholds['Rating Order'] = thresholds['Rating Text'].map(rating_order)
    ordered_thresholds = thresholds.sort_values('Rating Order').drop('Rating Order', axis=1).reset_index(drop=True)
    
    return ordered_thresholds
```
Uses pandas methods to perform aggregation

```python
rating_thresholds = rating_thresholds(restaurant_data)
print(rating_thresholds)
```

Output
```
| Rating Text | Min | Max |
|-------------|-----|-----|
| Excellent   | 4.5 | 4.9 |
| Very Good   | 4.0 | 4.4 |
| Good        | 3.5 | 3.9 |
| Average     | 2.5 | 3.4 |
| Poor        | 2.2 | 2.2 |
```

