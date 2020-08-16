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

mapshaper indg_territories.json -join coos_geology_grocktp_disslv.json -calc 'rock_type = collect(G_ROCK_TYP)' -o terr_rock_join_multi.json


I'm having issues joining the two and maintaining all the rocktypes, it seems to only give it one.

Solution:

1. Join Geology to Territories
mapshaper coos_geology_grocktp_disslv.json -join indg_territories.json  -o rock_terr_join.json

2. Dissolve on Territory
mapshaper rock_terr_join.json -dissolve multipart fields=Name,G_ROCK_TYP -o rock_terr_dissolve.json

3. Get Stats on each territory

Get All Territory Names
mapshaper indg_territories.json -calc 'collect(Name)'

```
mapshaper rock_terr_dissolve.json -calc 'collect(G_ROCK_TYP)' where='Name=="Cowlitz"'

[calc] collect(G_ROCK_TYP) where Name=="Cowlitz":  andesite,felsic composition lithologies,conglomerate,sandst
one,mudflow breccia

```

Now I was able to get all of the rock types for the Cowlitz tribe in a single array. At this point, because my plan is to integrate this data with a javascript web map I will be doing the filtering of the different territories within the web map code rather than the commmand line. Mapshaper does not seem to have the ability to filter unique instances of feature attributes, or in the case, the "Name" attribute. No matter, I can do this in Javascript when I have the main dataset loaded into memory.

My intention is to list all of the unique territories, and then have a secondary list with their associated rock types. I can decipher this information using basic javascript functions once rock_terr_dissolve.json is loaded.






**Next.. Script or process the entire state**

option 1: Create Script to perform the analysis for each county, then create a web map that lets you cycle through counties or indg_territories

option2: run this process for the entire state, and let the user hove over the indigenous territories to see a list of rock types that are present in that area..

option3: Keep it to coos county


**Update to plan**

Instead of mapping rock types to indigenous territory, I am going to map rock types based on counties

Make OR geology GeoJSON; rocktypes, join to Counties

mapshaper or_geology.json -dissolve fields=G_ROCK_TYP -o or_geology_rocktyp
e.json

mapshaper counties.shp -join or_geology_rocktype.json -calc 'rock_types = collect(G_ROCK_TYP)' -o or_counties_rock_type.json

---- Third try -----

Split counties into individual Shapefiles

SETUP:
mapshaper counties.shp name='' -split COUNTY_NAM -o format=geojson


```
Devins-MacBook-Pro:county_splits devin$ ls
ADAMS.json                      KING.json                       TILLAMOOK.json
ASOTIN.json                     KITSAP.json                     UMATILLA.json
BAKER.json                      KITTITAS.json                   UNION.json
BENTON.json                     KLAMATH.json                    WAHKIAKUM.json
CHELAN.json                     KLICKITAT.json                  WALLA WALLA.json
CLACKAMAS.json                  LAKE.json                       WALLOWA.json

CLALLAM.json                    LANE.json                       WASCO.json
CLARK.json                      LEWIS.json                      WASHINGTON.json
CLATSOP.json                    LINCOLN.json                    WHATCOM.json
COLUMBIA.json                   LINN.json                       WHEELER.json
```

REPEATABLE:
clip geology by county

mapshaper or_geology_rocktype.json -clip COOS.json -o county_geology.json

join county name to geology

mapshaper coos_geology.json -each 'county="coos"' -o coos_geology_clip_join.csv

Generic: mapshaper coos_geology_clip_join.csv -each 'county="county"' -o county_geology_join.csv

(I could potentially merge all back together into one )


These were notes on how I might make this repeatable, more work would have to be done to export the csv, process the results and recombine all the rock types if I wanted that to happen before import into the web map.

For now I am going to take Curry, coos, and douglass, jackson Counties, and run that process, import that into the map.


mapshaper or_geology_rocktype.json -clip curry.json -o curry_geology.json

mapshaper or_geology_rocktype.json -clip douglas.json -o douglas_geology.json

mapshaper or_geology_rocktype.json -clip jackson.json -o jackson_geology.json

mapshaper or_geology_rocktype.json -clip coos.json -o coos_geology.json

Then I will make a POINT layer with the bedding angles on top of it, this will be clustered using leaflet.

convert to geojson:

mapshaper bedding.shp -o format=geojson


------

export geology layer that was dissolved on rock type to a csv so that I could get all unique rock types for reference.

mapshaper or_geology_rocktype.json -o or_geology_rocktype.csv delimiter=,


merge the 4 geology layers into one.

mapshaper coos_geology.json curry_geology.json douglas_geology.json jackson_geo
logy.json combine-files -merge-layers -o all_geology.json

Switch to only Curry and Coos COUNTY

mapshaper coos_geology.json curry_geology.json combine-files -merge-layers -o all_geology.json

Try random colors:

```
style: function (feature) {
  // use the colors object to style each polygon a unique color
  return {
    color: colors.Pastel["10"][Math.floor((Math.random() * 10) + 1)],
    fillOpacity: .6
  }
}
```

assign a number to each color in the geology GeoJSON
a
```
mapshaper all_geology.json -each 'color=this.id' -o all_geology_num.json
[o] Wrote all_geology_num.json
```

replace all_geology with all_geology_num
```
cp all_geology_num.json all_geology.json
```

Clip all bedding outside of all_geology_num

```
mapshaper bedding.json -clip all_geology.json -o bedding_clip.json
[o] Wrote bedding_clip.json
```

Create json object to be read to generate legend in web map.

```
mapshaper all_geology.json -dissolve G_ROCK_TYP copy-fields=color -o rock_type_legend.json
```

modify the addClass highlight to also perform an underline and overline.

https://api.jquery.com/addClass/


I spent a lot of time debugging an issue with rock types that have have empty spaces in the name. The div ID seemed to not be readable by jquery,css, or the leaflet process when there were rock types with multiple words.

So I implemented a simple string method chain to remove spaces and replace them wish dashes when appropriate. Basically, everywhere other than the legend text.

**Conclusion and Notes**

This turned out to be a difficult project. I think one of the main reasons was because I didn't really have a specific purpose or end goal in mind when I started processing data. Mapping native American land was intriguing but I did not plan out a strategy for a compelling final map. I ended up not using the native territories dataset because I didn't find any relevant companion datasets to go with it. So in the future I will outline out ALL of the datasets I think will be needed for the final product before going so deep into manipulating a single one in isolation.

However during that process of manipulating the native territories I went very deep into the mapshaper library, using a huge swath of the available commands within the CMD interface. I feel like I have a good grasp of mapshaper now and understand its power but also some of it's limitations. I was not able to do complex calculations on attribute tables, when I wanted to use the -calc flag with spatial joins it wasn't able to carry over all the attributes I needed in the way I needed them. Perhaps there is some advance javascript code that could be piped into mapshaper but I wasn't able to get to that. I ended up having to do some work arounds and then rely on the javascript code *from the web map* rather than doing it within mapshaper. This is not a problem, but took me a while to realize that would be necessary.

I also ended up using quite a bit of example code from lesson 2. Given the time I was taking for the data prep portion I felt I would stick to that code and modify as I go. Since my datasets were very different than the example code I ended up having to debug and rework a lot of what was there. Overall more outlining and planning next time should help a lot. I should clearly define and segment the stages of map building:
1. The ideation; coming up with an interesting idea for a map, and what the final map product should generally look like
2. Thinking through and acquiring all of the datasets necessary (before heavy data cleansing)
3. Cleansing/Manipulating the data with forethought on the tools to be used
4. Thinking through the libraries/modules for writing the actual web map code.
