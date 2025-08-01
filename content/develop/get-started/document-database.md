---
categories:
- docs
- develop
- stack
- oss
- rs
- rc
- oss
- kubernetes
- clients
description: Understand how to use Redis as a document database
linkTitle: Document database
stack: true
title: Redis as a document database quick start guide
weight: 2
---


This quick start guide shows you how to:

1. Create a secondary index
2. Add [JSON]({{< relref "/develop/data-types/json/" >}}) documents
3. Search and query your data

The examples in this article refer to a simple bicycle inventory that contains JSON documents with the following structure:

```json
{
  "brand": "brand name",
  "condition": "new | used | refurbished",
  "description": "description",
  "model": "model",
  "price": 0
}
```

## Setup

The easiest way to get started with [Redis]({{< relref "/operate/oss_and_stack/" >}}) is to use Redis Cloud:

1. Create a [free account](https://redis.com/try-free?utm_source=redisio&utm_medium=referral&utm_campaign=2023-09-try_free&utm_content=cu-redis_cloud_users).

   <img src="../img/free-cloud-db.png" width="500px">
2. Follow the instructions to create a free database.

This free Redis Cloud database comes out of the box with all the Redis Open Source features.

You can alternatively use the [installation guides]({{< relref "/operate/oss_and_stack/install/install-stack/" >}}) to install Redis Open Source on your local machine.


## Connect

The first step is to connect to your Redis Open Source database. You can find further details about the connection options in this documentation site's [Tools section]({{< relref "/develop/tools" >}}). The following example shows how to connect to a Redis Open Source server that runs on localhost (`-h 127.0.0.1`) and listens on the default port (`-p 6379`): 

{{< clients-example search_quickstart connect >}}
> redis-cli -h 127.0.0.1 -p 6379
{{< /clients-example>}}

<br/>
{{% alert title="Tip" color="warning" %}}
You can copy and paste the connection details from the Redis Cloud database configuration page. Here is an example connection string of a Cloud database that is hosted in the AWS region `us-east-1` and listens on port 16379: `redis-16379.c283.us-east-1-4.ec2.cloud.redislabs.com:16379`. The connection string has the format `host:port`. You must also copy and paste your Cloud database's username and password and then pass the credentials to your client or use the [AUTH command]({{< relref "/commands/auth" >}}) after the connection is established.
{{% /alert  %}}


## Create an index

As explained in the [in-memory data store]({{< relref "/develop/get-started/data-store" >}}) quick start guide, Redis allows you to access an item directly via its key. You also learned how to scan the keyspace. Whereby you can use other data structures (e.g., hashes and sorted sets) as secondary indexes, your application would need to maintain those indexes manually. Redis is a document database that allows you to declare which fields are auto-indexed. Redis currently supports secondary index creation on the [hashes]({{< relref "/develop/data-types/hashes" >}}) and [JSON]({{< relref "/develop/data-types/json" >}}) documents.

The following example shows an [FT.CREATE]({{< relref "commands/ft.create" >}}) command that creates an index with some text fields, a numeric field (price), and a tag field (condition). The text fields have a weight of 1.0, meaning they have the same relevancy in the context of full-text searches. The field names follow the [JSONPath]({{< relref "/develop/data-types/json/path" >}}) notion. Each such index field maps to a property within the JSON document.


{{< clients-example search_quickstart create_index >}}
> FT.CREATE idx:bicycle ON JSON PREFIX 1 bicycle: SCORE 1.0 SCHEMA $.brand AS brand TEXT WEIGHT 1.0 $.model AS model TEXT WEIGHT 1.0 $.description AS description TEXT WEIGHT 1.0 $.price AS price NUMERIC $.condition AS condition TAG SEPARATOR ,
OK
{{< / clients-example >}}

Any pre-existing JSON documents with a key prefix `bicycle:` are automatically added to the index. Additionally, any JSON documents with that prefix created or modified after index creation are added or re-added to the index.

## Add JSON documents

The example below shows you how to use the [JSON.SET]({{< relref "commands/json.set" >}}) command to create new JSON documents:

{{< clients-example search_quickstart add_documents "" 2 >}}
> JSON.SET "bicycle:0" "." "{\"brand\": \"Velorim\", \"model\": \"Jigger\", \"price\": 270, \"description\": \"Small and powerful, the Jigger is the best ride for the smallest of tikes! This is the tiniest kids\\u2019 pedal bike on the market available without a coaster brake, the Jigger is the vehicle of choice for the rare tenacious little rider raring to go.\", \"condition\": \"new\"}"
OK
> JSON.SET "bicycle:1" "." "{\"brand\": \"Bicyk\", \"model\": \"Hillcraft\", \"price\": 1200, \"description\": \"Kids want to ride with as little weight as possible. Especially on an incline! They may be at the age when a 27.5\\\" wheel bike is just too clumsy coming off a 24\\\" bike. The Hillcraft 26 is just the solution they need!\", \"condition\": \"used\"}"
OK
> JSON.SET "bicycle:2" "." "{\"brand\": \"Nord\", \"model\": \"Chook air 5\", \"price\": 815, \"description\": \"The Chook Air 5  gives kids aged six years and older a durable and uberlight mountain bike for their first experience on tracks and easy cruising through forests and fields. The lower  top tube makes it easy to mount and dismount in any situation, giving your kids greater safety on the trails.\", \"condition\": \"used\"}"
OK
> JSON.SET "bicycle:3" "." "{\"brand\": \"Eva\", \"model\": \"Eva 291\", \"price\": 3400, \"description\": \"The sister company to Nord, Eva launched in 2005 as the first and only women-dedicated bicycle brand. Designed by women for women, allEva bikes are optimized for the feminine physique using analytics from a body metrics database. If you like 29ers, try the Eva 291. It\\u2019s a brand new bike for 2022.. This full-suspension, cross-country ride has been designed for velocity. The 291 has 100mm of front and rear travel, a superlight aluminum frame and fast-rolling 29-inch wheels. Yippee!\", \"condition\": \"used\"}"
OK
> JSON.SET "bicycle:4" "." "{\"brand\": \"Noka Bikes\", \"model\": \"Kahuna\", \"price\": 3200, \"description\": \"Whether you want to try your hand at XC racing or are looking for a lively trail bike that's just as inspiring on the climbs as it is over rougher ground, the Wilder is one heck of a bike built specifically for short women. Both the frames and components have been tweaked to include a women\\u2019s saddle, different bars and unique colourway.\", \"condition\": \"used\"}"
OK
> JSON.SET "bicycle:5" "." "{\"brand\": \"Breakout\", \"model\": \"XBN 2.1 Alloy\", \"price\": 810, \"description\": \"The XBN 2.1 Alloy is our entry-level road bike \\u2013 but that\\u2019s not to say that it\\u2019s a basic machine. With an internal weld aluminium frame, a full carbon fork, and the slick-shifting Claris gears from Shimano\\u2019s, this is a bike which doesn\\u2019t break the bank and delivers craved performance.\", \"condition\": \"new\"}"
OK
> JSON.SET "bicycle:6" "." "{\"brand\": \"ScramBikes\", \"model\": \"WattBike\", \"price\": 2300, \"description\": \"The WattBike is the best e-bike for people who still feel young at heart. It has a Bafang 1000W mid-drive system and a 48V 17.5AH Samsung Lithium-Ion battery, allowing you to ride for more than 60 miles on one charge. It\\u2019s great for tackling hilly terrain or if you just fancy a more leisurely ride. With three working modes, you can choose between E-bike, assisted bicycle, and normal bike modes.\", \"condition\": \"new\"}"
OK
> JSON.SET "bicycle:7" "." "{\"brand\": \"Peaknetic\", \"model\": \"Secto\", \"price\": 430, \"description\": \"If you struggle with stiff fingers or a kinked neck or back after a few minutes on the road, this lightweight, aluminum bike alleviates those issues and allows you to enjoy the ride. From the ergonomic grips to the lumbar-supporting seat position, the Roll Low-Entry offers incredible comfort. The rear-inclined seat tube facilitates stability by allowing you to put a foot on the ground to balance at a stop, and the low step-over frame makes it accessible for all ability and mobility levels. The saddle is very soft, with a wide back to support your hip joints and a cutout in the center to redistribute that pressure. Rim brakes deliver satisfactory braking control, and the wide tires provide a smooth, stable ride on paved roads and gravel. Rack and fender mounts facilitate setting up the Roll Low-Entry as your preferred commuter, and the BMX-like handlebar offers space for mounting a flashlight, bell, or phone holder.\", \"condition\": \"new\"}"
OK
> JSON.SET "bicycle:8" "." "{\"brand\": \"nHill\", \"model\": \"Summit\", \"price\": 1200, \"description\": \"This budget mountain bike from nHill performs well both on bike paths and on the trail. The fork with 100mm of travel absorbs rough terrain. Fat Kenda Booster tires give you grip in corners and on wet trails. The Shimano Tourney drivetrain offered enough gears for finding a comfortable pace to ride uphill, and the Tektro hydraulic disc brakes break smoothly. Whether you want an affordable bike that you can take to work, but also take trail in mountains on the weekends or you\\u2019re just after a stable, comfortable ride for the bike path, the Summit gives a good value for money.\", \"condition\": \"new\"}"
OK
> JSON.SET "bicycle:9" "." "{\"model\": \"ThrillCycle\", \"brand\": \"BikeShind\", \"price\": 815, \"description\": \"An artsy,  retro-inspired bicycle that\\u2019s as functional as it is pretty: The ThrillCycle steel frame offers a smooth ride. A 9-speed drivetrain has enough gears for coasting in the city, but we wouldn\\u2019t suggest taking it to the mountains. Fenders protect you from mud, and a rear basket lets you transport groceries, flowers and books. The ThrillCycle comes with a limited lifetime warranty, so this little guy will last you long past graduation.\", \"condition\": \"refurbished\"}"
OK
{{< / clients-example >}}

## Search and query using the Redis Query Engine

### Wildcard query

You can retrieve all indexed documents using the [FT.SEARCH]({{< relref "commands/ft.search" >}}) command. Note the `LIMIT` clause below, which allows result pagination.

{{< clients-example search_quickstart wildcard_query "" 10 >}}
> FT.SEARCH "idx:bicycle" "*" LIMIT 0 10
1) (integer) 10
 2) "bicycle:1"
 3) 1) "$"
    2) "{\"brand\":\"Bicyk\",\"model\":\"Hillcraft\",\"price\":1200,\"description\":\"Kids want to ride with as little weight as possible. Especially on an incline! They may be at the age when a 27.5\\\" wheel bike is just too clumsy coming off a 24\\\" bike. The Hillcraft 26 is just the solution they need!\",\"condition\":\"used\"}"
 4) "bicycle:2"
 5) 1) "$"
    2) "{\"brand\":\"Nord\",\"model\":\"Chook air 5\",\"price\":815,\"description\":\"The Chook Air 5  gives kids aged six years and older a durable and uberlight mountain bike for their first experience on tracks and easy cruising through forests and fields. The lower  top tube makes it easy to mount and dismount in any situation, giving your kids greater safety on the trails.\",\"condition\":\"used\"}"
 6) "bicycle:4"
 7) 1) "$"
    2) "{\"brand\":\"Noka Bikes\",\"model\":\"Kahuna\",\"price\":3200,\"description\":\"Whether you want to try your hand at XC racing or are looking for a lively trail bike that's just as inspiring on the climbs as it is over rougher ground, the Wilder is one heck of a bike built specifically for short women. Both the frames and components have been tweaked to include a women\xe2\x80\x99s saddle, different bars and unique colourway.\",\"condition\":\"used\"}"
 8) "bicycle:5"
 9) 1) "$"
    2) "{\"brand\":\"Breakout\",\"model\":\"XBN 2.1 Alloy\",\"price\":810,\"description\":\"The XBN 2.1 Alloy is our entry-level road bike \xe2\x80\x93 but that\xe2\x80\x99s not to say that it\xe2\x80\x99s a basic machine. With an internal weld aluminium frame, a full carbon fork, and the slick-shifting Claris gears from Shimano\xe2\x80\x99s, this is a bike which doesn\xe2\x80\x99t break the bank and delivers craved performance.\",\"condition\":\"new\"}"
10) "bicycle:0"
11) 1) "$"
    2) "{\"brand\":\"Velorim\",\"model\":\"Jigger\",\"price\":270,\"description\":\"Small and powerful, the Jigger is the best ride for the smallest of tikes! This is the tiniest kids\xe2\x80\x99 pedal bike on the market available without a coaster brake, the Jigger is the vehicle of choice for the rare tenacious little rider raring to go.\",\"condition\":\"new\"}"
12) "bicycle:6"
13) 1) "$"
    2) "{\"brand\":\"ScramBikes\",\"model\":\"WattBike\",\"price\":2300,\"description\":\"The WattBike is the best e-bike for people who still feel young at heart. It has a Bafang 1000W mid-drive system and a 48V 17.5AH Samsung Lithium-Ion battery, allowing you to ride for more than 60 miles on one charge. It\xe2\x80\x99s great for tackling hilly terrain or if you just fancy a more leisurely ride. With three working modes, you can choose between E-bike, assisted bicycle, and normal bike modes.\",\"condition\":\"new\"}"
14) "bicycle:7"
15) 1) "$"
    2) "{\"brand\":\"Peaknetic\",\"model\":\"Secto\",\"price\":430,\"description\":\"If you struggle with stiff fingers or a kinked neck or back after a few minutes on the road, this lightweight, aluminum bike alleviates those issues and allows you to enjoy the ride. From the ergonomic grips to the lumbar-supporting seat position, the Roll Low-Entry offers incredible comfort. The rear-inclined seat tube facilitates stability by allowing you to put a foot on the ground to balance at a stop, and the low step-over frame makes it accessible for all ability and mobility levels. The saddle is very soft, with a wide back to support your hip joints and a cutout in the center to redistribute that pressure. Rim brakes deliver satisfactory braking control, and the wide tires provide a smooth, stable ride on paved roads and gravel. Rack and fender mounts facilitate setting up the Roll Low-Entry as your preferred commuter, and the BMX-like handlebar offers space for mounting a flashlight, bell, or phone holder.\",\"condition\":\"new\"}"
16) "bicycle:9"
17) 1) "$"
    2) "{\"model\":\"ThrillCycle\",\"brand\":\"BikeShind\",\"price\":815,\"description\":\"An artsy,  retro-inspired bicycle that\xe2\x80\x99s as functional as it is pretty: The ThrillCycle steel frame offers a smooth ride. A 9-speed drivetrain has enough gears for coasting in the city, but we wouldn\xe2\x80\x99t suggest taking it to the mountains. Fenders protect you from mud, and a rear basket lets you transport groceries, flowers and books. The ThrillCycle comes with a limited lifetime warranty, so this little guy will last you long past graduation.\",\"condition\":\"refurbished\"}"
18) "bicycle:3"
19) 1) "$"
    2) "{\"brand\":\"Eva\",\"model\":\"Eva 291\",\"price\":3400,\"description\":\"The sister company to Nord, Eva launched in 2005 as the first and only women-dedicated bicycle brand. Designed by women for women, allEva bikes are optimized for the feminine physique using analytics from a body metrics database. If you like 29ers, try the Eva 291. It\xe2\x80\x99s a brand new bike for 2022.. This full-suspension, cross-country ride has been designed for velocity. The 291 has 100mm of front and rear travel, a superlight aluminum frame and fast-rolling 29-inch wheels. Yippee!\",\"condition\":\"used\"}"
20) "bicycle:8"
21) 1) "$"
    2) "{\"brand\":\"nHill\",\"model\":\"Summit\",\"price\":1200,\"description\":\"This budget mountain bike from nHill performs well both on bike paths and on the trail. The fork with 100mm of travel absorbs rough terrain. Fat Kenda Booster tires give you grip in corners and on wet trails. The Shimano Tourney drivetrain offered enough gears for finding a comfortable pace to ride uphill, and the Tektro hydraulic disc brakes break smoothly. Whether you want an affordable bike that you can take to work, but also take trail in mountains on the weekends or you\xe2\x80\x99re just after a stable, comfortable ride for the bike path, the Summit gives a good value for money.\",\"condition\":\"new\"}"
{{< / clients-example >}}

### Single-term full-text query

The following command shows a simple single-term query for finding all bicycles with a specific model:

{{< clients-example search_quickstart query_single_term >}}
> FT.SEARCH "idx:bicycle" "@model:Jigger" LIMIT 0 10
1) (integer) 1
2) "bicycle:0"
3) 1) "$"
   2) "{\"brand\":\"Velorim\",\"model\":\"Jigger\",\"price\":270,\"description\":\"Small and powerful, the Jigger is the best ride for the smallest of tikes! This is the tiniest kids\xe2\x80\x99 pedal bike on the market available without a coaster brake, the Jigger is the vehicle of choice for the rare tenacious little rider raring to go.\",\"condition\":\"new\"}"
{{< / clients-example >}}

### Exact match query

Below is a command to perform an exact match query that finds all bicycles with the brand name `Noka Bikes`. You must use double quotes around the search term when constructing an exact match query on a  text field.

{{< clients-example search_quickstart query_exact_matching >}}
> FT.SEARCH "idx:bicycle" "@brand:\"Noka Bikes\"" LIMIT 0 10
1) (integer) 1
2) "bicycle:4"
3) 1) "$"
   2) "{\"brand\":\"Noka Bikes\",\"model\":\"Kahuna\",\"price\":3200,\"description\":\"Whether you want to try your hand at XC racing or are looking for a lively trail bike that's just as inspiring on the climbs as it is over rougher ground, the Wilder is one heck of a bike built specifically for short women. Both the frames and components have been tweaked to include a women\xe2\x80\x99s saddle, different bars and unique colourway.\",\"condition\":\"used\"}"
{{< / clients-example >}}

Please see the [query documentation]({{< relref "/develop/ai/search-and-query/query/" >}}) to learn how to make more advanced queries.

## Next steps

You can learn more about how to use Redis Open Source as a vector database in the following quick start guide:

* [Redis as a vector database]({{< relref "/develop/get-started/vector-database" >}})

## Continue learning with Redis University

{{< university-links >}}