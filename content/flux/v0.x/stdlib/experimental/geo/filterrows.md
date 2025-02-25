---
title: geo.filterRows() function
description: >
  The geo.filterRows() function filters data by a specified geographic region with
  the option of strict filtering.
aliases:
  - /influxdb/v2.0/reference/flux/stdlib/experimental/geo/filterrows/
  - /influxdb/cloud/reference/flux/stdlib/experimental/geo/filterrows/
menu:
  flux_0_x_ref:
    name: geo.filterRows
    parent: geo
weight: 401
flux/v0.x/tags: [transformations, filters, geotemporal, geo]
related:
  - /flux/v0.x/stdlib/experimental/geo/gridfilter/
  - /flux/v0.x/stdlib/experimental/geo/strictfilter/
  - /{{< latest "influxdb" >}}/query-data/flux/geo/
introduced: 0.63.0
---

The `geo.filterRows()` function filters data by a specified geographic region with
the option of strict filtering.
This function is a combination of [`geo.gridFilter()`](/flux/v0.x/stdlib/experimental/geo/gridfilter/)
and [`geo.strictFilter()`](/flux/v0.x/stdlib/experimental/geo/strictfilter/).

```js
import "experimental/geo"

geo.filterRows(
  region: {lat: 40.69335938, lon: -73.30078125, radius: 20.0},
  minSize: 24,
  maxSize: -1,
  level: -1,
  s2cellIDLevel: -1,
  correlationKey: ["_time"],
  strict: true
)
```

{{% note %}}
#### s2_cell_id must be part of the group key
To filter geo-temporal data with `geo.filterRows()`, `s2_cell_id` must be part
of the [group key](/flux/v0.x/get-started/data-model/#group-key).
To add `s2_cell_id` to the group key, use [`experimental.group`](/flux/v0.x/stdlib/experimental/group/):

```js
import "experimental"

// ...
  |> experimental.group(columns: ["s2_cell_id"], mode: "extend")
```
{{% /note %}}

### Strict and non-strict filtering
In most cases, the specified geographic region does not perfectly align with S2 grid cells.

- **Non-strict filtering** returns points that may be outside of the specified region but
  inside S2 grid cells partially covered by the region.
- **Strict filtering** returns only points inside the specified region.

<span class="key-geo-cell"></span> S2 grid cell  
<span class="key-geo-region"></span> Filter region  
<span class="key-geo-point"></span> Returned point

{{< flex >}}
{{% flex-content %}}
**Strict filtering**
{{< svg "/static/svgs/geo-strict.svg" >}}
{{% /flex-content %}}
{{% flex-content %}}
**Non-strict filtering**
{{< svg "/static/svgs/geo-non-strict.svg" >}}
{{% /flex-content %}}
{{< /flex >}}

## Parameters

### region {data-type="record"}
The region containing the desired data points.
Specify record properties for the shape.
_See [Region definitions](/flux/v0.x/stdlib/experimental/geo/#region-definitions)._

### minSize {data-type="int"}
Minimum number of cells that cover the specified region.
Default is `24`.

### maxSize {data-type="int"}
Maximum number of cells that cover the specified region.
Default is `-1`.

### level {data-type="int"}
[S2 cell level](https://s2geometry.io/resources/s2cell_statistics.html) of grid cells.
Default is `-1`.

{{% warn %}}
`level` is mutually exclusive with `minSize` and `maxSize` and must be less than
or equal to `s2cellIDLevel`.
{{% /warn %}}

### s2cellIDLevel {data-type="int"}
[S2 Cell level](https://s2geometry.io/resources/s2cell_statistics.html) used in `s2_cell_id` tag.
Default is `-1`.

{{% note %}}
When set to `-1`, `geo.filterRows()` attempts to automatically detect the S2 Cell ID level.
{{% /note %}}

### correlationKey {data-type="array of strings"}
List of columns used to uniquely identify a row for output.
Default is `["_time"]`.

### strict {data-type="bool"}
Enable strict geographic data filtering which filters points by longitude (`lon`) and latitude (`lat`).
For S2 grid cells that are partially covered by the defined region, only points
with coordinates in the defined region are returned.
Default is `true`.
_See [Strict and non-strict filtering](#strict-and-non-strict-filtering) above._

### tables {data-type="stream of tables"}
Input data.
Default is piped-forward data ([`<-`](/flux/v0.x/spec/expressions/#pipe-expressions)).

## Examples

##### Strictly filter data in a box-shaped region
```js
import "experimental/geo"

from(bucket: "example-bucket")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "example-measurement")
  |> geo.filterRows(
    region: {
      minLat: 40.51757813,
      maxLat: 40.86914063,
      minLon: -73.65234375,
      maxLon: -72.94921875
    }
  )
```

##### Approximately filter data in a circular region
The following example returns points with coordinates located in S2 grid cells partially
covered by the defined region even though some points my be located outside of the region.

```js
import "experimental/geo"

from(bucket: "example-bucket")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "example-measurement")
  |> geo.filterRows(
    region: {
      lat: 40.69335938,
      lon: -73.30078125,
      radius: 20.0
    }
    strict: false
  )
```

##### Filter data in a polygonal region
```js
import "experimental/geo"

from(bucket: "example-bucket")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "example-measurement")
  |> geo.filterRows(
    region: {
      points: [
        {lat: 40.671659, lon: -73.936631},
        {lat: 40.706543, lon: -73.749177},
        {lat: 40.791333, lon: -73.880327}
      ]
    }
  )
```

## Function definition
{{% truncate %}}
```js
filterRows = (
  tables=<-,
  region,
  minSize=24,
  maxSize=-1,
  level=-1,
  s2cellIDLevel=-1,
  strict=true
) => {
  _columns =
    |> columns(column: "_value")
    |> tableFind(fn: (key) => true )
    |> getColumn(column: "_value")
  _rows =
    if contains(value: "lat", set: _columns) then
      tables
        |> gridFilter(
          region: region,
          minSize: minSize,
          maxSize: maxSize,
          level: level,
          s2cellIDLevel: s2cellIDLevel)
    else
      tables
        |> gridFilter(
          region: region,
          minSize: minSize,
          maxSize: maxSize,
          level: level,
          s2cellIDLevel: s2cellIDLevel)
        |> toRows()
  _result =
    if strict then
      _rows
        |> strictFilter(region)
    else
      _rows
  return _result
}
```
{{% /truncate %}}
