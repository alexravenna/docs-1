---
aliases:
- /develop/interact/search-and-query/query/geo-spatial
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
description: Query based on geographic data
linkTitle: Geospatial
title: Geospatial queries
weight: 4
---

The geospatial feature in Redis Open Source allows you to query for data associated with geographic locations. You can either query for locations within a specific radius or based on geometric shapes, such as polygons. A polygon shape could, for instance, represent a lake or the layout of a building.

The examples in this article use the following schema:

| Field name       | Field type   |
| --------------   | ----------   |
| `store_location` | `GEO`        |
| `pickup_zone`    | `GEOSHAPE`   |


{{% alert title="Note" color="warning" %}}
Redis version 7.2.0 or higher is required to use the `GEOSHAPE` field type.
{{% /alert  %}}

## Radius

You can construct a radius query by passing the center coordinates (longitude, latitude), the radius, and the distance unit to the [FT.SEARCH]({{< relref "commands/ft.search" >}}) command.

```
FT.SEARCH index "@geo_field:[lon lat radius unit]"
```

Allowed units are `m`, `km`, `mi`, and `ft`.

The following query finds all bicycle stores within a radius of 20 miles around London:

{{< clients-example query_geo geo1 >}}
FT.SEARCH idx:bicycle "@store_location:[-0.1778 51.5524 20 mi]"
{{< /clients-example >}}

## Shape

The only supported shapes are points and polygons. You can query for polygons or points that either contain or are within a given geometric shape.

```
FT.SEARCH index "@geo_shape_field:[{WITHIN|CONTAINS|INTERSECTS|DISJOINT} $shape] PARAMS 2 shape "shape_as_wkt" DIALECT 3
```

Here is a more detailed explanation of this query:

1. **Field name**: you need to replace `geo_shape_field` with the `GEOSHAPE` field's name on which you want to query.
2. **Spatial operator**: spatial operators define the relationship between the shapes in the database and the shape you are searching for. You can either use `WITHIN`, `CONTAINS`, `INTERSECTS`, or `DISJOINT`. `WITHIN` finds any shape in the database that is inside the given shape. `CONTAINS` queries for any shape that surrounds the given shape. `INTERSECTS` finds any shape that has coordinates in common with the provided shape. `DISJOINT` finds any shapes that have nothing in common with the provided shape. `INTERSECTS` and `DISJOINT` were introduced in v2.10.
3. **Parameter**: the query refers to a parameter named `shape`. You can use any parameter name here. You need to use the `PARAMS` clause to set the parameter value. The value follows the [well-known text representation of a geometry](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry). Supported types are `POINT(x y)` and `POLYGON((x1 y1, x2 y2, ...))`.
4. **Dialect**: Shape-based queries have been available since version three of the query dialect.

The following example query verifies if a bicycle is within a pickup zone:

{{< clients-example query_geo geo2 >}}
FT.SEARCH idx:bicycle "@pickup_zone:[CONTAINS $bike]" PARAMS 2 bike "POINT(-0.1278 51.5074)" DIALECT 3
{{< /clients-example >}}

If you want to find all pickup zones that are approximately within Europe, then you can use the following query:

{{< clients-example query_geo geo3 >}}
FT.SEARCH idx:bicycle "@pickup_zone:[WITHIN $europe]" PARAMS 2 europe "POLYGON((-25 35, 40 35, 40 70, -25 70, -25 35))" DIALECT 3
{{< /clients-example >}}