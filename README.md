# Wrangle OpenStreetMap Data - New York

## Project Overview

You will choose any area of the world in https://www.openstreetmap.org and use data munging techniques, such as assessing the quality of the data for validity, accuracy, completeness, consistency and uniformity, to clean the OpenStreetMap data for a part of the world that you care about. Finally, you will choose either MongoDB or SQL as the data schema to complete your project.

## What will I learn?

After completing the project, you will be able to:

- Assess the quality of the data for validity, accuracy, completeness, consistency and uniformity.
- Parsing and gather data from popular file formats such as .json, .xml, .csv, .html
- Process data from many files and very large files that can be cleaned with spreadsheet programs.
- Learn how to store, query, and aggregate data using MongoDB or SQL.

## Map Area

New York City, NY, Unisted States

I am living in New York now. As the largest city in the world, I’m interested to see how does the database look like, and I’d like an opportunity to contribute to its improvement this map.

![New York OpenStree Map]()

Dateset: [XML Data Source](https://mapzen.com/data/metro-extracts/metro/new-york_new-york/)

## Overview of the Data

We first use the `getsample.py` to only extract 1/100 data from the whole data set. The sample dataset is stroed in `newyork_sample.osm` and is about 270 MB.

### Data File Size

```
new-york_new-york.osm ... 2.6 GB
newyork_sample.osm ...... 270 MB
nodes.csv ............... 97.5 MB
nodes_tags.csv .......... 2.7 MB
ways.csv ................ 11.1 MB
ways_tags.csv ........... 27.8 MB
ways_nodes.cv ........... 33.1 MB
newyork.db .............. 138 MB
```

### Tag Nmae

```
'member': 14703
'nd': 1450874
'node': 1144670
'osm': 1
'relation': 909
'tag': 967033
'way': 179340
```

### Check Tag Key Type

We first check the "k" value for each tag and see if there are any potential problems. I proposed 3 regular expressions to check for certain patterns in the "k" value.

- "lower", for tags that contain only lowercase letters and are valid
- "lower_colon", for otherwise valid tags with a colon in their names
- "problemchars", for tags with problematic characters
- "other", for other tags that do not fall into the other three categorie

```
'lower': 375253 
'lower_colon': 578910
'other': 10370
'problemchars': 2500
```

## Unique Users

`len(users): 2428`

## Problems encountered in the Map Data

After runing some code(in the jupyter notebook file) on the `newyork_sample.osm` dataset, I noticed five main problems with the data:

- Different styles of steet names
- Different styles of postal codes

### Street Names

- Abbreviation *(Ave., Ave, St., St, Blvd, Pkwt)
- Capital      *(AVenue, avenue, CIRCLE)*
- Wrong spell  *(Avene)*
- Wrong fields *(New York)* 

We design a mapping dicnary that will replace all the problematic street name with the full standard name.

```
mapping = { "AVENUE": "Avenue",
            "AVenue": "Avenue",
            "Ave": "Avenue",
            "Ave.": "Avenue",
            "Avene": "Avenue",
            "Blvd": "Boulevard",
            "Ct": "Court",
            "DRIVE": "Drive",
            "Dr": "Drive",
            "Pkwy": "Parkway",
            "Plz": "Plaza",
            "ROAD": "Road",
            "Rd": "Road",
            "STREET": "Street",            
            "St": "Street",
            "St.": "Street",
            "Steet": "Street",
            "Trce": "Terrace",
            "Tpke": "Turnpike",
            "avenue": "Avenue",
            "street": "Street"
            }
```

### Postcodes

New York State postcodes range from 00051 to 14925. So we will exame any postcodes that don't fall in this range.

Here are some problems with the postcode field:


- Doesn't give a specific postcode: *08854-5627*
- Not in the range: *100014, 320*
- With State name: *CT 06870, NY 10533*

I will change all the wrong postcode to the New York default postcode 10001.

## Generate the CSV Database

If the element top level tag is "node":
The dictionary returned should have the format {"node": .., "node_tags": ...}

The "node" field should hold a dictionary of the following top level node attributes:

- id
- user
- uid
- version
- lat
- lon
- timestamp
- changeset

All other attributes can be ignored.

The "node_tags" field should hold a list of dictionaries, one per secondary tag. Secondary tags are
child tags of node which have the tag name/type: "tag". Each dictionary should have the following fields from the secondary tag attributes:

- id: the top level node id attribute value
- key: the full tag "k" attribute value if no colon is present or the characters after the colon if one is.
- value: the tag "v" attribute value
- type: either the characters before the colon in the tag "k" value or "regular" if a colon is not present.

Additionally,

- if the tag "k" value contains problematic characters, the tag should be ignored
- if the tag "k" value contains a ":" the characters before the ":" should be set as the tag type and characters after the ":" should be set as the tag key
- if there are additional ":" in the "k" value they and they should be ignored and kept as part of the tag key. For example:
```
  <tag k="addr:street:name" v="Lincoln"/>
```
  should be turned into:
```
  {'id': 12345, 'key': 'street:name', 'value': 'Lincoln', 'type': 'addr'}
```
- If a node has no secondary tags then the "node_tags" field should just contain an empty list.

The final return value for a "node" element should look something like:
```
{'node': {'id': 757860928,
          'user': 'uboot',
          'uid': 26299,
          'version': '2',
          'lat': 41.9747374,
          'lon': -87.6920102,
          'timestamp': '2010-07-22T16:16:51Z',
          'changeset': 5288876},

 'node_tags': [{'id': 757860928,
                'key': 'amenity',
                'value': 'fast_food',
                'type': 'regular'},
               {'id': 757860928,
                'key': 'cuisine',
                'value': 'sausage',
                'type': 'regular'},
               {'id': 757860928,
                'key': 'name',
                'value': "Shelly's Tasty Freeze",
                'type': 'regular'}]}
```
If the element top level tag is "way":
The dictionary should have the format 
```
{"way": ..., "way_tags": ..., "way_nodes": ...}
```

The "way" field should hold a dictionary of the following top level way attributes:

- id
- user
- uid
- version
- timestamp
- changeset

All other attributes can be ignored

The "way_tags" field should again hold a list of dictionaries, following the exact same rules as
for "node_tags".

Additionally, the dictionary should have a field "way_nodes". "way_nodes" should hold a list of
dictionaries, one for each nd child tag.  Each dictionary should have the fields:

- id: the top level element (way) id
- node_id: the ref attribute value of the nd tag
- position: the index starting at 0 of the nd tag i.e. what order the nd tag appears within the way element

The final return value for a "way" element should look something like:
```
{'way': {'id': 209809850,
         'user': 'chicago-buildings',
         'uid': 674454,
         'version': '1',
         'timestamp': '2013-03-13T15:58:04Z',
         'changeset': 15353317},
 'way_nodes': [{'id': 209809850, 'node_id': 2199822281, 'position': 0},
               {'id': 209809850, 'node_id': 2199822390, 'position': 1},
               {'id': 209809850, 'node_id': 2199822392, 'position': 2},
               {'id': 209809850, 'node_id': 2199822369, 'position': 3},
               {'id': 209809850, 'node_id': 2199822370, 'position': 4},
               {'id': 209809850, 'node_id': 2199822284, 'position': 5},
               {'id': 209809850, 'node_id': 2199822281, 'position': 6}],
 'way_tags': [{'id': 209809850,
               'key': 'housenumber',
               'type': 'addr',
               'value': '1412'},
              {'id': 209809850,
               'key': 'street',
               'type': 'addr',
               'value': 'West Lexington St.'},
              {'id': 209809850,
               'key': 'street:name',
               'type': 'addr',
               'value': 'Lexington'},
              {'id': '209809850',
               'key': 'street:prefix',
               'type': 'addr',
               'value': 'West'},
              {'id': 209809850,
               'key': 'street:type',
               'type': 'addr',
               'value': 'Street'},
              {'id': 209809850,
               'key': 'building',
               'type': 'regular',
               'value': 'yes'},
              {'id': 209809850,
               'key': 'levels',
               'type': 'building',
               'value': '1'},
              {'id': 209809850,
               'key': 'building_id',
               'type': 'chicago',
               'value': '366409'}]}
```

## Import CSV into database

Data schema is defined in `data_wrangling_schema.sql`

To create a ```newyork.db```
```
$ sqlite3 newyork.db

sqlite> CREATE TABLE nodes(schema)
sqlite> .mode csv
sqlite> .import nodes.csv nodes
...

```

## SQL

### Number of nodes

```
SELECT COUNT(*) FROM nodes;

1048575
```

### Number of ways

```
SELECT COUNT(*) FROM ways;

179340
```

### Find number of unique users

```
SELECT COUNT(*) FROM (SELECT uid from nodes UNION SELECT uid FROM ways);

2394
```

### Top 10 contributing users

```
SELECT T.user, COUNT(*) as num
FROM (SELECT uid, user from nodes UNION ALL SELECT uid, user FROM ways) as T
GROUP BY T.user
ORDER BY num DESC
LIMIT 10;

Rub21_nycbuildings|488888
ingalls_nycbuildings|93519
woodpeck_fixbot|62369
minewman|49411
SuffolkNY|29050
Northfork|28583
ediyes_nycbuildings|27237
MySuffolkNY|24950
lxbarth_nycbuildings|23444
smlevine|22017
```

### Number of users having less than 5 posts

```
SELECT COUNT(*)
FROM
  (SELECT T.user, COUNT(*) as num
  FROM (SELECT uid, user from nodes UNION ALL SELECT uid, user FROM ways) as T
  GROUP BY T.user
  HAVING num < 5);

1311
```

### Average Number of user's posts

```
SELECT AVG(num)
FROM
  (SELECT T.user, COUNT(*) as num
  FROM (SELECT uid, user from nodes UNION ALL SELECT uid, user FROM ways) as T
  GROUP BY T.user);

513
```

## Additional Exploration

###  Top 10 amenities

```
SELECT value, COUNT(*) as num
FROM nodes_tags
WHERE key = 'amenity'
GROUP BY value
ORDER BY num DESC
LIMIT 10;


bicycle_parking|483
school|332
place_of_worship|297
restaurant|275
cafe|83
fast_food|77
bank|68
bench|59
fire_station|58
bar|42

```

### Top 10 Restaurant Food

We emphasize "restaurant" here. So the amenity should be "restaurant" for that food style.

```
SELECT nodes_tags.value as food, COUNT(*) as num
FROM nodes_tags
  JOIN (SELECT id FROM nodes_tags WHERE value = 'restaurant') as a
  on nodes_tags.id = a.id
WHERE nodes_tags.key = 'cuisine'
GROUP BY food
ORDER by num DESC
LIMIT 10;

italian|24
american|18
pizza|11
chinese|9
japanese|9
mexican|9
burger|6
french|6
indian|5
asian|4
```

## Future Improvement

### More auditing

- Phone numbers formatting (xxx-xxx-xxxx)
- Validate postcode based on street address


## Conlcusion

New York is a very big city and it's very interesting to discover the insights from the OpenStreetMap dataset. But obviously, the open-souce dataset is not perfect and one has to do a lot of auditing before diving into the data. It also worth comparing the dataset with the commerical Google map and to see if this dataset can provide extra information for the Google map.


