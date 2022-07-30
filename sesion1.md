---
layout: page
title: Sesión 1
---


**Importancia de las proyecciones:**  

- [Sistema de Referencia de Coordenadas](https://sumapa.com/crsxpais.cfm): Coordenadas que sitúan un punto en la esfera de la Tierra.  

- Diferencia entre SRC proyectados y no proyectados

<img src="imagenes/proyecciones.png" width="600">

- Unidades (metros, pies (ft), para estimar áreas o distancias) *vs* Latitud y longitud (WGS84 (EPSG: 4326))

Por si les interesa leer más sobre [cómo elegir proyecciones](https://source.opennews.org/articles/choosing-right-map-projection/) en sus proyectos.  

\
\
\
**Tipos de datos espaciales:**  

- Vectores (puntos, líneas, polígonos) (por ejemplo: shapefiles, geojson, json, kml, geopackage)  
- Ráster (definen el espacio como un conjunto de celdas del mismo tamaño. Cada celda contine un valor) (por ejemplo: geotiff, netCDF, DEM)

<img src="imagenes/vector_raster.png" width="400">

\
\
\


**¿Qué es el análisis espacial?**  
- Datos geospaciales: ubicación + valores (atributos).  
- Análisis espacial: Cuando cambia la ubicación, el contenido de los datos cambia.  
- Análisis no-espacial: la ubicacion NO importa (*locational invariance*)  

\
\
\

### Comparando análisis espacial *vs* no espacial:  

#### Distribución espacial  

<img src="imagenes/mapa_pov.png" width="600">

\
\
\
\

#### Distribución no espacial  

<img src="imagenes/hist_pov.png" width="600">

https://github.com/ifarah/t/blob/main/assets/img/head.png
\
\
\
\
\

#### Autocorrelación espacial (Índice de Moran)  

<img src="imagenes/moran_pov.png" width="600"> 

\
\
\
\

**¿Qué es la econometría espacial?**  
Tratamiento explícito de la ubicación:  
1. Especificación del modelo  
2. Estimación del modelo  
3. Diagnóstico del modelo  
4. Predicción del modelo  

\
\
\


**Críticas de la econometría espacial:** (Goodchild & Longley 2021)  

1. Escala de nuestros datos (citando a Openshaw 1981)  
  ¿Cuál es la escala apropiada para estudiar el fenómeno que nos interesa?
  El problema de unidad de área modificable (MAUP): Definir unidad de análisis. Problema de escala y agregación. Variación de los resultados obtenidos en relación con el número de zonas en que se divide el total de la zona de estudio. Referencia a las diferencias que se producen cuando la información se agrega a una escala distinta.
  
2. Falacia ecológica (interpretación)  
  Estimar en una escala agregada y concluir explicando el fenómenos a nivel individual. Por ejemplo, municipios con niveles altos de criminalidad no explican comportamiento criminal a nivel individual.
  
  Tener información de estadísticas de salud a nivel punto y luego tener información a nivel electoral y después datos agegados a nivel estado. Solucionar agregando los datos a una sola unidad geográfica o imputando datos mediante interpolación.


\
\
\

**Efectos espaciales:**  


- *Heterogeneidad espacial*  
  Características intrísecas distribuídas de forma desigual en el espacio.   
  Diferencias estructurales en los datos (cambian los coeficientes según la ubicación - structural breaks). 
  Por ejemplo, diferencias entre el norte y el sur del país.

- *Dependencia espacial*  
  Interacción entre los vecinos.  
  Soy el vecino de mi vecino. Estimar la interacción entre las ubicaciones de forma simultánea.
  Se puede estimar mediante la variable espacialmente rezagada, *spatial lag* ($[Wy]_i$) que es el promedio de los valores de los vecinos de una observación.).
  
$$[Wy]_i =  w_{i,1}y_1 + w_{i,2}y_2 + w_{i,3}y_3 + w_{i,n}y_n$$ o

$$[Wy]_i = \sum_{j=1}^{n} w_{i,j}y_j$$

El problema con cuantificar los efectos espaciales es que es imposible saber si los valores vienen de interacción (dependencia) o de características intrínsecas (heterogeneidad).

\
\
\

**Autocorrelación Espacial**
Hipótesis nula: Aleatoriedad espacial is absence of any pattern
Queremos rechazar la hipótesis nula.

La base de todo:  
*Primera Ley de Geografía de Tobler:* Todo depende de todo lo demás, pero ubicaciones cercanas más (distancia de decaimiento).

Estadística de autocorrelación espacial (combina similitud atributos y geográfica) (univariada a diferencia de correlación de Pearson).

- Autocorrelación positiva (e.g., *clustering*).
- Autocorrelación negativa (más difícil de identificar que la autocorrelación positiva por que no es claro cómo difiere de la aleatoriedad espacial, e.g., tablero de damas).

Como el cerebro nos engaña, tenemos que pensar como formalizar las estructuras espaciales.
Esto se logra imponiendo una estructura de pesos espaciales (*Spatial Weights*), $w_{i,j}$

\
\
\

### Tipos de pesos más comúnes:  
**1. Contigüidad (polígono)**  
- Queen (reina: vértices y aristas)  
- Rook (torre: aristas)  
- Bishop (alfil: vértices)  
- Pesos de distinto orden

<img src="imagenes/pesos.png" width="400">  

**2. Distancia (puntos)**  
- KNN (k vecinos más cercanos)
- Distancia

### Ejemplos de vecindad reina  

<img src="imagenes/connected.png" width="600">  

<img src="imagenes/matrix.png" width="600">

Es importante que la matriz tenga muchos ceros (matriz dispersa).
*A priori* estamos definiendo la interacción (puede ser un problema, por lo cual debe estar muy bien fundmentado teóricamente).  

Se estandariza por filas para reescalar los pesos y que todas las filas sumen 1.
Esto ayuda cuando estemos estimando la variable espacialmente rezagada haciendo un promedio de los vecinos y que los análisis sean comparables.

La distribución del número de vecinos debe tener una distribución uni-modal. Si no es así, pensar en dividir los datos espacialmente en regímenes espacials.

Pensar cómo incorporar islas.  

También se puede generar pesos basados en redes sociales, no tiene que estar incorporada una noción espacial necesariamente.

\
\
\


```{r En caso de no tener las bibliotecas, quitar comentario e instalar}
#install.packages("sf")
#install.packages("mapview")
#install.packages("ggplot2")
#install.packages("tmap")
#install.packages("spdep")
#install.packages("rgeos") 
#install.packages("rgdal")
```

```{r setup, include=FALSE}
# Para exploración de datos espaciales y modelos espaciales.
library(spdep) # Spatial Dependence: Weighting Schemes, Statistics and Models (e.g., weights, lm.morantest)
library(rgeos) # Interface to Geometry Engine - Open Source ('GEOS')
library(rgdal) # Bindings for the 'Geospatial' Data Abstraction Library (e.g., readOGR)
```

## Exploración de datos
Primero usar la libreria de sf para mapear y explorar nuestros datos
```{r Usar sf}
#Importar datos espaciales con sf
library(sf) # Simple features (leer spatial data) Primordialmente para usar datos vectoriales 
municipios = st_read("./data/mx_city_metro.shp")

```
Pueden usar `attach(municipios)` para referirse a las variables por nombre, pero no lo hare en este caso.

* **Obligatorios**: 
  - `shp`: Shapefile: archivo principal que guarda la geometría

  - `shx`: Indice posicional para ubicar la geometría del shp

  - `dbf`: La tabla (en formato dBase IV) que guarda la información (atributos)

* **Opcionales**

  - `prj`: Guarda infomación del Sistema de Referencia de Coordenadas (crs, en inglés) (**debería ser obligatorio!**).
  
  - `sbn` y `sbx`: índices espaciales para agilizar operaciones geométricas

  - `xml`: Metadatos — Guarda información del shp
  
  - `cpg`: Especifica el código de la página para identificar la codificación de cada caracter.
  
Formato puede ser punto, línea, polígono.
  
## Datos provienen de:  

- [CONEVAL](https://www.coneval.org.mx/Medicion/Paginas/PobrezaInicio.aspx)  
- Marco Geoestadístico 2020 del [INEGI](https://www.inegi.org.mx/temas/mg/#Descargas)


| Nombre | Tipo de variable | Descripción |
| ----------- | ----------- | ----------- | 
| ID | integer | ID = estado + municpality código |
| ID_TEXT | string | ID (texto) = estado + municpality código |
| state | string | estado código |
| state_name | string | Nombre of estado |
| mun | string | municipio código |
| mun_name | string | Nombre of municipio |
| pop_10 | integer | Population in 2010 |
| pop_15 | integer | Population in 2015 |
| pop_20 | integer | Population in 2020 |
| ppov_10 | float | Porcentaje de la población en pobreza en 2010 |
| ppov_15 | float | Porcentaje de la población en pobreza en 2015 |
| ppov_20 | float | Porcentaje de la población en pobreza en 2020 |
| pepov_10 | float | Porcentaje de la población en pobreza extrema en 2010 |
| pepov_15 | float | Porcentaje de la población en pobreza extrema en 2015 |
| pepov_20 | float | Porcentaje de la población en pobreza extrema en 2020 |
| peduc_10 | float | Porcentaje de la población con rezago educativo en 2010 |
| peduc_15 | float | Porcentaje de la población con rezago educativo en 2015 |
| peduc_20 | float | Porcentaje de la población con rezago educativo en 2020 |
| pheal_10 | float | Porcentaje de la población con falta de acceso a servicios de salud en 2010 |
| pheal_15 | float | Porcentaje de la población con falta de acceso a servicios de salud en 2015 |
| pheal_20 | float | Porcentaje de la población con falta de acceso a servicios de salud en 2020 |
| pss_10 | float | Porcentaje de la población con falta de acceso a seguridad social en 2010 |
| pss_15 | float | Porcentaje de la población con falta de acceso a seguridad social en 2015 |
| pss_20 | float | Porcentaje de la población con falta de acceso a seguridad social en 2020 |
| pqdwel_10 | float | Porcentaje de la población con carencia de calidad a la vivienda en 2010 (calidad de pisos, techos, paredes y hacinamiento) |
| pqdwel_15 | float | Porcentaje de la población con carencia de calidad a la vivienda en 2015 (calidad de pisos, techos, paredes y hacinamiento) |
| pqdwel_20 | float | Porcentaje de la población con carencia de calidad a la vivienda en 2020 (calidad de pisos, techos, paredes y hacinamiento) |
| pserv_10 | float | Porcentaje de la población sin acceso a servicios básicos de la vivienda en 2010 (falta de acceso a agua potable, drenaje, electricidad, combustible para cocinar) |
| pserv_15 | float | Porcentaje de la población sin acceso a servicios básicos de la vivienda en 2015 (falta de acceso a agua potable, drenaje, electricidad, combustible para cocinar) |
| pserv_20 | float | Porcentaje de la población sin acceso a servicios básicos de la vivienda en 2020 (falta de acceso a agua potable, drenaje, electricidad, combustible para cocinar) |
| pfood_10 | float | Porcentaje de la población viviendo con inseguridad alimentaria en 2010  |
| pfood_15 | float | Porcentaje de la población viviendo con inseguridad alimentaria en 2015 |
| pfood_20 | float | Porcentaje de la población viviendo con inseguridad alimentaria en 2020 |
| ping1_10 | float | Porcentaje de la población por debajo de la línea de pobreza en 2010  |
| ping1_15 | float | Porcentaje de la población por debajo de la línea de pobreza en 2015 |
| ping1_20 | float | Porcentaje de la población por debajo de la línea de pobreza en 2020 |
| ping2_10 | float | Porcentaje de la población por debajo de la línea mínima de pobreza en 2010  |
| ping2_15 | float | Porcentaje de la población por debajo de la línea mínima de pobreza en 2015 |
| ping2_20 | float | Porcentaje de la población por debajo de la línea mínima de pobreza en 2020 |

**Ver proyecciones de los datos:**  

```{r Ver proyeccion}
st_crs(municipios)
```

```{r Transformar proyeccion}
municipios_4326 = st_transform(municipios, 4326)
st_crs(municipios_4326)

```

### Mapas coropléticos

```{r Mapear datos - ggplot}
library(ggplot2)

ggplot(municipios_4326, aes(fill = ppov_20)) + 
  geom_sf() +  # tells ggplot that geographic data are being plotted
  scale_fill_viridis_c() +
  theme_minimal() + 
  labs(title = "Porcentaje de pobreza, 2020")

```

```{r Mapear datos - mapview}
library(mapview) # bueno para explorar pero dificil ajustar
# Mapa interactivo
mapview(municipios_4326, zcol = 'ppov_20')

# Agregar los dos
mapview(municipios_4326, zcol = 'ppov_20') +
  mapview(municipios, color = 'red', alpha.regions = 0, legend=FALSE)
```



### Guía de código de colores en R
<img src="imagenes/colores.png" width="500"> 


```{r Mapear datos - tmap}
library(tmap) # Mapas estaticos e interactivos

#tmap_mode('plot') # estaticos
tmap_mode('view') # interactivo

tm_shape(municipios_4326) + 
  tm_polygons(col = 'tan', 
              border.col = 'darkgreen', 
              alpha = 0.5) # transparencia

```

### Examinar nuestros datos

```{r Mapas de otras variables, warning=FALSE, error=FALSE, message=FALSE}
tmap_mode('plot') # estaticos

pe = tm_shape(municipios_4326) + 
  tm_polygons(col = 'pepov_20', 
              border.col = 'darkgreen', 
              alpha = 0.5) + # transparencia
    tm_layout("Pobreza extrema, 2020",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)

ping1 = tm_shape(municipios_4326) + 
  tm_polygons(col = 'ping1_20', 
              border.col = 'darkgreen', 
              alpha = 0.5) + # transparencia
    tm_layout("Por debajo línea ingresos, 2020",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)

educ = tm_shape(municipios_4326) + 
  tm_polygons(col = 'peduc_20', 
              border.col = 'darkgreen', 
              alpha = 0.5) + # transparencia
    tm_layout("Educación, 2020",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)
  
salud = tm_shape(municipios_4326) + 
  tm_polygons(col = 'pheal_20', 
              border.col = 'darkgreen', 
              alpha = 0.5) + # transparencia
    tm_layout("Salud, 2020",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)

sbv = tm_shape(municipios_4326) + 
  tm_polygons(col = 'pqdwel_20', 
              border.col = 'darkgreen', 
              alpha = 0.5) + # transparencia
    tm_layout("Servicios básicos de la vivienda, 2020",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)

alim = tm_shape(municipios_4326) + 
  tm_polygons(col = 'pfood_20', 
              border.col = 'darkgreen', 
              alpha = 0.5) + # transparencia
    tm_layout("Inseguridad Alimentaria, 2020",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)

tmap_arrange(pe, ping1, educ, salud, sbv, alim, ncol=3, nrow=2)
```


# Importar datos para análisis espacial
```{r importar datos}
#Importar datos espaciales 
muni <- readOGR(dsn = "./data/", layer = "mx_city_metro")

muni <- readOGR("data/mx_city_metro.shp")


# Usando rgdal
head(muni)


```
```{r}
getwd()
setwd("../")
class(municipios)
#muni@data$pop20

class(muni)
attach(municipios)

municipios$ID
```

# Pesos espaciales en R

* listw usado en la mayoría de los comandos de `spdep`.
* `poly2nb(object)` convierte polígonos a nb.
* `nb2listw(object)` convierte nb a listw.

```{r pesos, error=FALSE, warning=FALSE, message=FALSE} 
#Importar de archivo existente (abrir archivo)
#Pasar de "neighbor lists" a "weight lists" (listw object)
queen.nb_existente = read.gal("data/mx_city_metro_q1.gal", region.id=muni$ID,override.id=TRUE) #spdep

summary(queen.nb_existente)
q1_existente <- nb2listw(queen.nb_existente)

class(queen.nb_existente) # Identificar el tipo de objeto
class(q1_existente) # Identificar el tipo de objeto

k4.nb_existente= read.gwt2nb("data/mx_city_metro_k4.gwt",region.id=muni$ID)
summary(k4.nb_existente)
```

```{r crear pesos reina} 
# queen weights:
queen.nb <- poly2nb(muni, row.names = muni$ID)
q1 <- nb2listw(queen.nb)
summary(queen.nb)
summary(q1)
```
nn = n*n
S0 =  suma global de los pesos

```{r crear pesos torre} 
# rook weights:
rook.nb <- poly2nb(muni, row.names = muni$ID, queen=FALSE)
summary(rook.nb)
```

```{r Mapear pesos} 
plot(muni)
plot(queen.nb, coordinates(muni),
     pch = 19,
     cex = 0.6,
     add = TRUE,
     col = "red")

plot(rook.nb, coordinates(muni), pch = 19, cex = 0.6, add = TRUE, col = "blue")
```


```{r centroides}
centroides = gCentroid(muni,byid=TRUE)
plot(muni)
points(centroides, col="red", pch=20)
```

```{r knn}
nc.5nn<-knearneigh(centroides, k=5, longlat=TRUE)
nc.5nn.nb<-knn2nb(nc.5nn)
summary(nc.5nn.nb)
knn5 <- nb2listw(nc.5nn.nb)
```

```{r plot knn}
xy <- coordinates(muni)
plot(muni)
plot(nc.5nn.nb, xy, pch=20, col="blue", add=T)
```

## Autocorrelación espacial en R  

La dependencia espacial puede expresarse mediante autocorrelación espacial. La autocorrelación espacial revela valores similares en ubicaciones cercanas; se puede estudiar global o localmente. Concretamente, la autocorrelación espacial revelará si existe aleatoriedad espacial en la distribución de la inseguridad alimentaria entre municipios.

Para estudiar distintos patrones espaciales, la definición de pesos espaciales es crucial. La estructura de pesos espaciales determina la conectividad entre ubicaciones vecinas. 

### Estadísticas de autocorrelación espacial global  

Ejemplos: Indice de Moran, Geary C, Getis-Ord, entre otras (mezclar similitud de atributos con pesos espaciales).
$$ \sum_{ij}= w_{ij}f(x_ix_j) $$ 
**Variable espacialmente rezagada**
```{r spatial lag}
#Examinar si hay autocorrelación espacial con variable de interés
muni$Vx <- lag.listw(q1, muni$pqdwel_20)
plot(muni$pqdwel_20, muni$Vx)
```


**Moran's I (la estadística más usada)**:
Los mapas muestran patrones de distribución espacial, dando indicios de presencia de autocorrelación espacial.
Para formalmente evaluar la presencia de autocorrelación espacial global, utilizamos el índice univariado global de Moran.

$$ I= [\sum_i\sum_j w_{ij} z_iz_j/S_0]/[\sum_iz_i^2/N] $$ 
donde $S_0=\sum_i \sum_j w_{ij}$ y $z_i = y_i-m_x$ es la desviación del promedio.  

Es una estadístics de producto cruzado $(z_i z_j)$ similar a un coeficiente de correlación. En este caso, los valores dependen de una estructura de pesos $(w_{ij})$

*Hipótesis nula: La distribución de los datos es aleatoria en el espacio.*

Consecuentemente, cuando la hipótesis nula es rechazada, podemos decir que hay diferencias espaciales en nuestros datos.

La autocorrelación positiva indica clustering, la autocorrelación negativa indica dispersión espaial. 

Hacer estudios con distintos tipos de pesos para evaluar qué tanto se sostienen nuestras pruebas.

Estadística que puede rechazar la hipótesis nula por encontrar autocorrelación, no-normalidad. No sabemos bien qué es lo que está mal con nuestros datos.


```{r}
set.seed(123456) # Fijar la permutación
moran.test(muni$pqdwel_20, q1, alternative="two.sided")
```


```{r Prueba Moran}
moranTest <- moran.test(muni$pqdwel_20, q1)
moranTest

moranTest$statistic
moranTest$p.value
```


### Estadísticas de autocorrelación espacial local

Los clústeres se pueden identificar a través de indicadores locales de asociación espacial (LISA, Anselin 1995). Estos indicadores muestran las desviaciones de los patrones globales generales al mostrar la contribución de cada observación a la autocorrelación global general, lo que permite una visualización de los puntos calientes/fríos de la variable de interés Como se mencionó anteriormente, diferentes ponderaciones estudian la presencia de patrones mediante el uso de distintas técnicas de conglomerados, como el I de Moran local univariado.

En esta sección vamos a identificar valores atípicos (outliers) mediante las estadísticas de autocorrelación espacial local, identificando hot spots y cold spots.

Los ejemplos anteriores de estadísticas de autocorrelación espacial global están diseñados para encontrar si los datos (en su totalidad) tienen patrones de aleatoriedad espacial. Sin embargo, no muestran la **ubicación** de los clústers.


$$ \sum_{j}=w_{ij} f(x_ix_j)$$ 

```{r Moran Scatter}
#Explicar cuadrantes
moran.plot(muni$pqdwel_20, q1)
```


```{r Local Moran}
local <- localmoran(muni$pqdwel_20, listw = q1)
```

```{r Local Moran mapa, warning=FALSE, error=FALSE, message=FALSE}
#Mapear la estadistica local de Moran (Ii)
moran.mapa <- cbind(muni, local)
tmap_mode("view")
tm_shape(moran.mapa) +
  tm_borders(alpha=0.7)+
  tm_fill(col = "Ii",
          alpha = 0.5,
          style = "quantile",
          title = "Estadistica Local de Moran I") 
```

Vecinos incluidos en el cluster.

``` {r LISAs 1}
#Examinemos aglomeraciones
quadrant <- vector(mode="numeric",length=nrow(local))

# Centra la variable de interés alrededor de promedio
m.pqdwel_20 <- muni$pqdwel_20 - mean(muni$pqdwel_20)     

# Centra estadística local de Moran's alrededor de promedio
m.local <- local[,1] - mean(local[,1])    

# Umbral de significancia
signif_1 <- 0.1 
signif_05 <- 0.05
signif_001 <- 0.001

# Construir cuadrantes para 0.1
quadrant[m.pqdwel_20 >0 & m.local>0] <- 4  
quadrant[m.pqdwel_20 <0 & m.local<0] <- 1      
quadrant[m.pqdwel_20 <0 & m.local>0] <- 2
quadrant[m.pqdwel_20 >0 & m.local<0] <- 3
quadrant[local[,5]>signif_1] <- 0   

# Armar mapa para 0.1
brks <- c(0,1,2,3,4)
colors <- c("white","blue",rgb(0,0,1,alpha=0.4),rgb(1,0,0,alpha=0.4),"red")
plot(muni,border="lightgray",col=colors[findInterval(quadrant,brks,all.inside=FALSE)])
box() #Agregar caja
legend("bottomleft", legend = c("No significativo","Bajo-bajo","Bajo-alto","Alto-bajo","Alto-alto"),
       title="p-value = 0.1", fill=colors,bty="n")

```

```{r LISAs 2, warning=FALSE, error=FALSE, message=FALSE}
# Forma alternativa
muni$LISA=quadrant
colores <- c("#ffffff", "#2c7bb6", "#abd9e9", "#fdae61", "#d7191c")
clusteres <- c("No significativo","Bajo-bajo","Bajo-alto","Alto-bajo","Alto-alto")
tmap_mode("plot")
p1 = tm_shape(muni) +
  tm_fill(col = "LISA", style = "cat", palette = colores[c(sort(unique(quadrant)))+1],
          labels = clusteres[c(sort(unique(quadrant)))+1], popup.vars = c("ID")) +
  tm_borders(alpha=0.5)+
  tm_layout("LISA pvalue = 0.1",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)
  

# Construir cuadrantes para 0.05
quadrant <- vector(mode="numeric",length=nrow(local))
quadrant[m.pqdwel_20 >0 & m.local>0] <- 4  
quadrant[m.pqdwel_20 <0 & m.local<0] <- 1      
quadrant[m.pqdwel_20 <0 & m.local>0] <- 2
quadrant[m.pqdwel_20 >0 & m.local<0] <- 3
quadrant[local[,5]>signif_05] <- 0  

muni$LISA=quadrant
p2 = tm_shape(muni) +
  tm_fill(col = "LISA", style = "cat", palette = colores[c(sort(unique(quadrant)))+1],
          labels = clusteres[c(sort(unique(quadrant)))+1], popup.vars = c("ID")) +
  tm_borders(alpha=0.5)+
  tm_layout("LISA pvalue = 0.05",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)

# Construir cuadrantes para 0.001
quadrant <- vector(mode="numeric",length=nrow(local))
quadrant[m.pqdwel_20 >0 & m.local>0] <- 4  
quadrant[m.pqdwel_20 <0 & m.local<0] <- 1      
quadrant[m.pqdwel_20 <0 & m.local>0] <- 2
quadrant[m.pqdwel_20 >0 & m.local<0] <- 3
quadrant[local[,5]>signif_001] <- 0  

muni$LISA=quadrant
p3 = tm_shape(muni) +
  tm_fill(col = "LISA", style = "cat", palette = colores[c(sort(unique(quadrant)))+1],
          labels = clusteres[c(sort(unique(quadrant)))+1], popup.vars = c("ID")) +
  tm_borders(alpha=0.5)+
  tm_layout("LISA pvalue = 0.001",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)

tmap_arrange(p1, p2, p3)
```

## Con KNN = 5  

Al considerar a los vecinos como todos los municipios abarcados dentro de una distancia determinada, los municipios más grandes tendrán menor conectividad y menos vecinos que los municipios más pequeños.

```{r LISAs 2 knn4, warning=FALSE, error=FALSE, message=FALSE}
# con KNN = 5
local <- localmoran(muni$pqdwel_20, listw = knn5)

#Examinemos aglomeraciones
quadrant <- vector(mode="numeric",length=nrow(local))

# Centra la variable de interés alrededor de promedio
m.pqdwel_20 <- muni$pqdwel_20 - mean(muni$pqdwel_20)     

# Centra estadística local de Moran's alrededor de promedio
m.local <- local[,1] - mean(local[,1])    

# Umbral de significancia
signif_1 <- 0.1 
signif_05 <- 0.05
signif_001 <- 0.001

# Construir cuadrantes para 0.1
quadrant[m.pqdwel_20 >0 & m.local>0] <- 4  
quadrant[m.pqdwel_20 <0 & m.local<0] <- 1      
quadrant[m.pqdwel_20 <0 & m.local>0] <- 2
quadrant[m.pqdwel_20 >0 & m.local<0] <- 3
quadrant[local[,5]>signif_1] <- 0 

muni$LISA_kn=quadrant
colores <- c("#ffffff", "#2c7bb6", "#abd9e9", "#fdae61", "#d7191c")
clusteres <- c("No significativo","Bajo-bajo","Bajo-alto","Alto-bajo","Alto-alto")
tmap_mode("plot")
p1 = tm_shape(muni) +
  tm_fill(col = "LISA_kn", style = "cat", palette = colores[c(sort(unique(quadrant)))+1],
          labels = clusteres[c(sort(unique(quadrant)))+1], popup.vars = c("ID")) +
  tm_borders(alpha=0.5)+
  tm_layout("LISA pvalue = 0.1",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)
  

# Construir cuadrantes para 0.05
quadrant <- vector(mode="numeric",length=nrow(local))
quadrant[m.pqdwel_20 >0 & m.local>0] <- 4  
quadrant[m.pqdwel_20 <0 & m.local<0] <- 1      
quadrant[m.pqdwel_20 <0 & m.local>0] <- 2
quadrant[m.pqdwel_20 >0 & m.local<0] <- 3
quadrant[local[,5]>signif_05] <- 0  

muni$LISA_kn=quadrant
p2 = tm_shape(muni) +
  tm_fill(col = "LISA_kn", style = "cat", palette = colores[c(sort(unique(quadrant)))+1],
          labels = clusteres[c(sort(unique(quadrant)))+1], popup.vars = c("ID")) +
  tm_borders(alpha=0.5)+
  tm_layout("LISA pvalue = 0.05",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)

# Construir cuadrantes para 0.001
quadrant <- vector(mode="numeric",length=nrow(local))
quadrant[m.pqdwel_20 >0 & m.local>0] <- 4  
quadrant[m.pqdwel_20 <0 & m.local<0] <- 1      
quadrant[m.pqdwel_20 <0 & m.local>0] <- 2
quadrant[m.pqdwel_20 >0 & m.local<0] <- 3
quadrant[local[,5]>signif_001] <- 0  

muni$LISA_kn=quadrant
p3 = tm_shape(muni) +
  tm_fill(col = "LISA_kn", style = "cat", palette = colores[c(sort(unique(quadrant)))+1],
          labels = clusteres[c(sort(unique(quadrant)))+1], popup.vars = c("ID")) +
  tm_borders(alpha=0.5)+
  tm_layout("LISA pvalue = 0.001",
            title.size = 1,
            legend.title.size = 1,
            legend.text.size = 0.6,
            legend.position = c("left","bottom"),
            legend.bg.color = "white",
            legend.bg.alpha = 1)

tmap_arrange(p1, p2, p3)
```

### Heterogeneidad espacial en R

Se espera heterogeneidad espacial en México, donde algunas regiones son más heterogéneas que otras debido a las diferencias geográficas y económicas.

Imponer estructura de forma discreta (regímenes) o continua (smoothing, GWR - regresión geográficamente ponderada).

```{r dummy}
#Crear una variable dicotómica que sea 1 si el municipio es parte de la Ciudad de México y 0 si no.
muni$cdmx= ifelse(muni$state=="09", 1, 0)
municipios_4326$cdmx= ifelse(municipios$state=="09", 1, 0)

#municipios_4326%>%group_by(cdmx)%>%summarise(n=n())
```

```{r regimenes}
ggplot(municipios_4326, aes(fill = cdmx)) + 
  geom_sf() +  # tells ggplot that geographic data are being plotted
  theme_minimal() + 
  labs(title = "Régimen espacial")
```

No sabemos por qué hay diferencias, pero resolvemos el problema de heterogeneidad.

```{r dummy densidad poblacional}
#Crear una variable dicotómica que sea 1 si el municipio es parte de la Ciudad de México y 0 si no.
municipios_4326$dens=as.numeric(municipios_4326$pop_20/st_area(municipios_4326)*1000)

mean(municipios_4326$dens)

municipios_4326$pop_dens= ifelse(municipios_4326$dens>4.222332, 1, 0)

#municipios_4326%>%group_by(pop)%>%summarise(n=n())
```

```{r regimenes - densidad poblacional}
ggplot(municipios_4326, aes(fill = pop_dens)) + 
  geom_sf() +  # tells ggplot that geographic data are being plotted
  theme_minimal() + 
  labs(title = "Régimen espacial - densidad poblacional")
```

### Bibliografía
Anselin, L. (1995). Local indicators of spatial association—LISA. Geographical analysis, 27(2), 93-115.

Goodchild, M. F.; Longley, P. A. (2021). Geographic Information Science. En Fischer, M. M. and Nijkamp, P., editores, Handbook of Regional Science. Springer-Verlag Berlin Heidelberg.

Openshaw, S. (1981). The modifiable areal unit problem. Quantitative geography: A British view, 60-69.

