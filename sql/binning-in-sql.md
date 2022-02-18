# Binning data in SQL

I added a feature to [datasette-geojson-map](https://github.com/eyeseast/datasette-geojson-map) letting me use Mapbox's [simplestyle-spec](https://github.com/mapbox/simplestyle-spec/tree/master/1.1.0) to style features.

That meant I could generate a choropleth map of a dataset using a SQL query, if I could figure out how to group the data into bins. Here's a version of the query that did it:

```sql
with bucketed as (
  select
    *,
    ntile(5) over (
      order by
        total_pop
    ) as bucket
  from
    data
)
select
  census_tract,
  total_pop,
  bucket,
  CASE
    bucket
    WHEN 1 THEN "#edf8fb"
    WHEN 2 THEN "#b2e2e2"
    WHEN 3 THEN "#66c2a4"
    WHEN 4 THEN "#2ca25f"
    WHEN 5 THEN "#006d2c"
  END fill
from
  bucketed
```

I went through a couple versions of this before figuring out that I needed the `WITH` clause. This version failed:

```sql
select
  data.*,
  NTILE(5) OVER (
    order by
      data.total_pop
  ) bucket,
  CASE
    bucket
    WHEN 1 THEN "#edf8fb"
    WHEN 2 THEN "#b2e2e2"
    WHEN 3 THEN "#66c2a4"
    WHEN 4 THEN "#2ca25f"
    WHEN 5 THEN "#006d2c"
  END fill,
  tracts.geometry
from
  canopy
  join tracts on data.census_tract = tracts.GEOID
order by
  total_pop
```

I'd get a `no such column: bucket` error. After a bunch of searching, I landed on this [StackOverflow post](https://stackoverflow.com/questions/13997177/why-no-windowed-functions-in-where-clauses) from nine years ago. As I understand it, the issue is that the database needs to get a query back with the result of the window function (`NTILE(5)` in this case) before it can operate on that column with `CASE` (or `WHERE` or anything else).

In one query, I've used `WITH`, `CASE` and `NTILE` all for the first time.
