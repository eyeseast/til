# Using a spatial index with an attached database

SpatiaLite's [spatial index](https://www.gaia-gis.it/fossil/libspatialite/wiki?name=SpatialIndex) is both weird and essential. Weird, because you have to explicitly search it using a subquery, and essential because it speeds up spatial queries by an order of magnitude. [This query](https://alltheplaces-datasette.fly.dev/alltheplaces/dunkin_in_suffolk) is a good example.

SQLite _also_ lets you [attach one database to another](https://www.sqlite.org/lang_attach.html) (because they're just files) and search both. This can get useful in keeping data organized and partitioned. [Datasette has built-in support for this](https://docs.datasette.io/en/stable/sql_queries.html#cross-database-queries).

Can you use both together? Yes you can.

The trick is that you need to tell the subquery where to find the tables you're searching. Here's an example, joining a `wildfires` table in one database with a `states` table in another to find all the fires in California in 2021:

```sql
select
    FIRE_YEAR,
    INCIDENT,
    GIS_ACRES,
    w.geometry
from
    wildfires.wildfires as w,
    places.states as s
where
    FIRE_YEAR = '2021'
    and s.name = 'California'
    and within(w.geometry, s.geometry)
    and w.rowid in (
        select
            rowid
        from
            SpatialIndex
        where
            f_table_name = 'DB=wildfires.wildfires'
            and search_frame = s.geometry
    )
```

The key is this clause: `f_table_name = 'DB=wildfires.wildfires'`. That tells the spatial index to look for the `wildfires` table in the `wildfires` database. Behind the scenes, Datasette is running `ATTACH wildfires.db as wildfires`. Without `DB=wildfires`, the spatial index would look in its primary database, and the query would come back empty.
