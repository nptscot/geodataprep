

# Simplified road network data: open_roads_scotland.geojson

``` r
data_dir = "../npt/inputdata/"
simplified_roads = sf::st_read(file.path(data_dir, "open_roads_scotland.geojson")) |>
  sf::st_transform(crs = 27700)
```

    Reading layer `open_roads_scotland' from data source 
      `/home/robin/github/nptscot/npt/inputdata/open_roads_scotland.geojson' 
      using driver `GeoJSON'
    Simple feature collection with 466311 features and 20 fields
    Geometry type: LINESTRING
    Dimension:     XY
    Bounding box:  xmin: 9123 ymin: 530366 xmax: 468964.8 ymax: 1216649
    Projected CRS: OSGB36 / British National Grid

# Scotland region and local authority boundaries

``` r
lads = sf::st_read(file.path(data_dir, "boundaries/la_regions_scotland_bfe_simplified_2023.geojson")) |>
  sf::st_transform(crs = 27700)
```

    Reading layer `la_regions_scotland_bfe_simplified_2023' from data source 
      `/home/robin/github/nptscot/npt/inputdata/boundaries/la_regions_scotland_bfe_simplified_2023.geojson' 
      using driver `GeoJSON'
    Simple feature collection with 32 features and 3 fields
    Geometry type: MULTIPOLYGON
    Dimension:     XY
    Bounding box:  xmin: -8.649422 ymin: 54.63359 xmax: -0.7323456 ymax: 60.84569
    Geodetic CRS:  WGS 84

``` r
# [1] "LAD23CD"  "LAD23NM"  "Region"   "geometry"
```

lads\$Region \[1\] “Tayside, Central and Fife” “Scotland South” \[3\]
“Glasgow and Strathclyde” “Edinburgh and Lothians” \[5\] “Highlands and
Islands” “Aberdeen and North East”

lads\$LAD23NM \[1\] “Clackmannanshire” “Dumfries and Galloway” “East
Ayrshire”  
\[4\] “East Lothian” “East Renfrewshire” “Na h-Eileanan Siar” \[7\]
“Falkirk” “Highland” “Inverclyde” \[10\] “Midlothian” “Moray” “North
Ayrshire” \[13\] “Orkney Islands” “Scottish Borders” “Shetland Islands”
\[16\] “South Ayrshire” “South Lanarkshire” “Stirling” \[19\] “Aberdeen
City” “Aberdeenshire” “Argyll and Bute” \[22\] “City of Edinburgh”
“Renfrewshire” “West Dunbartonshire” \[25\] “West Lothian” “Angus”
“Dundee City” \[28\] “East Dunbartonshire” “Fife” “Perth and Kinross”
\[31\] “Glasgow City” “North Lanarkshire”

# MasterMap data can be downloaded from https://digimap.edina.ac.uk/

Ordnance Survey Tab -\> OS Digimap -\> log in with organisation -\>
Download Data -\> OS MasterMap -\> Select layers (e.g. Topography or
Highways Road)

The folloing code can be used to read the data as save as Rds file.

``` r
#remotes::install_github("acteng/mastermapr")
# directory = "data/MasterMap Highways Network_roads_5779840"
# system.time({
#   mm_data = mastermapr::mm_read(directory, pattern = "RoadLink", n_files = Inf) 
# })
saveRDS(mm_data, "data/mm_data_RoadLink.Rds")
mm_data_2d = sf::st_zm(mm_data)
list_cols = sapply(mm_data_2d, is.list)
mm_data_list = mm_data_2d[, !list_cols]
mm_data_scot = mm_data_list[st_union(lads), , op = st_intersects]
# sf::st_write(mm_data_scot, "data/mm_data_RoadLink_Scotland.gpkg")
# system("gh release create v1")
# system("gh release upload v1 data/mm_data_RoadLink_Scotland.gpkg")
```

``` r
mm_data_scot = sf::read_sf("mm_data_RoadLink_Scotland.gpkg")
```

The MasterMap Roadlink data contains the following columns:

\[1\] “gml_id” \[2\] “identifier” \[3\] “beginLifespanVersion” \[4\]
“localId” \[5\] “namespace” \[6\] “fictitious” \[7\] “validFrom” \[8\]
“reasonForChange” \[9\] “roadClassification” \[10\] “routeHierarchy”
\[11\] “formOfWay” \[12\] “trunkRoad” \[13\] “primaryRoute” \[14\]
“operationalState” \[15\] “provenance” \[16\] “length” \[17\]
“length_uom” \[18\] “matchStatus” \[19\] “startGradeSeparation” \[20\]
“endGradeSeparation” \[21\] “averageWidth” \[22\] “averageWidth_uom”
\[23\] “minimumWidth” \[24\] “minimumWidth_uom” \[25\] “confidenceLevel”
\[26\] “inDirection” \[27\] “inDirection_uom” \[28\]
“inOppositeDirection” \[29\] “inOppositeDirection_uom” \[30\]
“cycleFacility” \[31\] “wholeLink” \[32\] “roadStructure” \[33\]
“roadName” \[34\] “alternateIdentifier\|ThematicIdentifier\|identifier”
\[35\] “identifierScheme” \[36\] “alternateName” \[37\]
“roadClassificationNumber” \[38\] “centrelineGeometry”

# OSM data can be downloaded using osmactive package

``` r
library(osmactive)
osm_national = get_travel_network("Scotland")
```

    The input place was matched with: Scotland

    The chosen file was already detected in the download directory. Skip downloading.

    Starting with the vectortranslate operations on the input file!

    0...10...20...30...40...50...60...70...80...90...100 - done.

    Finished the vectortranslate operations on the input file!

    Reading layer `lines' from data source 
      `/data/bronze/osm/geofabrik_scotland-latest.gpkg' using driver `GPKG'
    Simple feature collection with 1444010 features and 46 fields
    Geometry type: LINESTRING
    Dimension:     XY
    Bounding box:  xmin: -20.62345 ymin: 52.69143 xmax: 9.975589 ymax: 65.36242
    Geodetic CRS:  WGS 84

``` r
osm_national = osm_national |> sf::st_transform(crs = 27700)
```

The OSM data contains the following columns:

\[1\] “osm_id” “name” “highway” \[4\] “man_made” “maxspeed” “oneway”
\[7\] “bicycle” “cycleway” “cycleway_left” \[10\] “cycleway_right”
“cycleway_both” “lanes” \[13\] “lanes_both_ways” “lanes_forward”
“lanes_backward” \[16\] “lanes_bus” “lanes_bus_conditional” “width”
\[19\] “segregated” “sidewalk” “footway” \[22\] “service” “surface”
“tracktype” \[25\] “smoothness” “access” “z_order” \[28\] “other_tags”
“geometry”

# Example of Edinburgh

``` r
library(zonebuilder)
library(mapview)
zone = zonebuilder::zb_zone("Edinburgh", n_circles = 1) |> sf::st_transform(crs = 27700)
```

    Loading required namespace: tmaptools

    Please set segment_center = TRUE to divide the centre into multiple segments

``` r
mm_data_zone = mm_data_scot[sf::st_union(zone), , op = sf::st_intersects]
osm_data_zone = osm_national[sf::st_union(zone), , op = sf::st_intersects]
simplified_roads_zone = simplified_roads[sf::st_union(zone), , op = sf::st_intersects]

mm_data_zone_2d = sf::st_zm(mm_data_zone)
list_cols = sapply(mm_data_zone_2d, is.list)
mm_data_zone_clean = mm_data_zone_2d[, !list_cols]

mapview(simplified_roads_zone, color = "blue")  + mapview(osm_data_zone, color = "green") + mapview(mm_data_zone_clean, color = "red")
```

    file:////tmp/RtmpnUBh94/file13d1fc268457f1/widget13d1fc6fe8c247.html screenshot completed

![](README_files/figure-commonmark/unnamed-chunk-6-1.png)

# Join averageWidth from MasterMap RoadLink data to simplified road network data

``` r
library(stplanr)
library(ggplot2)

rnet_joined = stplanr::rnet_merge(simplified_roads_zone, mm_data_zone_clean, max_angle_diff = 15, dist = 15, funs = list(averageWidth  = sum))
```

    Warning: st_centroid assumes attributes are constant over geometries

    Joining with `by = join_by(id)`

``` r
buffer = sf::st_buffer(rnet_joined, 20)
summary(mm_data_zone_clean$averageWidth)
```

       Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
       2.40    7.20    9.60   10.49   13.10   35.10      35 

``` r
summary(rnet_joined$averageWidth)
```

         Min.   1st Qu.    Median      Mean   3rd Qu.      Max.      NA's 
       0.5965    7.5027   10.6470   14.8333   15.4573 1756.3570        53 

``` r
mapview(rnet_joined) + mapview(mm_data_zone_clean, color = "red") 
```

    file:////tmp/RtmpYc2piw/file13fc4f6b13d827/widget13fc4f2c6d7786.html screenshot completed

![](README_files/figure-commonmark/unnamed-chunk-7-1.png)

``` r
mm_data_zone_clean_f = mm_data_zone_clean |> dplyr::filter(gml_id%in% c("osgb4000000006400630", "osgb4000000006400630", "osgb4000000006400631", "osgb4000000006400631","osgb5000005304693961", "osgb4000000006400658"))
simplified_roads_zone_f = simplified_roads_zone |> dplyr::filter(id %in% c("A1E134C9-85D1-4A94-A4B6-F08E8A863B17"))
mapview(mm_data_zone_clean_f) + mapview(simplified_roads_zone_f)
```

    file:////tmp/RtmpYc2piw/file13fc4f64d91e0a/widget13fc4f3378734c.html screenshot completed

![](README_files/figure-commonmark/unnamed-chunk-7-2.png)

``` r
if (!dir.exists("rnetmatch")) {
  system("gh repo clone rnetmatch")
}
# wd = setwd("rnetmatch/r")
# getwd()
# devtools::build()
# # devtools::install() # fails
# devtools::load_all()
# setwd(wd)
devtools::load_all("rnetmatch/r")

matched_df = rnet_match(simplified_roads_zone, mm_data_zone_clean, dist_tolerance = 20, angle_tolerance = 10, trees = "xy")

sf::st_write(mm_data_zone_clean_f, "two_lines.geojson")
sf::st_write(simplified_roads_zone_f, "one_line.geojson")
readr::write_csv(matched_df, "matched_df.csv")
```

``` r
matched_df = readr::read_csv("matched_df.csv")
```

    Rows: 2411 Columns: 3
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    dbl (3): i, j, shared_len

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
matched_df |>
  head() |>
  knitr::kable()
```

|   i |    j | shared_len |
|----:|-----:|-----------:|
|   1 |   97 |   7.071068 |
|   2 | 1238 |  51.623638 |
|   2 |  377 |   0.000000 |
|   3 | 1382 |   0.000000 |
|   3 |  320 |  46.872167 |
|   4 |  793 |  39.802801 |
