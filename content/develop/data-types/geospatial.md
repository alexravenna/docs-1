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
description: 'Introduction to the Redis Geospatial data type

  '
linkTitle: Geospatial
title: Redis geospatial
weight: 80
---

Redis geospatial indexes let you store coordinates and search for them.
This data structure is useful for finding nearby points within a given radius or bounding box.

{{< note >}}Take care not to confuse the Geospatial data type with the
[Geospatial]({{< relref "/develop/ai/search-and-query/advanced-concepts/geo" >}})
features in [Redis Query Engine]({{< relref "/develop/ai/search-and-query" >}}).
Although there are some similarities between these two features, the data type is intended
for simpler use cases and doesn't have the range of format options and queries
available in Redis Query Engine.
{{< /note >}}

## Basic commands

* [`GEOADD`]({{< relref "/commands/geoadd" >}}) adds a location to a given geospatial index (note that longitude comes before latitude with this command).
* [`GEOSEARCH`]({{< relref "/commands/geosearch" >}}) returns locations with a given radius or a bounding box.

See the [complete list of geospatial index commands]({{< relref "/commands/" >}}?group=geo).


## Examples

Suppose you're building a mobile app that lets you find all of the bike rental stations closest to your current location.

Add several locations to a geospatial index:
{{< clients-example geo_tutorial geoadd >}}
> GEOADD bikes:rentable -122.27652 37.805186 station:1
(integer) 1
> GEOADD bikes:rentable -122.2674626 37.8062344 station:2
(integer) 1
> GEOADD bikes:rentable -122.2469854 37.8104049 station:3
(integer) 1
{{< /clients-example >}}

Find all locations within a 5 kilometer radius of a given location, and return the distance to each location:
{{< clients-example geo_tutorial geosearch >}}
> GEOSEARCH bikes:rentable FROMLONLAT -122.2612767 37.7936847 BYRADIUS 5 km WITHDIST
1) 1) "station:1"
   2) "1.8523"
2) 1) "station:2"
   2) "1.4979"
3) 1) "station:3"
   2) "2.2441"
{{< /clients-example >}}

## Learn more

* [Redis Geospatial Explained](https://www.youtube.com/watch?v=qftiVQraxmI) introduces geospatial indexes by showing you how to build a map of local park attractions.
* [Redis University's RU101](https://university.redis.com/courses/ru101/) covers Redis geospatial indexes in detail.
