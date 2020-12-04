# Server Side Expressions

Server Side expressions allow you to manipulate data returned from queries with math and other operations. Expressions create new data and do not manipulate the data returned by data sources (with the exception of some data restructuring to make it acceptable input for expressions).

## Overview

The primary use case for expressions is with alerting. Like alerting, processing is done server side, so expressions can operate without a browser session. However, Expressions can be used with backend data sources and visualization as well.

Expressions work with data source queries that return time series or number data. They also operate on [multiple-dimensional data](https://grafana.com/docs/grafana/latest/getting-started/timeseries-dimensions/) (for example, a query that returns multiple series, where each series is identified by labels or tags).

An individual expression takes one or more queries or other expressions as input, and adds data to the result. Each individual expression or query is represented by a named identifer known as it a RefID (e.g., the default letter `A` or `B`).