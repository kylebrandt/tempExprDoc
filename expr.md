# Server Side Expressions

Server Side expressions allow you to manipulate data returned from queries with math and other operations. Expressions create new data and do not manipulate the data returned by data sources (with the exception of some data restructuring to make it acceptable input for expressions).

## Grafana 7.4 Status

Alpha, beta, preview ... whatever we call it it. The important thing is that we will be making breaking changes during this time. So, if a user creates an expression and saves it. That is expression may stop working in 7.x.x after an upgrade.

## Overview

The primary use case for expressions is with alerting. Like alerting, processing is done server side, so expressions can operate without a browser session. However, Expressions can be used with backend data sources and visualization as well.

Expressions work with data source queries that return time series or number data. They also operate on [multiple-dimensional data](https://grafana.com/docs/grafana/latest/getting-started/timeseries-dimensions/) (for example, a query that returns multiple series, where each series is identified by labels or tags).

An individual expression takes one or more queries or other expressions as input, and adds data to the result. Each individual expression or query is represented by a named identifer known as it a RefID (e.g., the default letter `A` or `B`). To reference the output of an individual expression or a data source query in another expression, this identifer is used.

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

### Math

- Input Type: Time Series or Numbers
- Output Type: Time Series or Numbers