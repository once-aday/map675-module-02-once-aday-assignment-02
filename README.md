# map675-module-02-once-aday-assignment-02



**Planning and Data Collection**

I'm interested in mapping Native American land in North America. I will also look into geology in similar areas and see how different rock types may change based on native territories.

Data Sources:

https://native-land.ca/api-docs/ - (The API for native-land.ca)

Oregon Shapefiles (incl. County boundaries): https://www.oregon.gov/geo/Pages/census.aspx
References:

https://native-land.ca/

https://www.gislounge.com/mapping-native-lands/

http://www.tribalnationsmaps.com/

OGDC-6, Oregon Geologic Data Compilation, release 6, compiled by Rachel L. Smith and Warren P. Roe.
https://www.oregongeology.org/pubs/dds/p-OGDC-6.htm

Notes:

 Aaron Capella, creator of Tribal Nations Maps, speaking about the native-land.ca project:

> "This map is in honor of all the Indigenous Nations [of colonial states]. It seeks to encourage people — Native and non-Native — to remember that these were once a vast land of autonomous Native peoples, who called the land by many different names according to their languages and geography. The hope is that it instills pride in the descendants of these People, brings an awareness of Indigenous history and remembers the Nations that fought and continue to fight valiantly to preserve their way of life."

Intersections Between Tribal Land, Geology, Oregon Counties...

**Cleaning up and Analyzing the Geologic Data**

The OGDC-6, Oregon Geologic Data Compilation had to be downloaded as a Geodatabase archive, which I opened in QGIS.

The Source CRS is EPSG:2994 (NAD83(HARN) / Oregon GIC Lambert (ft)). It is translated to WGS84 (EPSG:4326) when imported into QGIS.

The layer MapUnitPolygon is the major polygon layer that contains the geologic units and ages for the entire state of Oregon.

MapUnitPolygon Attributes:

Field Name | Description
---------- | -----------
REF_ID_COD | Unqie Id Code For Original Ref Map
MAP_UNIT_L | Map unit label
MAP_UNIT_N | Map unit name
G_MRG_U_L | Geol. Merge Unit; combines original map units into 7 gen. classifications
GEO_GENL_U | Geol. Merge Unit expressed by Genesis of the unit
AGE_NAME | Age of Merge Unit
TERRANE_GR | Stratigraphic Name (formally or informally)
FORMATION | Stratigraphic Formation Name
MEMBER | Stratigraphic Member
UNIT | Stratigraphic unit name
G_ROCK_TYP | Characteristic rock type for for Geol. Merge Unit
LITH_M_U_L | Label for lithologic merge unit
LITH_GEN_U | Lithologic general unit
LTH_RK_TYP | Lithologic Rock Type
LAYERING | Layering/Rock stratum info from original source data for lithologic merge unit
CR_GRN_SIZ | Crystal Grain/Size
GETEC_PROP | Geotechnical Properties, Rock,Structural props for Lithologic merge units
GN_LITH_TY | General Lithology Type

The Shapefile is very large, 186 MB. I will try to simplify it and only maintain a few attribute columns.

```
Devins-MBP:data devin$ mapshaper OregonGeologyMapUnitsPoly.shp -info
[info]
==================================================
Layer:    OregonGeologyMapUnitsPoly
--------------------------------------------------
Type:     polygon
Records:  120,049
Bounds:   221868.40551181138,91115.40387138724,2345213.183727041,1659913.3198818862
CRS:      +proj=lcc +lat_1=43 +lat_2=45.5 +lat_0=41.75 +lon_0=-120.5 +x_0=399999.99999840005 +to_me
ter=0.3048 +ellps=GRS80
Source:   OregonGeologyMapUnitsPoly.shp


Attribute data
------------+-------------------------------------
 Field      | First value
------------+-------------------------------------
 AGE_NAME   | 'Miocene'
 ArcJoinKey | 'AchJA1991Tdc'
 CR_GRN_SIZ | 'no data'
 DataSource |       1
 FORMATION  | 'Devine Canyon Welded Ashflow Tuff'
 G_MRG_U_L  | 'vlc.M.hbv.dv.nd.nd.aft'
 G_ROCK_TYP | 'ashflow tuff'
 GEO_GENL_U | 'volcaniclastic rocks'
 GETEC_PROP | 'no data'
 GN_LITH_TY | 'volcanic'
 IDConf     |      99
 LAYERING   | 'massive'
  LITH_GEN_U | 'volcaniclastic rocks'
  LITH_M_U_L | 'vlc.TFF.Lma.Gnd.End'
  LTH_RK_TYP | 'tuff'
  MAP_UNIT_L | 'Tdc'
  MAP_UNIT_N | 'Devine Canyon Ash-flow Tuff'
  MapUnitPK  |    8395
  MEMBER     | 'No data'
  OBJECTID   |       1
  REF_ID_COD | 'AchJA1991'
  Shape_Area | 3221353.051100082
  Shape_Leng |   52995.1882956897
  TERRANE_GR | 'Harney Basin Volcanic Field'
  UNIT       | 'no data'

 ------------+-------------------------------------
 ```


I will reduce the dataset to only contain the General Lithology Type(**GN_LITH_TY**), General Rock Type(**G_ROCK_TYP**), Geol. Merge Unit expressed by Genesis of the unit (**GEO_GENL_U**), Geologic Age of Merge Unit (**AGE_NAME**). I'll include RED_ID_CODE as well for a unique id. I'm not exactly sure what analysis I will be doing so I may reduce this further or add other fields to it later.

```
mapshaper OregonGeologyMapUnitsPoly.shp -filter-fields REF_ID_COD,GN_LITH_TY,G_ROCK_TYP,GEO_GENL_U -o OregonGeologyMapUnitsPoly_reducefields.shp
```

Try again with Simplification, GeoJSON output, keep-shapes, and stats:

```
mapshaper OregonGeologyMapUnitsPoly.shp -filter-fields REF_ID_COD,GN_LITH_TY,G_ROCK_TYP,GEO_GENL_U -simplify dp 20% -stats -keepshapes -o format=geojson OregonGeologyMapUnitsPoly_reducefields.json
```

keep-shapes: to make sure none dissapear.
stats: for summary statistics

**Filter county shapefile to only Coos County**
Also convert to wgs84, filter resultant fields.

mapshaper counties.shp -info -filter-fields COUNTY -filter 'COUNTY == 033' -proj wgs84 -o format=geojson county_coos.json


**Clip Geology to Single County**

mapshaper or_geology.json -clip county_coos.json -o coos_geology.json

**Clean Native American Land Data**

```
Devins-MBP:data_pre devin$ mapshaper native_lands/indigenousTerritories.json -info
[info]
==================================================
Layer:    indigenousTerritories
--------------------------------------------------
Type:     polygon
Records:  1,487
Bounds:   -188.588688,-56.02359,178.54426,83.236426
CRS:      [unknown]
Source:   native_lands/indigenousTerritories.json

Attribute data
-------------------+------------------------------
 Field             | First value
-------------------+------------------------------
 color             | '#9022FF'
 description       | 'https://native-land.ca/maps/territories/acaxees-xiximes/'
 FID               | '007d3821e00f3a136fa7285091646eff'
 FrenchDescription | 'https://native-land.ca/maps/territories/acaxees-xiximes/'
 FrenchName        | 'Acaxees'
 Name              | 'Acaxees'
 Slug              | 'acaxees-xiximes'

-------------------+------------------------------
```

```
mapshaper native_lands/indigenousTerritories.json -clip county_coos.json -o indg_territories.json
```






**Simplify Geology to merge on General Lithology Type (GN_LITH_TY) or (G_ROCK_TYP)**

I want to reduce the complexity of the geologic data, so as to make bring it closer to the more large and broadly define indigenous territories polygons.

```
mapshaper coos_geology.json -dissolve fields=GN_LITH_TY -o coos_geology_gnlith_disslv.json
```

```
mapshaper coos_geology.json -dissolve fields=G_ROCK_TYP -o coos_geology_grocktp_disslv.json
```

Rock type gives more detail and is more interesting.

**Compare/Join Geology data to Native America Land**

```
mapshaper indg_territories.json -join coos_geology_grocktp_disslv.json -calc 'rock_types = collect(G_ROCK_TYP)' -o indg_ter_rock_type.json
```

I'm having issues joining the two and maintaining all the rocktypes, it seems to only give it one.

**Next.. Script or process the entire state**

option 1: Create Script to perform the analysis for each county, then create a web map that lets you cycle through counties or indg_territories

option2: run this process for the entire state, and let the user hove over the indigenous territories to see a list of rock types that are present in that area..
