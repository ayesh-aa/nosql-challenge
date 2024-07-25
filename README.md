
UK Food Establishments Analysis
Project Overview
This project involves analyzing food establishment data from the UK, stored in a MongoDB database. The primary objective is to perform exploratory analysis on the data to answer specific queries related to hygiene scores, rating values, and other criteria.

Prerequisites
Before we begin, ensure we mmust have the following installed:

Python 3.x
MongoDB
PyMongo
Pandas

Start MongoDB:
Ensure your MongoDB server is running. By default, it should run on port 27017.

Load Data into MongoDB:
Import your data into the uk_food database in MongoDB if it's not already done.

Run the Analysis Script:
Execute the script to perform the analysis.


Analysis Script: analysis.py
1. Load the Data from MongoDB
Connect to the MongoDB server and load the establishments collection from the uk_food database.

2. Display the First Document
Fetch and display the first document to understand the structure of the data.

3. Query for Establishments in London with a RatingValue >= 4
Filter and fetch establishments in London with a RatingValue greater than or equal to 4. Display the count and the first document from the results. Convert the results to a Pandas DataFrame and display the first 10 rows.

4. Handle Missing 'Geocode' Information for 'Penang Flavours'
Check if the 'geocode' field exists for the 'Penang Flavours' establishment. If it exists, proceed to query nearby establishments with a rating value of 5 and sort by the hygiene score. Convert the results to a Pandas DataFrame and display the first 10 rows.

5. Count of Establishments with Hygiene Score of 0
Count the number of establishments with a hygiene score of 0 in each Local Authority area using an aggregation pipeline. Convert the results to a Pandas DataFrame and display the first 10 rows.

python

from pymongo import MongoClient
import pandas as pd
from pprint import pprint

# Create an instance of MongoClient
mongo = MongoClient(port=27017)

# Assign the uk_food database to a variable name
db = mongo['uk_food']

# Assign the collection to a variable
establishments = db['establishments']
2. Display the First Document
Fetch and display the first document from the collection to understand its structure.

python

print("First document in the collection:")
first_doc = establishments.find_one()
pprint(first_doc)
print("-" * 40)
3. Query for Establishments in London with a RatingValue >= 4
Filter and fetch establishments in London with a RatingValue greater than or equal to 4, then display the count and the first document.

python

query = {
    'LocalAuthorityName': {'$regex': 'London'},
    'RatingValue': {'$gte': 4}
}

result_count = establishments.count_documents(query)
print(f"No. of Documents in result: {result_count}")
print("-" * 40)

first_result_doc = establishments.find_one(query)
print("First document with RatingValue >= 4 in London:")
pprint(first_result_doc)
print("-" * 40)

data = list(establishments.find(query))
df = pd.DataFrame(data)

print(f"The number of rows in the dataframe is {len(df)}")
print("-" * 40)

print(df.head(10))
4. Handle Missing 'Geocode' Information for 'Penang Flavours'
Check for the existence of 'geocode' information for the 'Penang Flavours' establishment.

python

penang_flavours = establishments.find_one({'BusinessName': 'Penang Flavours'}, {'geocode': 1})

if penang_flavours and 'geocode' in penang_flavours:
    latitude = penang_flavours['geocode']['latitude']
    longitude = penang_flavours['geocode']['longitude']
    print(f"Latitude: {latitude}, Longitude: {longitude}")
else:
    print("Penang Flavours does not have geocode information")
5. Further Analysis if 'Geocode' Exists
Perform further analysis by querying nearby establishments with a rating value of 5, sorted by the hygiene score.

python

if penang_flavours and 'geocode' in penang_flavours:
    degree_search = 0.01

    query = {
        'RatingValue': 5,
        'geocode.latitude': {'$gte': latitude - degree_search, '$lte': latitude + degree_search},
        'geocode.longitude': {'$gte': longitude - degree_search, '$lte': longitude + degree_search}
    }

    sort = [('scores.Hygiene', 1)]
    limit = 5
    fields = {'scores.Hygiene': 1, 'RatingValue': 1, 'BusinessName': 1, 'geocode.latitude': 1, 'geocode.longitude': 1}

    data = list(establishments.find(query, fields).sort(sort).limit(limit))
    pprint(data)

    df = pd.DataFrame(data)
    print(df.head())
else:
    print("Skipping further analysis as geocode information is missing for Penang Flavours")
6. Count of Establishments with Hygiene Score of 0
Count the number of establishments with a hygiene score of 0 in each Local Authority area.

python

pipeline = [
    {'$match': {'scores.Hygiene': 0}},
    {'$group': {'_id': '$LocalAuthorityName', 'count': {'$sum': 1}}},
    {'$sort': {'count': -1}}
]

results = list(establishments.aggregate(pipeline))

print(f"The number of documents in the result: {len(results)}")
print("-" * 40)

print("First 10 results:")
for result in results[:10]:
    pprint(result)
print("-" * 40)

df = pd.DataFrame(results)

print(f"The number of rows in the dataframe is {len(df)}")
print("-" * 40)

print(df.head(10))

Conclusion
This project provides a detailed analysis of UK food establishment data stored in MongoDB. The analysis includes querying for specific hygiene scores, rating values, and other criteria, and presenting the results in a readable format using Pandas DataFrames.


