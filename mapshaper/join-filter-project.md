# Join, filter, project in one step

I had a dataset of parcels and a spreadsheet of property values I need to join. [Mapshaper](https://github.com/mbloch/mapshaper/wiki/Command-Reference#-join) to the rescue:

```makefile
data/parcels.geojson: data/Parcels/Parcels.shp data/parcels.csv
	mapshaper data/Parcels/Parcels.shp \
	 -join data/parcels.csv keys=PIN,Parcel field-types=PIN:str,Parcel:str \
	 -filter 'Boolean(Street)' \
	 -proj wgs84 \
	 -o $@
```

[Here's the result.](https://www.greenvilleonline.com/in-depth/news/local/2021/05/04/historic-mill-redevelopment-has-changed-greenville-neighborhoods/7203703002/#asset-4942027001)
