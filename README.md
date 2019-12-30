# bytewedge
ByteWedge is a schema agnostic Online Analytical Processing system designed for append only time series data. The highlights are:
* Schema agnostic. Upload a JSON, search, aggregate. DONE!
* Low total cost ownership. Data storage requires less than 50% data size. For highly repetitive data (machine generated data), storage can be as low as 5% of the original data size.
* Supports UTF-8 encoded full text search. Prefix, suffix, exact match, wild card, English, Chinese...
* Supports GraphQL with, group by, order by and  aggregation functions, like avg, min, max, sum, count, quantile(histogram).
* Supports [CEL](https://opensource.google/projects/cel) (except builtin macros) in where clause
