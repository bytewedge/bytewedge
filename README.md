# ByteWedge
A schema agnostic Online Analytical Processing system designed and built for append only time series data. ByteWedge at its core is a self compressed index, [FM-Index](https://en.wikipedia.org/wiki/FM-index). Data and index are colocated and compressed in sublinear space. With carefully designed succinct data structuce, ByteWedge can answer complex queires with [limited network/disk IO](https://github.com/bytewedge/bytewedge/wiki/A-Byte's-Life). Here are some of the highlights:

* Schema agnostic.
* Flexible query with tags. 
* Low total cost ownership. Data storage can be as low as **5% of the actual data**.
* Supports UTF-8 encoded full text search. Prefix, suffix, exact match, wild card, English, Chinese...
* Supports GraphQL, group by, order by and  aggregation functions, like avg, min, max, sum, count, quantile(histogram).
* Supports [CEL](https://opensource.google/projects/cel) (except builtin macros) in where clause

With the proliferation of iots, robotics and machines, so is the exponential growth of machine generated data, and the types data they generate. ByteWedge offers business an innovative tool to manage/store/analyze those data with ultra low TCO. ByteWedge employs scale out architecture, please refer to [Wiki](https://github.com/bytewedge/bytewedge/wiki) for details. 
```
┌─────────────ByteWedge Kubernetes Namespace───────┐                                                   
│                                                  │                  ┌───Customer Data Center────────┐
│  ┌──────cockroachdb───────┐                      │                  │ ┌───────────┐                 │
│  │      BLOB Store        │◀──────────────┐      │                  │ │ Wedge A   .      ┌──────┐   │
│  └───────▲────────────────┘               │      │                  │ │          ( )◀────│json A│   │
│   ┌──────┼───┐                            │   wss://mqtt    ┌───────┤ │     tag A '      └──────┘   │
│   │┌─────┴─┬─┴┐         .───.        ┌────────┐  │          │       │ │     ...   │                 │
│┌─▶││┌──────┼──┴┐       (     )       │I┌──────┴─┐.          │       │ └───────────┘                 │
│├──┴▶│┌─────┴───┴┐      │`───'│◀──────│ │Injector( )◀────────┤       │                               │
│├───▶┤│ Query    │      │DB/HA│       └─┤  ...   │'          │       │ ┌───────────┐                 │
│├────┴▶ Executor │      └──▲──┘         └────────┘│          │       │ │ Wedge B   .      ┌──────┐   │
││     └──────────┘         │                      │          │       │ │          ( )◀────│json B│   │
││                          │                      │          └───────┤ │     tag B '      └──────┘   │
││                          │                      │                  │ │     ...   │                 │
││                          │                      │                  │ └───────────┘                 │
││                        ┌─┴───┐                  │                  │                               │
││                        │ ┌───┴─┐                │                  │                               │
│└────dns load balance────┴─┤ API │                │                  └───────────────────────────────┘
│                           └─────┘                │                                                   
└─────────────────────────────( )──────────────────┘                                                   
                               '                                                                       
                         https://graphql
```
# The difference from other systems
Like all systems, ByteWedge tries to limit IO and make data available in main memory for computation. The followings are ByteWedge does differently from others.
1. Indexing storage. In ByteWedge, data and index are colocated and compressed. Other systems need separate storage for data and index.
2. Indexing time. In ByteWedge, indexing is done at Edge, instantly usable in its compressed form. Other systems are building index on the server.
3. Inherently secure. Unlike other system where data and index are in clear text. ByteWedge stores data and index in binary format that can only recognized by ByteWedge.
4. Flexible data source selection. ByteWedge uses tags to label data sources, user can mix and match at query time. Where other systems can only search among predefined index.
5. ETL is optional. ByteWedge doesn't need expensive ETL. With embedding hierarchy tree in its succinct data structure, ByteWedge can answer queries in different shape and sizes. Other systems require ETL to slice and dice data before the data is queriable. 
6. Low IO. ByteWedge requires [much fewer IO](https://github.com/bytewedge/bytewedge/wiki/A-Byte's-Life), where others need to shuttle data during either ETL or query processing.
7. Advanced full text search. ByteWedge supports prefix, suffix, wildcard etc without any resource constraint.
8. Support Agile development. ByteWedge is schema agnostic, add/delete/change data fields requires no actions. In other systems, such change requires updating ETL pipeline or rebuilding entire index.

# live demo
ByteWedge uses tags (up to 10) to identify data source. In [demo](http://ui.demo.bytewedge.com/) cluster, [Two files](https://github.com/bytewedge/bytewedge/tree/master/demo) have been uploaded. Note: both have **demo** tag.

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

## Install Free Community Edition
[Instructions](https://github.com/bytewedge/bytewedge/tree/master/charts)

* Patent Pending : A distributed system architect design for time series data analytics
