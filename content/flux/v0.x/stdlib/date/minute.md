---
title: date.minute() function
description: >
  The `date.minute()` function returns the minute of a specified time.
  Results range from `[0-59]`.
aliases:
  - /influxdb/v2.0/reference/flux/functions/date/minute/
  - /influxdb/v2.0/reference/flux/stdlib/date/minute/
  - /influxdb/cloud/reference/flux/stdlib/date/minute/
menu:
  flux_0_x_ref:
    name: date.minute
    parent: date
weight: 301
introduced: 0.37.0
---

The `date.minute()` function returns the minute of a specified time.
Results range from `[0-59]`.

```js
import "date"

date.minute(t: 2019-07-17T12:05:21.012Z)

// Returns 5
```

## Parameters

### t {data-type="time, duration"}
The time to operate on.
Use an absolute time, relative duration, or integer.
Durations are relative to `now()`.

## Examples

##### Return the minute of a time value
```js
import "date"

date.minute(t: 2020-02-11T12:21:03.293534940Z)

// Returns 21
```

##### Return the minute of a relative duration
```js
import "date"

option now = () => 2020-02-11T12:21:03.293534940Z

date.minute(t: -45m)

// Returns 36
```
