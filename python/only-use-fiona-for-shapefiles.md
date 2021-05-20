# Only use Fiona to read shapefiles

I'm a big fan of [Fiona](https://fiona.readthedocs.io/en/latest/manual.html). It's the easiest way to read a shapefile in Python. But it does say right up front that it's not the best choice for other formats:

> In what cases would you not benefit from using Fiona?
>
> - If your data is in or destined for a JSON document you should use Python’s json or simplejson modules.
> - If your data is in a RDBMS like PostGIS, use a Python DB package or ORM like SQLAlchemy or GeoAlchemy. Maybe you’re using GeoDjango already. If so, carry on.
> - If your data is served via HTTP from CouchDB or CartoDB, etc, use an HTTP package (httplib2, Requests, etc) or the provider’s Python API.
> - If you can use ogr2ogr, do so.

It turns out, this is good advice. I've been running [speed tests](https://github.com/eyeseast/gis-speed-tests). Here's the results:

| path                           | function           | count  | time           |
| ------------------------------ | ------------------ | ------ | -------------- |
| nhgis0038_ds172_2010_block.csv | read_csv           | 157509 | 0:00:00.869328 |
| ma-2010-blocks.geojson         | read_geojson       | 155463 | 0:00:12.225532 |
| ma-2010-blocks.geojson         | read_geojson_fiona | 155463 | 0:00:38.971221 |
| ma-2010-blocks.ndjson          | read_json_nl       | 155463 | 0:00:06.262876 |
| ma-2010-blocks.ndjson          | read_geojson_fiona | 1      | 0:00:00.242557 |
| MA_block_2010.shp              | read_shp           | 155463 | 0:00:07.643704 |

Reading CSV is fast. Reading newline-delimited GeoSJON is about twice as fast as reading a whole feature collection in one shot.

Using Fiona to read a shapefile is _almost_ as fast as newline-delimited GeoSJON, but Fiona takes _more than five times as long_ to read a GeoJSON file. It takes three times as long to read through a JSON file as plain Python. For ND-JSON, it just stopped after the first feature. Oops.

Fiona is great. You should use it. But also read the docs about what it's best at.
