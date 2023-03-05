# Simulación numérica del problema de Rayleigh-Bénard 

## Detalle del mallado y de las condiciones de borde

El recinto de la simulación consiste en un volumen de 9x1x2 m y la malla en 90x20x1 celdas. Esto se traduce en celdas de 10x5x1 cm que priorizan la obtención de información en la dirección vertical debido a que es en este eje en donde se esperan más cambios.  
La profundidad unitaria de las celdas da cuenta de la bidimensionalidad del problema.

La malla se generó con la utilidad `blockMesh`. La siguiente figura muestra la malla con sus respectivas dimensiones de recinto y de celda:

![Detalle de la malla](/B1900t10000/Detalle_mallado.png)

Tanto las paredes superior e inferior como izquierda y derecha del recinto se consideran sólidos (`wall`) mientras que a las caras frontal y posterior se les atribuye la condición `empty` que denota la irrelevancia de la dirección en la solución numérica.

La temperatura en la pared inferior es $T_H=300\\,K$ y la de la pared superior $T_C=301\\,K$, ambas son uniformes y permanecen constantes. Para las paredes laterales se fija una condición de gradiente nulo (`zeroGradient`).  
El campo de velocidades es uniforme y vale $U(x, y, z) = (1e-4, 0, 0)\\, m/s$. Para las paredes se impone la condición de no deslizamiento.  
El campo de presiones es uniforme y nulo.

Si bien, explícitamente no hay condiciones de simetría, [los resultados de las simulaciones](#anexo-i-tabla-de-resultados) muestran que los vórtices son, efectivamente, estructuras simétricas ya que los valores máximos y mínimos en la dirección horizontal y en la vertical son siempre del mismo orden y coincidentes en, al menos, 2 dígitos.

## Parámetros numéricos

El solver empleado es `buoyantPimpleFoam` y es el más recomendado para simulaciones en régimen transitorio en casos de transferencia de calor por permitir considerar la aproximación de Boussinesq y simplificar, de esta manera, el término de las fuerzas volumétricas (flotación) en la ecuación de conservación de la cantidad de movimiento.

Como el regimen es claramente laminar, el modelo de turbulencia no merece consideración. No obstante, en el código se adopta el modelo *k-epsilon* del tipo RAS.

Los intervalos de tiempo simulados son de $t=10000 \\, s$ con un paso de $\\Delta T = 1 \\, s$. Los intervalos de escritura son de $50 \\, s$. Los mismos se encuentran especificados en [controlDict](/B1900t10000/system/controlDict).

Más detalles acerca de los parámetros numéricos se encuentran en los archivos [fvSchemes](/B1900t10000/system/fvSchemes) y [fvSolution](/B1900t10000/system/fvSolution).

## Propiedades físicas del sistema

El fluido del recinto es agua y presenta las siguientes propiedades:

- Peso molecular: $M = 18 \\, g/mol$
- Densidad: $\\rho = 1 \\, kg/m^{3}$
- Viscosidad cinemática: $\\nu = 5 \times 10^{-3} \\: m^{2}/s$
- Número de Prandtl: $Pr = 5$
- Coeficiente de expansión térmica: $\\beta = 1 \times 10^{-3} \\: 1/K$
- Difusividad térmica: $a = \\frac{\\nu}{Pr} = 5 \\, m^{2}/s$

Para determinar el valor del número de Rayleigh ($Ra$) correspondiente a cada simulación, se trabaja sobre el valor de la gravedad. Esto se debe a que, si bien la situación no es necesariamente realista, se trata de una constante universal directamente proporcional al número adimensional por lo que se considera una opción de  suma conveniencia.  
Por ejemplo, si se calcula el valor del número de Rayleigh para la gravedad terrestre, se tiene:

$Ra= \\frac{g \\, \\cdot \\, \\beta}{\\nu \\, \cdot \\, a} \cdot (T_H - T_C) \cdot h^{3} = \\frac{9.8 \\, m/s^{2} \\, \\cdot \\, 1 \times 10^{-3} \\: 1/K}{5 \times 10^{-3} \\: m^{2}/s \\, \cdot \\, 5 \\, m^{2}/s} \cdot (301 \\, K - 301 \\, K) \cdot (1 \\, m)^{3} = 1960$

## Determinación del umbral de convección natural

Para hallar el valor crítico de $Ra$, se realizó una serie de simulaciones para distintos valores del mismo y se procuró determinar la velocidad máxima en la dirección vertical para el último instante ($t=10000$) a modo de obtener una curva a la obtenida por [Wesfreid et al. [1978]](https://www.researchgate.net/profile/Jose-Wesfreid/publication/43326017_Critical_effects_in_Rayleigh-Benard_convection/links/00463518264c70c91a000000/Critical-effects-in-Rayleigh-Benard-convection.pdf).

Se [ajustan](/rayleigh_benard2.ipynb) los puntos obtenidos de acuerdo a la ley potencial $V = V_0 \\cdot \\varepsilon ^{1/2}$ donde $\\varepsilon = \\frac{Ra-Ra_c}{Ra_c}$ (Bergé et al. [1975]) y se obtienen los siguientes gráficos:

![Ajustes_rb](/rb_plot.png)

Se aprecia sin relleno el punto correspondiente al valor crítico, que corresponde a $Ra_c=1731,66$.

Estudios experimentales (Jeffreys, H. [1928]) demostraron que el valor crítico corresponde a $Ra_c=1708$ lo que implica un error relativo de un $1,39 \\, \\%$ para el resultado obtenido en la simulación numérica y permite concluir que la misma valida satisfactoriamente la evidencia experimental.

## Anexo I: Tabla de resultados

| Ra   | Ux           | Uy           | Uz          | U           | Uxmax       | Uxmin        | Uymax       | Uymin        |
|:----:|:------------:|:------------:|:-----------:|:-----------:|:-----------:|:------------:|:-----------:|:------------:|
| 1700 | -8,19292E-07 | -3,31475E-09 | 0           | 8,19299E-07 | 8,19002E-07 | -8,19292E-07 | 8,04311E-07 | -8,00715E-07 |
| 1725 | -1,61525E-06 | -4,84806E-08 | 0           | 1,61598E-06 | 1,61468E-06 | -1,61525E-06 | 1,57016E-06 | -1,61328E-06 |
| 1750 | -5,79664E-06 | -2,82295E-07 | 0           | 5,80351E-06 | 5,7946E-06  | -5,79664E-06 | 5,73878E-06 | -5,71496E-06 |
| 1775 | -5,84568E-07 | -4,53934E-05 | 2,69371E-24 | 4,53972E-05 | 4,43225E-05 | -4,43062E-05 | 4,33513E-05 | -4,53934E-05 |
| 1800 | 2,01063E-06  | -0,000360639 | 0           | 0,000360645 | 0,000351712 | -0,000351536 | 0,000345999 | -0,000360639 |
| 1812 | 2,18582E-06  | -0,000920583 | 0           | 0,000920586 | 0,000897697 | -0,000897047 | 0,000886241 | -0,000920583 |
| 1825 | -2,13631E-06 | -0,00198987  | 0           | 0,001989871 | 0,00194324  | -0,00194128  | 0,00193394  | -0,00198987  |
| 1850 | -2,69883E-05 | -0,00308753  | 0           | 0,003087648 | 0,00301954  | -0,00301631  | 0,00305029  | -0,00308753  |
| 1875 | -5,63762E-05 | -0,00343962  | 0           | 0,003440082 | 0,0033557   | -0,00335124  | 0,00340983  | -0,00343962  |
| 1900 | 4,65398E-05  | 0,00375122   | 4,45001E-22 | 0,003751509 | 0,00365721  | -0,00365784  | 0,00375122  | -0,00374949  |
| 1960 | -1,72519E-06 | 0,00444693   | 5,28658E-22 | 0,00444693  | 0,00431306  | -0,000025703 | 0,00444693  | -0,00444635  |
| 2100 | -4,94175E-05 | -0,00564152  | 0           | 0,005641736 | 0,00543965  | -0,00543943  | 0,00563025  | -0,00564152  |

## Anexo II: Videos

Las siguientes simulaciones corresponden a valores de $Ra=1960$, intervalos de tiempo $t=10000 \\, s$ y, para una mejor apreciación del fenómeno, se realizan escrituras cada $10 \\, s$.

### Temperatura
[![VideoT](/B1900t10000/Ra1960t10000_T_preview.png)](https://drive.google.com/file/d/1wK7DCpHmZBhm0HHUPW8WgpBd5w-wMazR/view?usp=sharing)

### Velocidad (valor absoluto)
[![VideoU](/B1900t10000/Ra1960t10000_U_preview.png)](https://drive.google.com/file/d/1wU70bRDix9Pj-kZfNyIOt5o7c9976dlI/view?usp=sharing)

### Velocidad horizontal
[![VideoUx](/B1900t10000/Ra1960t10000_Ux_preview.png)](https://drive.google.com/file/d/1wJ7gdjOLyfOpzFSEocXVVJOKu2eH8J-I/view?usp=sharing)

### Velocidad vertical
[![VideoUy](/B1900t10000/Ra1960t10000_Uy_preview.png)](https://drive.google.com/file/d/1wRRTLcBRNPjp-T7v4KlpV5BwzWFegG1H/view?usp=sharing)
