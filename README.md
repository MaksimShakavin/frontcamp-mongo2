### 1.	How many records does each airline class have? Use $project to show result as { class: "Z", total: 999 }
#### Query:
```javascript
db.airlines.aggregate(
  [
    {$group: {_id:'$class', total: {$sum: 1}} },
    {$project:{ _id: 0, "class":"$_id", "total": 1}}
  ],
  {allowDiskUse: true}
)
```
#### Result:
```
{ "total" : 140343, "class" : "F" }
{ "total" : 23123, "class" : "L" }
{ "total" : 5683, "class" : "P" }
{ "total" : 17499, "class" : "G" }
```
### 2.	What are the top 3 destination cities outside of the United States (destCountry field, not included) with the highest average passengers count? Show result as { "avgPassengers" : 2312.380, "city" : "Minsk, Belarus" 
#### Query
```javascript
db.airlines.aggregate(
 [
     {$match:{destCountry:{$not:{$eq:"United States"}}}},
     {$group:{_id:{ destCity:"$destCity"}, avgPassengers:{$avg:"$passengers"}}},
     {$sort:{'avgPassengers':-1}},
     {$limit:3},
     {$project:{_id:0, "avgPassengers":1, "city":"$_id.destCity"}}
   ],
   {allowDiskUse: true}
 )
```
#### Results:
```
{ "avgPassengers" : 8052.380952380952, "city" : "Abu Dhabi, United Arab Emirates" }
{ "avgPassengers" : 7176.596638655462, "city" : "Dubai, United Arab Emirates" }
{ "avgPassengers" : 7103.333333333333, "city" : "Guangzhou, China" }
```
### 3.	Which carriers provide flights to Latvia (destCountry)? Show result as one document { "_id" : "Latvia", "carriers" : [ "carrier1", " carrier2", …] }
#### Query:
```javascript
db.airlines.aggregate(
   [
     {$match: {destCountry: "Latvia"}},
     {$group: {_id: {destCountry: "$destCountry"},carriers: {$addToSet: "$carrier"}}},
     {$project: { _id: "$_id.destCountry",carriers: 1}}
   ],
...
```
#### Results:
```
{ "carriers" : [ "Blue Jet SP Z o o", "Uzbekistan Airways", "JetClub AG" ], "_id" : "Latvia" }
```
### 4.	What are the carriers which flue the most number of passengers from the United State to either Greece, Italy or Spain? Find top 10 carriers, but provide the last 7 carriers (do not include the first 3). Show result as { "_id" : "<carrier>", "total" : 999}
#### Query:
```javascript
db.airlines.aggregate(
  [
    {$match: {destCountry: {$in: ["Italy", "Greece", "Spain"]},originCountry: "United States"}},
    {$group: { _id: {tempCarrier: "$carrier"},totalPassengers: { $sum: "$passengers"}}},
    {$sort: {totalPassengers: -1}},
    {$limit: 10},
    {$project: { _id: "$_id.tempCarrier",total: "$totalPassengers"}},
    {$skip: 3}
  ],
  {allowDiskUse: true}
)
```
#### Results:
```
{ "_id" : "Compagnia Aerea Italiana", "total" : 280256 }
{ "_id" : "United Air Lines Inc.", "total" : 229936 }
{ "_id" : "Emirates", "total" : 100903 }
{ "_id" : "Air Europa", "total" : 94968 }
{ "_id" : "Meridiana S.p.A", "total" : 20308 }
{ "_id" : "Norwegian Air Shuttle ASA", "total" : 13344 }
{ "_id" : "VistaJet Limited", "total" : 183 }
```
### 5.	Find the city (originCity) with the highest sum of passengers for each state (originState) of the United States (originCountry). Provide the city for the first 5 states ordered by state alphabetically (you should see the city for Alaska, Arizona and etc). Show result as { "totalPassengers" : 999, "location" : { "state" : "abc", "city" : "xyz" } } 
#### Query:
```javascript
db.airlines.aggregate(
  [
    {$match: {originCountry: "United States"}},
    {$group: { _id: {originCity: "$originCity",originState: "$originState"},totalPassengers: {$sum: "$passengers"}}},
    {$sort: {"_id.originState": 1,"totalPassengers": -1,}},
    {$group: {_id: {originState: "$_id.originState"},city: {$first: "$_id.originCity"},totalPassengers: { $max: "$totalPassengers"}}},
    {$sort: {"_id.originState": 1 }},
    {$project: {_id: 0,totalPassengers: 1,location: {city: "$city",state: "$_id.originState"}}},
    {$limit: 5}
  ],
  {allowDiskUse: true}
)
```
#### Results:
```
{ "totalPassengers" : 760120, "location" : { "city" : "Birmingham, AL", "state" : "Alabama" } }
{ "totalPassengers" : 1472404, "location" : { "city" : "Anchorage, AK", "state" : "Alaska" } }
{ "totalPassengers" : 13152753, "location" : { "city" : "Phoenix, AZ", "state" : "Arizona" } }
{ "totalPassengers" : 571452, "location" : { "city" : "Little Rock, AR", "state" : "Arkansas" } }
{ "totalPassengers" : 23701556, "location" : { "city" : "Los Angeles, CA", "state" : "California" } }
```
