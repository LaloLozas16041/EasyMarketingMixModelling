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
estacionales también se denominan series de tiempo periódicas.

Por ejemplo, las ventas minoristas aumentan en algunos festivales o
eventos en particular, o la temperatura del clima muestra su
comportamiento estacional de días cálidos en verano y días fríos en
invierno, etc.

``` 2
ggseasonplot(AirPassengers)
```

``` 2
#Step 1.a.First Install required Packages
install.packages("Robyn")
install.packages("reticulate")
library(reticulate)

#Step 1.b Setup virtual Environment & Install nevergrad library
virtualenv_create("r-reticulate")
py_install("nevergrad", pip = TRUE)
use_virtualenv("r-reticulate", required = TRUE)
```

## Including Plots

You can also embed plots, for example:

![](Prueba_files/figure-gfm/pressure-1.png)<!-- -->

Note that the `echo = FALSE` parameter was added to the code chunk to
prevent printing of the R code that generated the plot.