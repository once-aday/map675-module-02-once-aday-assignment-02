# map675-module-02-once-aday-assignment-02



**Planning and Data Collection**

I'm interested in mapping Native American land in North America. I will also look into geology in similar areas and see if how different rock types may change based on native territories.

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

REF_ID_COD
MAP_UNIT_L : Map unit label
MAP_UNIT_N : Map unit name
G_MRG_U_L : Geol. Merge Unit; combines original map units into 7 gen. classifications
GEO_GENL_U : Geol. Merge Unit expressed by Genesis of the unit
AGE_NAME : Age of Merge Unit
TERRANE_GR : Stratigraphic Name (formally or informally)
FORMATION : Stratigraphic Formation Name
MEMBER : Stratigraphic Member
UNIT : Stratigraphic unit name
G_ROCK_TYP : Characteristic rock type for for Geol. Merge Unit
LITH_M_U_L : Label for lithologic merge unit
LITH_GEN_U : Lithologic general unit
LTH_RK_TYP : Lithologic Rock Type
LAYERING : Layering/Rock stratum info from original source data for lithologic merge unit
CR_GRN_SIZ : Crystal Grain/Size
GETEC_PROP : Geotechnical Properties, Rock,Structural props for Lithologic merge units
GN_LITH_TY : General Lithology Type

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

mapshaper OregonGeologyMapUnitsPoly.shp -filter-fields REF_ID_COD,GN_LITH_TY,G_ROCK_TYP,GEO_GENL_U -o OregonGeologyMapUnitsPoly_reducefields.shp

Try again with Simplification, GeoJSON output, keep-shapes, and stats:

mapshaper OregonGeologyMapUnitsPoly.shp -filter-fields REF_ID_COD,GN_LITH_TY,G_ROCK_TYP,GEO_GENL_U -simplify dp 20% -stats -keepshapes -o format=geojson OregonGeologyMapUnitsPoly_reducefields.json

keep-shapes: to make sure none dissapear.
stats: for summary statistics
