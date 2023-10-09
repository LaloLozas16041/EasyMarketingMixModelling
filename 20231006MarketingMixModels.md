Guía Definitiva para el Marketing Mix Modelling
================

## Introducción

Ya sea que se trate de una empresa establecida o bastante nueva en el
mercado, casi todas las empresas utilizan diferentes canales de
marketing como televisión, radio, correo electrónico, redes sociales,
etc., para llegar a sus clientes potenciales y aumentar el conocimiento
de su producto y, a su vez, maximizar las ventas o ingresos.

Pero con tantos canales de marketing a su disposición, las empresas
deben decidir qué canales de marketing son eficaces en comparación con
otros y, lo que es más importante, cuánto presupuesto se debe asignar a
cada canal. Con el surgimiento del marketing online y varias plataformas
y herramientas de big data, el marketing es una de las áreas de
oportunidades más destacadas para las aplicaciones de ciencia de datos y
aprendizaje automático.

<strong>Objetivos de aprendizaje</strong>
</p>
<ol>
<li>
¿Qué es el Marketing Mix Modelling y por qué MMM con Robyn es mejor que
un MMM tradicional?
</li>
<li>
Componentes de las series temporales: tendencia, estacionalidad,
ciclicidad, ruido, etc.
</li>
<li>
Adstocks publicitarios: efecto de arrastre y efecto de rendimientos
decrecientes, y transformación de Adstock: geométrico, Weibull CDF y
Weibull PDF.
</li>
<li>
¿Qué son la optimización sin gradientes y la optimización de
hiperparámetros multiobjetivo con Nevergrad?
</li>
<li>
Implementación del Marketing Mix Modelling utilizando Robyn.
</li>
</ol>

Entonces, sin más preámbulos, demos el primer paso para comprender cómo
implementar el Marketing Mix Modelling utilizando la biblioteca Robyn
desarrollada por el equipo de Facebook (ahora Meta) y, lo más
importante, cómo interpretar los resultados de salida.

## Estacionalidad

Si observa un ciclo periódico en la serie con frecuencias fijas,
entonces puede decir que hay una estacionalidad en los datos. Estas
frecuencias pueden ser diarias, semanales, mensuales, etc. En palabras
simples, la estacionalidad siempre es de un período fijo y conocido, lo
que significa que notará una cantidad de tiempo definida entre los picos
y los valles de los datos; ergo, a veces, las series de tiempo
estacionales también se denominan series de tiempo periódicas. no Por
ejemplo, las ventas minoristas aumentan en algunos festivales o eventos
en particular, o la temperatura del clima muestra su comportamiento
estacional de días cálidos en verano y días fríos en invierno, etc.

``` r
library(forecast) # Librería clásica de Pronósticos
```

    ## Warning: package 'forecast' was built under R version 4.2.3

    ## Registered S3 method overwritten by 'quantmod':
    ##   method            from
    ##   as.zoo.data.frame zoo

``` r
ggseasonplot(AirPassengers)
```

![](20231006MarketingMixModels_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
library(fpp2) # Librería con la información para ilustrar ***
```

    ## Warning: package 'fpp2' was built under R version 4.2.3

    ## ── Attaching packages ────────────────────────────────────────────── fpp2 2.5 ──

    ## ✔ ggplot2   3.4.2     ✔ expsmooth 2.3  
    ## ✔ fma       2.5

    ## Warning: package 'ggplot2' was built under R version 4.2.3

    ## Warning: package 'fma' was built under R version 4.2.3

    ## Warning: package 'expsmooth' was built under R version 4.2.3

    ## 

``` r
autoplot(lynx) + xlab("Anio") + ylab("Número de linces atrapados")
```

![](20231006MarketingMixModels_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

## Paso 1: Instalar los paquetes adecuados

### Paso 1.a.Primera instalación de los paquetes necesarios, descomentar la primera vez

``` r
#install.packages("Robyn")
#install.packages("reticulate")
library(reticulate)
```

    ## Warning: package 'reticulate' was built under R version 4.2.3

``` r
library(Robyn)
```

    ## Warning: package 'Robyn' was built under R version 4.2.3

### Paso 1.b Configurar el entorno virtual e instalar la biblioteca Nevergrad

``` r
virtualenv_create("r-reticulate")
```

    ## virtualenv: r-reticulate

``` r
py_install("nevergrad", pip = TRUE)
#use_virtualenv("r-reticulate", required = TRUE) #Descomentar esta parte la primera vez
```

### Paso 1.c Importar paquetes y configurar CWD

``` r
library(Robyn) 
library(reticulate)
set.seed(123)

setwd('MMM')
```

``` r
#Paso 1.d Puedes forzar el uso de múltiples núcleos ejecutando la siguiente línea de código
Sys.setenv(R_FUTURE_FORK_ENABLE = "true")
options(future.fork.enable = TRUE)

# Puedes configurar create_files en FALSE para evitar la creación de archivos localmente
create_files <- TRUE
```

## Paso 2: Cargar Datos

``` r
#Paso 2.a Cargar datos
data("dt_simulated_weekly")
head(dt_simulated_weekly)
```

    ## # A tibble: 6 × 12
    ##   DATE        revenue    tv_S  ooh_S print_S facebook_I search_clicks_P search_S
    ##   <date>        <dbl>   <dbl>  <dbl>   <dbl>      <dbl>           <dbl>    <dbl>
    ## 1 2015-11-23 2754372.  67075. 0       38185.  72903853.              0         0
    ## 2 2015-11-30 2584277.  85840. 0           0   16581100.          29512.    12400
    ## 3 2015-12-07 2547387.      0  3.97e5   1362.  49954774.          36132.    11360
    ## 4 2015-12-14 2875220  250351. 0       53040   31649297.          36804.    12760
    ## 5 2015-12-21 2215953.      0  8.32e5      0    8802269.          28402.    10840
    ## 6 2015-12-28 2569922.  99676. 0       95767.  49902081.          38062.    11320
    ## # ℹ 4 more variables: competitor_sales_B <int>, facebook_S <dbl>, events <chr>,
    ## #   newsletter <dbl>

``` r
#Paso 2.b Cargar datos de vacaciones desde Prophet
data("dt_prophet_holidays")
head(dt_prophet_holidays)
```

    ## # A tibble: 6 × 4
    ##   ds         holiday                                               country  year
    ##   <date>     <chr>                                                 <chr>   <int>
    ## 1 1995-01-01 Ano Nuevo [New Year's Day]                            AR       1995
    ## 2 1995-02-27 Dia de Carnaval [Carnival's Day]                      AR       1995
    ## 3 1995-02-28 Dia de Carnaval [Carnival's Day]                      AR       1995
    ## 4 1995-03-24 Dia Nacional de la Memoria por la Verdad y la Justic… AR       1995
    ## 5 1995-04-02 Dia del Veterano y de los Caidos en la Guerra de Mal… AR       1995
    ## 6 1995-04-13 Semana Santa (Jueves Santo)  [Holy day (Holy Thursda… AR       1995

``` r
# Exportar resultados al directorio deseado
robyn_object<- "~/MyRobyn.RDS"
```

## Paso 3: Especificación del Modelo

### Paso 3.1 Definir variables de entrada

Dado que Robyn es una herramienta semiautomática, usar una tabla como la
siguiente puede ser valioso para ayudar a articular variables
independientes y de destino para su modelo:

``` r
InputCollect <- robyn_inputs(
  dt_input = dt_simulated_weekly,
  dt_holidays = dt_prophet_holidays,
  dep_var = "revenue",
  dep_var_type = "revenue",
  date_var = "DATE",
  prophet_country = "DE",
  prophet_vars = c("trend", "season", "holiday"), 
  context_vars = c("competitor_sales_B", "events"),
  paid_media_vars = c("tv_S", "ooh_S", "print_S", "facebook_I", "search_clicks_P"),
  paid_media_spends = c("tv_S", "ooh_S", "print_S", "facebook_S", "search_S"),
  organic_vars = "newsletter", 
  # factor_vars = c("events"),
  adstock = "geometric", 
  window_start = "2016-01-01",
  window_end = "2018-12-31",
  
)
```

    ## Automatically set these variables as 'factor_vars': "events"

    ## Input 'window_start' is adapted to the closest date contained in input data: 2016-01-04

``` r
print(InputCollect)
```

    ## Total Observations: 208 (weeks)
    ## Input Table Columns (12):
    ##   Date: DATE
    ##   Dependent: revenue [revenue]
    ##   Paid Media: tv_S, ooh_S, print_S, facebook_I, search_clicks_P
    ##   Paid Media Spend: tv_S, ooh_S, print_S, facebook_S, search_S
    ##   Context: competitor_sales_B, events
    ##   Organic: newsletter
    ##   Prophet (Auto-generated): trend, season, holiday on DE
    ##   Unused variables: None
    ## 
    ## Date Range: 2015-11-23:2019-11-11
    ## Model Window: 2016-01-04:2018-12-31 (157 weeks)
    ## With Calibration: FALSE
    ## Custom parameters: None
    ## 
    ## Adstock: geometric
    ## Hyper-parameters: [0;31mNot set yet[0m

#### Signo de coeficientes:

Predeterminado: significa que la variable podría tener coeficientes + o
– dependiendo del resultado del modelado. Sin embargo,
Positivo/Negativo: si conoce el impacto específico de una variable de
entrada en la variable objetivo, entonces Puede elegir el signo en
consecuencia. Nota: Todos los controles de signos se proporcionan
automáticamente: “+” para las variables orgánicas y de medios y
“predeterminado” para todas las demás. No obstante, aún puedes
personalizar las señales si es necesario.

## Problema 1

La base de datos `CARS2004` del paquete `PASWR2` recoge el número de
coches por 1000 habitantes (`cars`), el número total de accidentes con
víctimas mortales (`deaths`) y la población/1000 (`population`) para los
25 miembros de la Unión Europea en el año 2004.

1.  Proporciona con `R` resumen de los datos.
2.  Utiliza la función `eda` del paquete `PASWR2` para realizar un
    análisis exploratorio de la variable `deaths`

#### Paso 3.2 Especificar nombres y rangos de hiperparámetros

Los hiperparámetros de Robyn tienen cuatro componentes:

Parámetro de validación de series temporales (train_size). Parámetros de
Adstock (theta o forma/escala). Parámetros de saturación (alfa/gamma).
Parámetro de regularización (lambda). Especificar nombres de
hiperparámetros

``` r
hyper_names(adstock = InputCollect$adstock, all_media = InputCollect$all_media)
```

    ##  [1] "facebook_S_alphas" "facebook_S_gammas" "facebook_S_thetas"
    ##  [4] "newsletter_alphas" "newsletter_gammas" "newsletter_thetas"
    ##  [7] "ooh_S_alphas"      "ooh_S_gammas"      "ooh_S_thetas"     
    ## [10] "print_S_alphas"    "print_S_gammas"    "print_S_thetas"   
    ## [13] "search_S_alphas"   "search_S_gammas"   "search_S_thetas"  
    ## [16] "tv_S_alphas"       "tv_S_gammas"       "tv_S_thetas"

``` r
## Nota: Establezca plot = TRUE para producir gráficos de ejemplo para
#adstock e hiperparámetros de saturación.

plot_adstock(plot = FALSE)
plot_saturation(plot = FALSE)

# Para comprobar los límites máximos inferior y superior
hyper_limits()
```

    ##   thetas alphas gammas shapes scales
    ## 1    >=0     >0     >0    >=0    >=0
    ## 2     <1    <10    <=1    <20    <=1

``` r
# Especificar rangos de hiperparámetros para material publicitario geométrico
hyperparameters <- list(
  facebook_S_alphas = c(0.5, 3),
  facebook_S_gammas = c(0.3, 1),
  facebook_S_thetas = c(0, 0.3),
  print_S_alphas = c(0.5, 3),
  print_S_gammas = c(0.3, 1),
  print_S_thetas = c(0.1, 0.4),
  tv_S_alphas = c(0.5, 3),
  tv_S_gammas = c(0.3, 1),
  tv_S_thetas = c(0.3, 0.8),
  search_S_alphas = c(0.5, 3),
  search_S_gammas = c(0.3, 1),
  search_S_thetas = c(0, 0.3),
  ooh_S_alphas = c(0.5, 3),
  ooh_S_gammas = c(0.3, 1),
  ooh_S_thetas = c(0.1, 0.4),
  newsletter_alphas = c(0.5, 3),
  newsletter_gammas = c(0.3, 1),
  newsletter_thetas = c(0.1, 0.4),
  train_size = c(0.5, 0.8)
)

#Agregar hiperparámetros a robyn_inputs()

InputCollect <- robyn_inputs(InputCollect = InputCollect, hyperparameters = hyperparameters)
```

    ## >> Running feature engineering...

    ## Warning in .font_global(font, quiet = FALSE): Font 'Arial Narrow' is not
    ## installed, has other name, or can't be found

``` r
print(InputCollect)
```

    ## Total Observations: 208 (weeks)
    ## Input Table Columns (12):
    ##   Date: DATE
    ##   Dependent: revenue [revenue]
    ##   Paid Media: tv_S, ooh_S, print_S, facebook_I, search_clicks_P
    ##   Paid Media Spend: tv_S, ooh_S, print_S, facebook_S, search_S
    ##   Context: competitor_sales_B, events
    ##   Organic: newsletter
    ##   Prophet (Auto-generated): trend, season, holiday on DE
    ##   Unused variables: None
    ## 
    ## Date Range: 2015-11-23:2019-11-11
    ## Model Window: 2016-01-04:2018-12-31 (157 weeks)
    ## With Calibration: FALSE
    ## Custom parameters: None
    ## 
    ## Adstock: geometric
    ## Hyper-parameters ranges:
    ##   facebook_S_alphas: [0.5, 3]
    ##   facebook_S_gammas: [0.3, 1]
    ##   facebook_S_thetas: [0, 0.3]
    ##   print_S_alphas: [0.5, 3]
    ##   print_S_gammas: [0.3, 1]
    ##   print_S_thetas: [0.1, 0.4]
    ##   tv_S_alphas: [0.5, 3]
    ##   tv_S_gammas: [0.3, 1]
    ##   tv_S_thetas: [0.3, 0.8]
    ##   search_S_alphas: [0.5, 3]
    ##   search_S_gammas: [0.3, 1]
    ##   search_S_thetas: [0, 0.3]
    ##   ooh_S_alphas: [0.5, 3]
    ##   ooh_S_gammas: [0.3, 1]
    ##   ooh_S_thetas: [0.1, 0.4]
    ##   newsletter_alphas: [0.5, 3]
    ##   newsletter_gammas: [0.3, 1]
    ##   newsletter_thetas: [0.1, 0.4]
    ##   train_size: [0.5, 0.8]

``` r
##### Guarde InputCollect en el formato de archivo JSON para importarlo más tarde
robyn_write(InputCollect, dir = "./")
```

    ## >> Exported model inputs as ./RobynModel-inputs.json

``` r
InputCollect <- robyn_inputs(
  dt_input = dt_simulated_weekly,
  dt_holidays = dt_prophet_holidays,
  json_file = "./RobynModel-inputs.json")
```

    ## Imported JSON file succesfully: ./RobynModel-inputs.json

    ## >> Running feature engineering...

### Apartado 1

``` r
library(PASWR2)
```

    ## Warning: package 'PASWR2' was built under R version 4.2.3

    ## Loading required package: lattice

``` r
summary(CARS2004) 
```

    ##            country        cars           deaths        population   
    ##  Austria       : 1   Min.   :222.0   Min.   : 33.0   Min.   :  400  
    ##  Belgium       : 1   1st Qu.:354.0   1st Qu.: 72.0   1st Qu.: 3446  
    ##  Cyprus        : 1   Median :448.0   Median :112.0   Median : 8976  
    ##  Czech Republic: 1   Mean   :432.1   Mean   :111.4   Mean   :18273  
    ##  Denmark       : 1   3rd Qu.:491.0   3rd Qu.:135.0   3rd Qu.:16258  
    ##  Estonia       : 1   Max.   :659.0   Max.   :222.0   Max.   :82532  
    ##  (Other)       :19

Como puedes observar, al compilar tu documento aparecen las sentencias
de `R` y el output que te da el programa.

### Apartado 2

Ahora vamos a utilizar la función `eda` del paquete `PASWR2` para
realizar un análisis exploratorio de la variable `deaths`

``` r
eda(CARS2004$deaths)
```

![](20231006MarketingMixModels_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

    ## Size (n)  Missing  Minimum   1st Qu     Mean   Median   TrMean   3rd Qu 
    ##   25.000    0.000   33.000   72.000  111.400  112.000  110.000  135.000 
    ##      Max    Stdev      Var  SE Mean   I.Q.R.    Range Kurtosis Skewness 
    ##  222.000   47.023 2211.167    9.405   63.000  189.000    0.043    0.578 
    ## SW p-val 
    ##    0.243

En este caso, en tu documento final te aparece el código de `R`, el
output numérico de la función `eda` y el output gráfico de la función
`eda`.
