# Printing tables in the CLI

I've been testing how to read GIS files faster and printing results. Printing tables is tricky, because filenames and functions don't have a common length.

Fortunately, [python-tabulate](https://github.com/astanin/python-tabulate) exists. I found it in [this StackOverflow answer](https://stackoverflow.com/questions/9535954/printing-lists-as-tabular-data), and there are other options in that thread, too.

For printing [dataclasses](https://docs.python.org/3/library/dataclasses.html), I'm using `asdict`:

```python
# results is a list of Result objects, where Result is a dataclass
click.echo(tabulate(map(asdict, results), headers="keys", tablefmt="github"))
```

Comes out pretty nice:

| path                           | function           | count  | time           |
| ------------------------------ | ------------------ | ------ | -------------- |
| nhgis0038_ds172_2010_block.csv | read_csv           | 157509 | 0:00:00.869328 |
| ma-2010-blocks.geojson         | read_geojson       | 155463 | 0:00:12.225532 |
| ma-2010-blocks.geojson         | read_geojson_fiona | 155463 | 0:00:38.971221 |
| ma-2010-blocks.ndjson          | read_json_nl       | 155463 | 0:00:06.262876 |
| ma-2010-blocks.ndjson          | read_geojson_fiona | 1      | 0:00:00.242557 |
| MA_block_2010.shp              | read_shp           | 155463 | 0:00:07.643704 |
