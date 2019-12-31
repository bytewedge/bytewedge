# bytewedge
ByteWedge is a schema agnostic Online Analytical Processing system designed for append only time series data. The highlights are:
* Schema agnostic. Upload a JSON, search, aggregate. DONE!
* Low total cost ownership. Data storage requires less than 50% data size. For highly repetitive data (machine generated data), storage can be as low as 5% of the original data size.
* Supports UTF-8 encoded full text search. Prefix, suffix, exact match, wild card, English, Chinese...
* Supports GraphQL with, group by, order by and  aggregation functions, like avg, min, max, sum, count, quantile(histogram).
* Supports [CEL](https://opensource.google/projects/cel) (except builtin macros) in where clause

# live demo
Please refer to [Wiki](https://github.com/bytewedge/bytewedge/wiki) for detailed information. ByteWedge uses tags (up to 10) to identify data source. In [demo](http://ui.demo.bytewedge.com/) cluster, [Two files](https://github.com/bytewedge/bytewedge/tree/master/demo) have been uploaded. Note: both have **demo** tag.

| name  | origin  |  tag 1  | tag 2   |
|---|---|---|---|
| gevents.json | [github event api snapshot](https://api.github.com/events) | github | demo |   |   |
| countries-unescaped.json | [countries](https://github.com/mledoze/countries/blob/master/dist/countries-unescaped.json)|countries|demo|

Note: ByteWedge uses dotted json path notation.

### demo 1
returns count(actor.id) from data source tagged with "github", group and order by payload.push_id.
```graphql
query {
  json(
    bucket: { tags: ["github"], range: { begin: 0, end: 15771186448655 } }
    select: [
      { path: "actor.id", aggr: count }
      { path: "payload.push_id", group: true, sort: asc }
    ]
    limit: 40
  )
}
```
### demo 2
returns name currencies and capital from data source tagged with "countries" where full text search on translations.zho.common contains '阿鲁巴'
```graphql
query {
  json(
    bucket: { tags: ["countries"], range: { begin: 0, end: 15771186448655 } }
    select: [{ path: "name" }, {path: "currencies"}, {path: "capital"}]
    where: "translations.zho.common.contains('阿鲁巴')"
    limit: 40
  )
}

```
### demo 3 (query with **demo** tag)
returns translations of a country and actor in github from data source tagged "demo" where country area equals 180 and github user id contains character "1". (it doesn't make any sense, ByteWedge can handle it)
```graphql
query {
  json(
    bucket: { tags: ["demo"], range: { begin: 0, end: 15771186448655 } }
    select: [{ path: "translations" }, { path: "actor" }]
    where: "id.contains('1') || area == 180"
    limit: 40
  )
}
```
