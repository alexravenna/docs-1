---
acl_categories:
- '@timeseries'
- '@read'
- '@slow'
arguments:
- name: fromTimestamp
  type: string
- name: toTimestamp
  type: string
- name: LATEST
  optional: true
  since: 1.8.0
  type: string
- multiple: true
  name: Timestamp
  optional: true
  token: FILTER_BY_TS
  type: integer
- arguments:
  - name: FILTER_BY_VALUE
    token: FILTER_BY_VALUE
    type: pure-token
  - name: min
    type: double
  - name: max
    type: double
  name: fbv
  optional: true
  type: block
- arguments:
  - name: WITHLABELS
    token: WITHLABELS
    type: pure-token
  - arguments:
    - name: SELECTED_LABELS
      token: SELECTED_LABELS
      type: pure-token
    - multiple: true
      name: label1
      type: string
    name: SELECTED_LABELS_BLOCK
    type: block
  name: labels
  optional: true
  type: oneof
- name: count
  optional: true
  token: COUNT
  type: integer
- arguments:
  - name: value
    optional: true
    token: ALIGN
    type: integer
  - arguments:
    - name: avg
      token: AVG
      type: pure-token
    - name: first
      token: FIRST
      type: pure-token
    - name: last
      token: LAST
      type: pure-token
    - name: min
      token: MIN
      type: pure-token
    - name: max
      token: MAX
      type: pure-token
    - name: sum
      token: SUM
      type: pure-token
    - name: range
      token: RANGE
      type: pure-token
    - name: count
      token: COUNT
      type: pure-token
    - name: std.p
      token: STD.P
      type: pure-token
    - name: std.s
      token: STD.S
      type: pure-token
    - name: var.p
      token: VAR.P
      type: pure-token
    - name: var.s
      token: VAR.S
      type: pure-token
    - name: twa
      since: 1.8.0
      token: TWA
      type: pure-token
    name: aggregator
    token: AGGREGATION
    type: oneof
  - name: bucketDuration
    type: integer
  - name: buckettimestamp
    optional: true
    since: 1.8.0
    token: BUCKETTIMESTAMP
    type: pure-token
  - name: empty
    optional: true
    since: 1.8.0
    token: EMPTY
    type: pure-token
  name: aggregation
  optional: true
  type: block
- arguments:
  - name: l=v
    type: string
  - name: l!=v
    type: string
  - name: l=
    type: string
  - name: l!=
    type: string
  - name: l=(v1,v2,...)
    type: string
  - name: l!=(v1,v2,...)
    type: string
  multiple: true
  name: filterExpr
  token: FILTER
  type: oneof
- arguments:
  - name: GROUPBY
    token: GROUPBY
    type: pure-token
  - name: label
    type: string
  - name: REDUCE
    type: string
  - name: reducer
    type: string
  name: groupby
  optional: true
  type: block
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
complexity: O(n/m+k) where n = Number of data points, m = Chunk size (data points
  per chunk), k = Number of data points that are in the requested ranges
description: Query a range across multiple time-series by filters in reverse direction
group: timeseries
hidden: false
linkTitle: TS.MREVRANGE
module: TimeSeries
since: 1.4.0
stack_path: docs/data-types/timeseries
summary: Query a range across multiple time-series by filters in reverse direction
syntax: "TS.MREVRANGE fromTimestamp toTimestamp\n  [LATEST]\n  [FILTER_BY_TS ts...]\n\
  \  [FILTER_BY_VALUE min max]\n  [WITHLABELS | <SELECTED_LABELS label...>]\n  [COUNT\
  \ count]\n  [[ALIGN align] AGGREGATION aggregator bucketDuration [BUCKETTIMESTAMP\
  \ bt] [EMPTY]]\n  FILTER filterExpr...\n  [GROUPBY label REDUCE reducer]\n"
syntax_fmt: "TS.MREVRANGE fromTimestamp toTimestamp [LATEST]\n  [FILTER_BY_TS\_Timestamp\
  \ [Timestamp ...]] [FILTER_BY_VALUE min max]\n  [WITHLABELS | <SELECTED_LABELS label1\
  \ [label1 ...]>] [COUNT\_count]\n  [[ALIGN\_value] AGGREGATION\_<AVG | FIRST | LAST\
  \ | MIN | MAX | SUM |\n  RANGE | COUNT | STD.P | STD.S | VAR.P | VAR.S | TWA>\n\
  \  bucketDuration [BUCKETTIMESTAMP] [EMPTY]] FILTER\_<l=v | l!=v | l=\n  | l!= |\
  \ l=(v1,v2,...) | l!=(v1,v2,...) [l=v | l!=v | l= | l!= |\n  l=(v1,v2,...) | l!=(v1,v2,...)\
  \ ...]> [GROUPBY label REDUCE\n  reducer]"
syntax_str: "toTimestamp [LATEST] [FILTER_BY_TS\_Timestamp [Timestamp ...]] [FILTER_BY_VALUE\
  \ min max] [WITHLABELS | <SELECTED_LABELS label1 [label1 ...]>] [COUNT\_count] [[ALIGN\_\
  value] AGGREGATION\_<AVG | FIRST | LAST | MIN | MAX | SUM | RANGE | COUNT | STD.P\
  \ | STD.S | VAR.P | VAR.S | TWA> bucketDuration [BUCKETTIMESTAMP] [EMPTY]] FILTER\_\
  <l=v | l!=v | l= | l!= | l=(v1,v2,...) | l!=(v1,v2,...) [l=v | l!=v | l= | l!= |\
  \ l=(v1,v2,...) | l!=(v1,v2,...) ...]> [GROUPBY label REDUCE reducer]"
title: TS.MREVRANGE
---

Query a range across multiple time series by filters in the reverse direction.

{{< note >}}
This command will reply only if the current user has read access to all keys that match the filter.
Otherwise, it will reply with "*(error): current user doesn't have read permission to one or more keys that match the specified filter*".
{{< /note >}}

[Examples](#examples)

## Required arguments

<details open>
<summary><code>fromTimestamp</code></summary> 

is the start timestamp for the range query (integer Unix timestamp in milliseconds) or `-` to denote the timestamp of the earliest sample among all the time series that passes `FILTER filterExpr...`.
</details>

<details open>
<summary><code>toTimestamp</code></summary> 

is the end timestamp for the range query (integer Unix timestamp in milliseconds) or `+` to denote the timestamp of the latest sample among all the time series that passes `FILTER filterExpr...`.
</details>

<details open>
<summary><code>FILTER filterExpr...</code></summary> 

filters time series based on their labels and label values. Each filter expression has one of the following syntaxes:

  - `label!=` - the time series has a label named `label`
  - `label=value` - the time series has a label named `label` with a value equal to `value`
  - `label=(value1,value2,...)` - the time series has a label named `label` with a value equal to one of the values in the list
  - `label=` - the time series does not have a label named `label`
  - `label!=value` - the time series does not have a label named `label` with a value equal to `value`
  - `label!=(value1,value2,...)` - the time series does not have a label named `label` with a value equal to any of the values in the list

  <note><b>Notes:</b>
   - At least one filter expression with a syntax `label=value` or `label=(value1,value2,...)` is required.
   - Filter expressions are conjunctive. For example, the filter `type=temperature room=study` means that a time series is a temperature time series of a study room.
   - Whitespaces are unallowed in a filter expression except between quotes or double quotes in values - e.g., `x="y y"` or `x='(y y,z z)'`.
   </note>
</details>

## Optional arguments

<details open>
<summary><code>LATEST</code> (since RedisTimeSeries v1.8)</summary> 

is used when a time series is a compaction. With `LATEST`, TS.MREVRANGE also reports the compacted value of the latest (possibly partial) bucket, given that this bucket's start time falls within `[fromTimestamp, toTimestamp]`. Without `LATEST`, TS.MREVRANGE does not report the latest (possibly partial) bucket. When a time series is not a compaction, `LATEST` is ignored.
  
The data in the latest bucket of a compaction is possibly partial. A bucket is _closed_ and compacted only upon the arrival of a new sample that _opens_ a new _latest_ bucket. There are cases, however, when the compacted value of the latest (possibly partial) bucket is also required. In such a case, use `LATEST`.
</details>

<details open>
<summary><code>FILTER_BY_TS ts...</code> (since RedisTimeSeries v1.6)</summary> 

filters samples by a list of specific timestamps. A sample passes the filter if its exact timestamp is specified and falls within `[fromTimestamp, toTimestamp]`.

When used together with `AGGREGATION`: samples are filtered before being aggregated.
</details>

<details open>
<summary><code>FILTER_BY_VALUE min max</code> (since RedisTimeSeries v1.6)</summary> 

filters samples by minimum and maximum values.

When used together with `AGGREGATION`: samples are filtered before being aggregated.
</details>

<details open>
<summary><code>WITHLABELS</code></summary> 

includes in the reply all label-value pairs representing metadata labels of the time series. 
If `WITHLABELS` or `SELECTED_LABELS` are not specified, by default, an empty list is reported as label-value pairs.
</details>

<details open>
<summary><code>SELECTED_LABELS label...</code> (since RedisTimeSeries v1.6)</summary>

returns a subset of the label-value pairs that represent metadata labels of the time series. 
Use when a large number of labels exists per series, but only the values of some of the labels are required. 
If `WITHLABELS` or `SELECTED_LABELS` are not specified, by default, an empty list is reported as label-value pairs.
</details>

<details open>
<summary><code>COUNT count</code></summary> 

When used without `AGGREGATION`: limits the number of reported samples per time series.

When used together with `AGGREGATION`: limits the number of reported buckets.
</details>

<details open>
<summary><code>ALIGN align</code> (since RedisTimeSeries v1.6)</summary> 

is a time bucket alignment control for `AGGREGATION`. It controls the time bucket timestamps by changing the reference timestamp on which a bucket is defined. 

Values include:
   
 - `start` or `-`: The reference timestamp will be the query start interval time (`fromTimestamp`) which can't be `-`
 - `end` or `+`: The reference timestamp will be the query end interval time (`toTimestamp`) which can't be `+`
 - A specific timestamp: align the reference timestamp to a specific time
   
<note><b>Note:</b> When not provided, alignment is set to `0`.</note>
</details>

<details open>
<summary><code>AGGREGATION aggregator bucketDuration</code></summary> 

per time series, aggregates samples into time buckets, where:

  - `aggregator` takes one of the following aggregation types:

    | `aggregator` | Description                                                                    |
    | ------------ | ------------------------------------------------------------------------------ |
    | `avg`        | Arithmetic mean of all values                                                  |
    | `sum`        | Sum of all values                                                              |
    | `min`        | Minimum value                                                                  |
    | `max`        | Maximum value                                                                  |
    | `range`      | Difference between maximum value and minimum value                             |
    | `count`      | Number of values                                                               |
    | `first`      | Value with lowest timestamp in the bucket                                      |
    | `last`       | Value with highest timestamp in the bucket                                     |
    | `std.p`      | Population standard deviation of the values                                    |
    | `std.s`      | Sample standard deviation of the values                                        |
    | `var.p`      | Population variance of the values                                              |
    | `var.s`      | Sample variance of the values                                                  |
    | `twa`        | Time-weighted average over the bucket's timeframe (since RedisTimeSeries v1.8) |

  - `bucketDuration` is duration of each bucket, in milliseconds.
  
  Without `ALIGN`, bucket start times are multiples of `bucketDuration`.
  
  With `ALIGN align`, bucket start times are multiples of `bucketDuration` with remainder `align % bucketDuration`.
  
  The first bucket start time is less than or equal to `fromTimestamp`.  
</details>

<details open>
<summary><code>[BUCKETTIMESTAMP bt]</code> (since RedisTimeSeries v1.8)</summary> 

controls how bucket timestamps are reported.

| `bt`             | Timestamp reported for each bucket                            |
| ---------------- | ------------------------------------------------------------- |
| `-` or `start`   | the bucket's start time (default)                             |
| `+` or `end`     | the bucket's end time                                         |
| `~` or `mid`     | the bucket's mid time (rounded down if not an integer)        |
</details>

<details open>
<summary><code>[EMPTY]</code> (since RedisTimeSeries v1.8)</summary> 

is a flag, which, when specified, reports aggregations also for empty buckets.

| `aggregator`         | Value reported for each empty bucket |
| -------------------- | ------------------------------------ |
| `sum`, `count`       | `0`                                  |
| `last`               | The value of the last sample before the bucket's start. `NaN` when no such sample. |
| `twa`                | Average value over the bucket's timeframe based on linear interpolation of the last sample before the bucket's start and the first sample after the bucket's end. `NaN` when no such samples. |
| `min`, `max`, `range`, `avg`, `first`, `std.p`, `std.s` | `NaN` |

Regardless of the values of `fromTimestamp` and `toTimestamp`, no data is reported for buckets that end before the earliest sample or begin after the latest sample in the time series.
</details>

<details open>
<summary><code>GROUPBY label REDUCE reducer</code> (since RedisTimeSeries v1.6)</summary>

splits time series into groups, each group contains time series that share the same value for the provided label name, then aggregates results in each group.

When combined with `AGGREGATION` the `GROUPBY`/`REDUCE` is applied post aggregation stage.

  - `label` is label name. A group is created for all time series that share the same value for this label.

  - `reducer` is an aggregation type used to aggregate the results in each group.

    | `reducer` | Description                                                                                     |
    | --------- | ----------------------------------------------------------------------------------------------- |
    | `avg`     | Arithmetic mean of all non-NaN values (since RedisTimeSeries v1.8)                              |
    | `sum`     | Sum of all non-NaN values                                                                       |
    | `min`     | Minimum non-NaN value                                                                           |
    | `max`     | Maximum non-NaN value                                                                           |
    | `range`   | Difference between maximum non-NaN value and minimum non-NaN value (since RedisTimeSeries v1.8) |
    | `count`   | Number of non-NaN values (since RedisTimeSeries v1.8)                                           |
    | `std.p`   | Population standard deviation of all non-NaN values (since RedisTimeSeries v1.8)                |
    | `std.s`   | Sample standard deviation of all non-NaN values (since RedisTimeSeries v1.8)                    |
    | `var.p`   | Population variance of all non-NaN values (since RedisTimeSeries v1.8)                          |
    | `var.s`   | Sample variance of all non-NaN values (since RedisTimeSeries v1.8)                              |

<note><b>Notes:</b> 
  - The produced time series is named `<label>=<value>`
  - The produced time series contains two labels with these label array structures:
    - `__reducer__`, the reducer used (e.g., `"count"`)
    - `__source__`, the list of time series keys used to compute the grouped series (e.g., `"key1,key2,key3"`)
</note>
</details>

<note><b>Note:</b> An `MREVRANGE` command cannot be part of a transaction when running on a Redis cluster.</note>

## Examples

<details open>
<summary><b>Retrieve maximum stock price per timestamp</b></summary>

Create two stocks and add their prices at three different timestamps.

{{< highlight bash >}}
127.0.0.1:6379> TS.CREATE stock:A LABELS type stock name A
OK
127.0.0.1:6379> TS.CREATE stock:B LABELS type stock name B
OK
127.0.0.1:6379> TS.MADD stock:A 1000 100 stock:A 1010 110 stock:A 1020 120
1) (integer) 1000
2) (integer) 1010
3) (integer) 1020
127.0.0.1:6379> TS.MADD stock:B 1000 120 stock:B 1010 110 stock:B 1020 100
1) (integer) 1000
2) (integer) 1010
3) (integer) 1020
{{< / highlight >}}

You can now retrieve the maximum stock price per timestamp.

{{< highlight bash >}}
127.0.0.1:6379> TS.MREVRANGE - + WITHLABELS FILTER type=stock GROUPBY type REDUCE max
1) 1) "type=stock"
   2) 1) 1) "type"
         2) "stock"
      2) 1) "__reducer__"
         2) "max"
      3) 1) "__source__"
         2) "stock:A,stock:B"
   3) 1) 1) (integer) 1020
         2) 120
      2) 1) (integer) 1010
         2) 110
      3) 1) (integer) 1000
         2) 120
{{< / highlight >}}

The `FILTER type=stock` clause returns a single time series representing stock prices. The `GROUPBY type REDUCE max` clause splits the time series into groups with identical type values, and then, for each timestamp, aggregates all series that share the same type value using the max aggregator.
</details>

<details open>
<summary><b>Calculate average stock price and retrieve maximum average</b></summary> 

Create two stocks and add their prices at nine different timestamps.

{{< highlight bash >}}
127.0.0.1:6379> TS.CREATE stock:A LABELS type stock name A
OK
127.0.0.1:6379> TS.CREATE stock:B LABELS type stock name B
OK
127.0.0.1:6379> TS.MADD stock:A 1000 100 stock:A 1010 110 stock:A 1020 120
1) (integer) 1000
2) (integer) 1010
3) (integer) 1020
127.0.0.1:6379> TS.MADD stock:B 1000 120 stock:B 1010 110 stock:B 1020 100
1) (integer) 1000
2) (integer) 1010
3) (integer) 1020
127.0.0.1:6379> TS.MADD stock:A 2000 200 stock:A 2010 210 stock:A 2020 220
1) (integer) 2000
2) (integer) 2010
3) (integer) 2020
127.0.0.1:6379> TS.MADD stock:B 2000 220 stock:B 2010 210 stock:B 2020 200
1) (integer) 2000
2) (integer) 2010
3) (integer) 2020
127.0.0.1:6379> TS.MADD stock:A 3000 300 stock:A 3010 310 stock:A 3020 320
1) (integer) 3000
2) (integer) 3010
3) (integer) 3020
127.0.0.1:6379> TS.MADD stock:B 3000 320 stock:B 3010 310 stock:B 3020 300
1) (integer) 3000
2) (integer) 3010
3) (integer) 3020
{{< / highlight >}}

Now, for each stock, calculate the average stock price per a 1000-millisecond timeframe, and then retrieve the stock with the maximum average for that timeframe in reverse direction.

{{< highlight bash >}}
127.0.0.1:6379> TS.MREVRANGE - + WITHLABELS AGGREGATION avg 1000 FILTER type=stock GROUPBY type REDUCE max
1) 1) "type=stock"
   2) 1) 1) "type"
         2) "stock"
      2) 1) "__reducer__"
         2) "max"
      3) 1) "__source__"
         2) "stock:A,stock:B"
   3) 1) 1) (integer) 3000
         2) 310
      2) 1) (integer) 2000
         2) 210
      3) 1) (integer) 1000
         2) 110
{{< / highlight >}}
</details>

<details open>
<summary><b>Group query results</b></summary>

Query all time series with the metric label equal to `cpu`, then group the time series by the value of their `metric_name` label value and for each group return the maximum value and the time series keys (_source_) with that value.

{{< highlight bash >}}
127.0.0.1:6379> TS.ADD ts1 1548149180000 90 labels metric cpu metric_name system
(integer) 1548149180000
127.0.0.1:6379> TS.ADD ts1 1548149185000 45
(integer) 1548149185000
127.0.0.1:6379> TS.ADD ts2 1548149180000 99 labels metric cpu metric_name user
(integer) 1548149180000
127.0.0.1:6379> TS.MREVRANGE - + WITHLABELS FILTER metric=cpu GROUPBY metric_name REDUCE max
1) 1) "metric_name=system"
   2) 1) 1) "metric_name"
         2) "system"
      2) 1) "__reducer__"
         2) "max"
      3) 1) "__source__"
         2) "ts1"
   3) 1) 1) (integer) 1548149185000
         2) 45
      2) 1) (integer) 1548149180000
         2) 90
2) 1) "metric_name=user"
   2) 1) 1) "metric_name"
         2) "user"
      2) 1) "__reducer__"
         2) "max"
      3) 1) "__source__"
         2) "ts2"
   3) 1) 1) (integer) 1548149180000
         2) 99
{{< / highlight >}}
</details>

<details open>
<summary><b>Filter query by value</b></summary>

Query all time series with the metric label equal to `cpu`, then filter values larger or equal to 90.0 and smaller or equal to 100.0.

{{< highlight bash >}}
127.0.0.1:6379> TS.ADD ts1 1548149180000 90 labels metric cpu metric_name system
(integer) 1548149180000
127.0.0.1:6379> TS.ADD ts1 1548149185000 45
(integer) 1548149185000
127.0.0.1:6379> TS.ADD ts2 1548149180000 99 labels metric cpu metric_name user
(integer) 1548149180000
127.0.0.1:6379> TS.MREVRANGE - + FILTER_BY_VALUE 90 100 WITHLABELS FILTER metric=cpu
1) 1) "ts1"
   2) 1) 1) "metric"
         2) "cpu"
      2) 1) "metric_name"
         2) "system"
   3) 1) 1) (integer) 1548149180000
         2) 90
2) 1) "ts2"
   2) 1) 1) "metric"
         2) "cpu"
      2) 1) "metric_name"
         2) "user"
   3) 1) 1) (integer) 1548149180000
         2) 99
{{< / highlight >}}
</details>

<details open>
<summary><b>Query using a label</b></summary>

Query all time series with the metric label equal to `cpu`, but only return the team label.

{{< highlight bash >}}
127.0.0.1:6379> TS.ADD ts1 1548149180000 90 labels metric cpu metric_name system team NY
(integer) 1548149180000
127.0.0.1:6379> TS.ADD ts1 1548149185000 45
(integer) 1548149185000
127.0.0.1:6379> TS.ADD ts2 1548149180000 99 labels metric cpu metric_name user team SF
(integer) 1548149180000
127.0.0.1:6379> TS.MREVRANGE - + SELECTED_LABELS team FILTER metric=cpu
1) 1) "ts1"
   2) 1) 1) "team"
         2) (nil)
   3) 1) 1) (integer) 1548149185000
         2) 45
      2) 1) (integer) 1548149180000
         2) 90
2) 1) "ts2"
   2) 1) 1) "team"
         2) (nil)
   3) 1) 1) (integer) 1548149180000
         2) 99
{{< / highlight >}}
</details>

## Return information

{{< multitabs id="ts-mrevrange-return-info"
    tab1="RESP2"
    tab2="RESP3" >}}

If `GROUPBY label REDUCE reducer` is not specified:

[Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): for each time series matching the specified filters, the following is reported:
- [Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}}): The time series key name
- [Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): label-value pairs ([Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}}), [Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}}))
  - By default, an empty array is reported
  - If `WITHLABELS` is specified, all labels associated with this time series are reported
  - If `SELECTED_LABELS label...` is specified, the selected labels are reported
- [Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): timestamp-value pairs ([Integer reply]({{< relref "/develop/reference/protocol-spec#integers" >}}), [Simple string reply]({{< relref "/develop/reference/protocol-spec#simple-strings" >}})) representing all samples/aggregations matching the range in reverse chronological order

If `GROUPBY label REDUCE reducer` is specified:

[Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): for each group of time series matching the specified filters, the following is reported:
- [Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}}) with the format `label=value` where `label` is the `GROUPBY` label argument
- [Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): a single pair ([Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}}), [Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}})): the `GROUPBY` label argument and value
- [Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): a single pair ([Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}}), [Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}})):  the string `__reducer__` and the reducer argument
- [Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): a single pair ([Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}}), [Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}})): the string `__source__` and the time series key names separated by ","
- [Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): timestamp-value pairs ([Integer reply]({{< relref "/develop/reference/protocol-spec#integers" >}}), [Simple string reply]({{< relref "/develop/reference/protocol-spec#simple-strings" >}})) representing all samples/aggregations matching the range in reverse chronological order

-tab-sep-

If `GROUPBY label REDUCE reducer` is not specified:

[Map reply]({{< relref "/develop/reference/protocol-spec#maps" >}}): for each time series matching the specified filters, the following is reported:
- [Bulk string reply]({{< relref "/develop/reference/protocol-spec#bulk-strings" >}}): The time series key name
- [Map reply]({{< relref "/develop/reference/protocol-spec#maps" >}}) or [Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): label-value pairs
  - By default, an empty map is reported
  - If `WITHLABELS` is specified, all labels associated with this time series are reported as a map
  - If `SELECTED_LABELS label...` is specified, the selected labels are reported as a map
- Additional metadata including aggregators information
- [Array reply]({{< relref "/develop/reference/protocol-spec#arrays" >}}): timestamp-value pairs ([Integer reply]({{< relref "/develop/reference/protocol-spec#integers" >}}), [Double reply]({{< relref "/develop/reference/protocol-spec#doubles" >}})) representing all samples/aggregations matching the range in reverse chronological order

If `GROUPBY label REDUCE reducer` is specified:

Similar structure as RESP2 but with map-based organization for labels and metadata, and double values instead of string values.

{{< /multitabs >}}

## See also

[`TS.MRANGE`]({{< relref "commands/ts.mrange/" >}}) | [`TS.RANGE`]({{< relref "commands/ts.range/" >}}) | [`TS.REVRANGE`]({{< relref "commands/ts.revrange/" >}}) 

## Related topics

[RedisTimeSeries]({{< relref "/develop/data-types/timeseries/" >}})
