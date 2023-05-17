# Computing a moving average in SQL

I'm still playing with wildfire data. Here's how to compute a three-year moving average of acres burned:

```sql
select
    FIRE_YEAR as year,
    count(*) as count,
    trunc(sum(GIS_ACRES)) as acres,
    trunc(
        avg(sum(GIS_ACRES)) over (
            order by
                FIRE_YEAR rows between 3 preceding
                and current row
        )
    ) as moving_average
from
    wildfires
where
    FIRE_YEAR_INT > 1970
    and FIRE_YEAR_INT != 9999
group by
    FIRE_YEAR_INT
order by
    FIRE_YEAR_INT
```

This is using historic wildfire perimeters from the [National InterAgency Fire Center](https://data-nifc.opendata.arcgis.com/datasets/nifc::interagencyfireperimeterhistory-all-years-view/about).

The key to getting a moving average is the `over` clause, which makes `avg` into a [window function](https://www.sqlite.org/windowfunctions.html). I cribbed from [this example](https://learnsql.com/blog/rolling-average-in-sql/) (and decided to stick with three years). This is similar to what I learned with [binning in SQL](./binning-in-sql.md).
