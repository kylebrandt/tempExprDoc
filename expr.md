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

Reduce takes one or more time series returned from a query or an expression and turns each series into a single number. The labels of the time series are kept as labels on each outputted reduced number.

### Resample

- Input Type: Time Series
- Output Type: Time Series

Resampling resamples each time series to have a consistent time interval. The main use case is so you can resample time series that do not share the same timestamps so math can be performed between them.

### Math

- Input Type: Time Series or Numbers
- Output Type: Time Series or Numbers

Math is for free form math formulas on time series or number data. Data from other queries or expressions are referenced with the RefID prefixed with a dollar sign, for example `$A`.

#### Operators

The arithmetic (`+`, binary and unary `-`, `*`, `/`, `%`), relational (`<`, `>`, `==`, `!=`, `>=`, `<=`), and logical (`&&`, `||`, and unary `!`) operators are supported.

How the operation behaves with data depends on if it is a number or time series data.

With binary operations, such as `$A + $B` or `$A || $B`, the operator is applied in the following ways depending on the type of data:

- If both `$A` and `$B` are a number, then the operation is performed between the two numbers.
- If one variable is a number, and the other variable is a time series, then the operation between the value of each point in the time series and the number is performed.
- If both `$A` and `$B` are time series data, then the operation between each value in the two series is performed for each time stamp that exists in both `$A` and `$B`. The Resample operation can be used to line up time stamps. (Note: in the future, we plan to add options to the Math operation for different behaviors).

Additionally, since expressions work with multiple series or numbers represented by a single variable, binary operations also perform a Union between the two variables. This is done based on the identifying labels associated with each individual series or number.