# Connecting to a SpatiaLite database with ogr2ogr

In retrospect, this was very easy, but it's not documented in an obvious way.

I have a SQLite database with SpatiaLite enabled, and I've loaded lots of spatial data, like from [AllThePlaces](https://github.com/eyeseast/alltheplaces-datasette). I want to dump that out to a file. If it's a format GDAL can read, I can use ogr2ogr, like this:

```sh
ogr2ogr -f GeoJSON -sql 'select * from places limit 100' places.geojson alltheplaces.db
```

That will create a GeoJSON file called `places.geojson` with the first 100 rows in the `places` table. I can output other formats (like [FlatGeoBuf](https://flatgeobuf.org/)) and use other arguments to set a projection. I can read SQL from a file, which is useful if I'm using [datasette-query-files](https://github.com/eyeseast/datasette-query-files). This is very promising.
