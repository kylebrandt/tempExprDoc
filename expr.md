# Server Side Expressions

Server Side expressions allow you to manipulate data returned from queries with math and other operations. Expressions create new data and do not manipulate the data returned by data sources (with the exception of some data restructuring to make it acceptable input for expressions).

## Grafana 7.4 Status

Alpha, beta, preview ... whatever we call it it. The important thing is that we will be making breaking changes during this time. So, if a user creates an expression and saves it. That is expression may stop working in 7.x.x after an upgrade.

## Overview

The primary use case for expressions is with alerting. Like alerting, processing is done server side, so expressions can operate without a browser session. However, Expressions can be used with backend data sources and visualization as well.

Expressions work with data source queries that return time series or number data. They also operate on [multiple-dimensional data](https://grafana.com/docs/grafana/latest/getting-started/timeseries-dimensions/) (for example, a query that returns multiple series, where each series is identified by labels or tags).

An individual expression takes one or more queries or other expressions as input, and adds data to the result. Each individual expression or query is represented by a variable which is a named identifer known as it a RefID (e.g., the default letter `A` or `B`). To reference the output of an individual expression or a data source query in another expression, this identifer is used as a variable.

## Expression Types

Expressions work with two types of data. Either a collections of time series, or a collection of numbers (where each collection could be a single series or single number). Each collection is returned from a single data source query or expression, and represented by the RefID. Each collection is a set, where each item in the set is uniquely identified by it dimensions which are stored as [labels](https://grafana.com/docs/grafana/latest/getting-started/timeseries-dimensions/#labels) (key value pairs).

## Operations

### Reduce

- Input Type: Time Series
- Output Type: Numbers

Fields:

- Input: The variable (refID (e.g. `A`)) to resample
- Function: The reduction function to use

Reduce takes one or more time series returned from a query or an expression and turns each series into a single number. The labels of the time series are kept as labels on each outputted reduced number.

#### Reduction Functions

Note: In the future we plan to add options to control empty, NaN, and null behavior for reduction functions.

##### Count

Count returns the number of points in each series.

##### Mean

Mean returns the total of all values in each series divided by the number of points in that series. If any values in the series are null or nan, or if the series is empty, NaN is returned.

##### Min and Max

Min and Max return the smallest or largest value in the series respectively. If any values in the series are null or nan, or if the series is empty, NaN is returned.

##### Sum

Sum returns the total of all values in the series. If series is of zero length, the sum will be 0. If there are any NaN or Null values in the series, NaN is returned.

### Resample

- Input Type: Time Series
- Output Type: Time Series

Fields:

- Input: The variable of time series data (refID (e.g. `A`)) to resample
- Window: The duration of time to resample to, for example `10s`. Units may be `s` seconds, `m` for minutes, `h` for hours, `d` for days, `w` for weeks, and `y` of years.
- Downsample: The reduction function to use when there are more than one data point per window sample. See the reduction operation for behavior details.
- Upsample: The method to use to fill a window sample that has no data points: `pad` fills with the last know value, `backfill` with next known value, and `fillna` to fill empty sample windows with NaNs.

Resample changes the time stamps in each time series to have a consistent time interval. The main use case is so you can resample time series that do not share the same timestamps so math can be performed between them. This can be done by resample each of the two series, and then in a Math operation referencing the resampled variables.

### Math

- Input Type: Time Series or Numbers
- Output Type: Time Series or Numbers

Math is for free form math formulas on time series or number data. Data from other queries or expressions are referenced with the RefID prefixed with a dollar sign, for example `$A`.

Numeric constants may be in decimal (`2.24`), octal (with a leading zero like `072`), or hex (with a leading 0x like `0x2A`). Exponentials and signs are also supported (e.g., `-0.8e-2`).

#### Operators

The arithmetic (`+`, binary and unary `-`, `*`, `/`, `%`), relational (`<`, `>`, `==`, `!=`, `>=`, `<=`), and logical (`&&`, `||`, and unary `!`) operators are supported.

How the operation behaves with data depends on if it is a number or time series data.

With binary operations, such as `$A + $B` or `$A || $B`, the operator is applied in the following ways depending on the type of data:

- If both `$A` and `$B` are a number, then the operation is performed between the two numbers.
- If one variable is a number, and the other variable is a time series, then the operation between the value of each point in the time series and the number is performed.
- If both `$A` and `$B` are time series data, then the operation between each value in the two series is performed for each time stamp that exists in both `$A` and `$B`. The Resample operation can be used to line up time stamps. (Note: in the future, we plan to add options to the Math operation for different behaviors).

Additionally, since expressions work with multiple series or numbers represented by a single variable, binary operations also perform a union (join) between the two variables. This is done based on the identifying labels associated with each individual series or number. So if you have numbers with labels like `{host=web01}` in `$A` and another number in `$B` with the same labels then the operation is performed between those two items within each variable, and the result will share the same labels. The rules for the behavior of this union are as follows:

- An item with no labels will join to anything
- If both `$A` and `$B` each contain only one item (one series, or one number), they will join.
- If labels are exact math they will join.
- If labels are a subset of the other, for example and item in `$A` is labeled `{host=A,dc=MIA}` and and item in `$B` is labeled `{host=A}` they will join.
- Currently, if within a variable such as `$A` there are different tag _keys_ for each item, the join behavior is undefined.

## Data source queries

Data source queries need to be backend data sources. The data is generally assumed to be labeled time series data. In the future we intended to add an assertion of the query return type ("number" or "time series") data so expressions can handle errors better.

Data source queries, when used with expressions, are executed by the expression engine. When it does this, it restructures data to be either one time series or one number per data frame. So for example if using a data source that returns multiple series on one frame in the table view, you may notice it looks different when executed with expressions.

Currently, the only non time series format (number) is supported when using data frames are you have a table response that returns a data frame with no time, string columns, and one number column:

Loc | Host | Avg_CPU |
----|------| ------- |
MIA | A    | 1
NYC | B    | 2

will produce a number that works with expressions. The string columns become labels and the number column the corresponding value. For example `{"Loc": "MIA", "Host": "A"}` with a value of 1.

