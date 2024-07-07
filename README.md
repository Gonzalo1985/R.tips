---
title: README
subtitle: R.tips
tags: [R,leaflet,Jekyll, html, maps]
leafletmap: true
always_allow_html: yes
last_modified_at: 2024-07-06
output: 
  md_document:
    variant: gfm
    preserve_yaml: true
---

## Convertir un archivo netcdf a geojson

Las librerías previamente instaladas deben ser:

``` r
library("terra")
library("geojsonio")
```

Abrimos varios archivos netcdf y lo transformamos a proyección lat/lon y
transformamos a polígono:

``` r
nc <- rast(c("./data/WRFDETAR_01H_20240101_12_000.nc",
             "./data/WRFDETAR_01H_20240101_12_001.nc",
             "./data/WRFDETAR_01H_20240101_12_002.nc",
             "./data/WRFDETAR_01H_20240101_12_003.nc"))

T2.positions <- which(names(nc) == "T2")

T2.nc <- nc[[T2.positions]]

T2.nc.repro <- project(T2.nc, "+proj=longlat +datum=WGS84")

T2.polygons <- lapply(T2.nc, as.polygons)
```

Finalmente, se escribe un archivo shp y se utiliza la función
file_to_geojson para crear el archivo geojson para cada tiempo:

``` r
for (i in 1:4){
  writeVector(T2.polygons[[i]], paste0("temps_", i, ".shp"), overwrite = TRUE)

  s <- file_to_geojson(paste0("temps_", i, ".shp"), method = "local",
                       output = paste0("temps_", i))
}
```

    ## Success! File is at temps_1.geojson

    ## Success! File is at temps_2.geojson

    ## Success! File is at temps_3.geojson

    ## Success! File is at temps_4.geojson

## Visualizar geojson en mapa Leaflet

Las librerías previamente instaladas deben ser:

``` r
library("sf")
library("leaflet")
```

Primero se abre el geojson a graficar:

``` r
T2.geojson <- read_sf("temps_1.geojson") 
```

Luego se grafica la variable del geojson, en este caso temperatura:

``` r
pal <- colorNumeric("viridis", NULL)

mapa <- leaflet(T2.geojson) %>%
  addTiles() %>%
  addPolygons(stroke = FALSE, smoothFactor = 0.8, fillOpacity = 0.7,
              fillColor = ~pal(T2),
              label = ~paste0(T2, ": ", formatC(T2, big.mark = ","))) %>%
  addLegend(pal = pal, values = ~T2, opacity = 1.0)

mapa <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addMarkers(lng=174.768, lat=-36.852, popup="The birthplace of R")

mapa
```

<div class="leaflet html-widget html-fill-item" id="htmlwidget-a68a5fcb9e7c197fa6db" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-a68a5fcb9e7c197fa6db">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"https://openstreetmap.org/copyright/\">OpenStreetMap<\/a>,  <a href=\"https://opendatacommons.org/licenses/odbl/\">ODbL<\/a>"}]},{"method":"addMarkers","args":[-36.852,174.768,null,null,null,{"interactive":true,"draggable":false,"keyboard":true,"title":"","alt":"","zIndexOffset":0,"opacity":1,"riseOnHover":false,"riseOffset":250},"The birthplace of R",null,null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[-36.852,-36.852],"lng":[174.768,174.768]}},"evals":[],"jsHooks":[]}</script>

## Ploting MCD12Q1 MODIS Product

The data is available in data folder. First, open the data with terra
package

``` r
list.pathname <- list.files("./data", pattern = "hdf", full.names = TRUE)

data <- lapply(list.pathname, FUN = rast)
```

Now, creation of mosaic with each hdf.

``` r
data.mosaic <- mosaic(data[[1]], data[[2]], data[[3]], data[[4]], data[[5]], data[[6]], data[[7]], data[[8]], data[[9]])
```

    ## |---------|---------|---------|---------|=========================================                                          

Lastly, ploting of data with no change of projection, the numbers
represent soil type

``` r
plot(data.mosaic$LC_Type1)
```

![](README_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->
