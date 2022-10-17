# Generating bounding boxes with SpatiaLite

I needed an array of US states and their bounding boxes for a project. I already had a SpatiaLite database going, and a datasette instance for analysis.

## Getting state boundaries

The US Census has these, but I can never remeber where and it's annoying to download zipped shapefiles, unzip them (and maybe turn them into GeoJSON) and load them into a database.

Fortunately, the LA Times data desk has solved this with [census-map-downloader](https://github.com/datadesk/census-map-downloader), which gets boundary files and turns them into GeoJSON. It pairs perfectly with [geojson-to-sqlite](https://github.com/simonw/geojson-to-sqlite).

I added this to my Makefile:

```make
data/states_carto_2018.geojson:
	pipenv run censusmapdownloader --data-dir $(dir $@) states-carto
	mv $(dir $@)processed/* $(dir $@)
```

Now I have state boundaries. SpatiaLite has a couple ways to get a bounding box. There's `extent`, which is an aggregate function. In this case, it'll get me a rectange covering the whole United States, or any query. Not what I want.

There's `BuildMbr`, to build a minimum-bounding rectangle. That's good if I have corners and need to draw a box.

In this case, I have polygons and need a box around each. `Envelope` will return a full geometry representing a bounding box -- that is, it takes a polygon and returns another polygon that happens to be a rectangle. Cool.

What if I just want the corners of a bounding box, of the kind I can pass to a `fitBounds()` method in Mapbox or Leaflet? The first thing I tried was getting each corner on its own, like this:

```sql
select
    geoid,
    name,
    MbrMinX(geometry) as x1,
    MbrMinY(geometry) as y1,
    MbrMaxX(geometry) as x2,
    MbrMaxY(geometry) as y2,
    AsGeoJSON(Envelope(geometry)) as geometry
from
    states
```

So that works. The `MbrMinX` and similar (`min`, `max`, `X`, `Y` variants) each get a corner of a minimum bounding rectangle. What if don't need the full geometry, though? What if I want those as an array?

```sql
select
    geoid,
    name,
    slugify(name) as slug,
    AsGeoJSON(geometry, 4, 1) ->> 'bbox' as bbox
from
    states
order by
    name
```

It turns out `AsGeoJSON` takes up to three positional arguments:

```
AsGeoJSON( geom Geometry [ , precision Integer [ , options Integer ] ] ) : String
```

So this line

```sql
AsGeoJSON(geometry, 4, 1) ->> 'bbox' as bbox
```

tells `AsGeoJSON` to use four digits of precision and to include a `bbox` on the returned GeoJSON object. The `->>` extracts the BBOX. The final data looks like this:

| geoid | name           | slug           | bbox                                    |
| ----- | -------------- | -------------- | --------------------------------------- |
| 01    | Alabama        | alabama        | [-88.4732,30.2233,-84.8891,35.008]      |
| 02    | Alaska         | alaska         | [-179.1489,51.2142,179.7785,71.3652]    |
| 60    | American Samoa | american-samoa | [-171.0899,-14.5487,-168.1433,-11.0469] |
| 04    | Arizona        | arizona        | [-114.8165,31.3322,-109.0452,37.0043]   |
| 05    | Arkansas       | arkansas       | [-94.6179,33.0041,-89.6444,36.4996]     |
| 06    | California     | california     | [-124.4096,32.5342,-114.1312,42.0095]   |
