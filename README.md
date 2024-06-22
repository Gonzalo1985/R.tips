## Convertir un archivo netcdf a geojson

Las librerías previamente instaladas deben ser:

``` r
library("terra")
library("geojsonio")
```

Abrimos varios archivos netcdf y lo transformamos a proyección lat/lon y
transformamos a polígono:

``` r
nc <- rast(c("WRFDETAR_01H_20240101_12_000.nc",
             "WRFDETAR_01H_20240101_12_001.nc",
             "WRFDETAR_01H_20240101_12_002.nc",
             "WRFDETAR_01H_20240101_12_003.nc"))

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

mapa
```

![](README_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->
