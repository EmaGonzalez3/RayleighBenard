# Simulación numérica del problema de Rayleigh-Bénard 

# Índice
1. [Detalle del mallado y de las condiciones de borde](#1-detalle-del-mallado-y-de-las-condiciones-de-borde)  
    1.1. [Influencia del tamaño de malla](#11-influencia-del-tamaño-de-malla)  
    1.2. [Condiciones de borde](#12-condiciones-de-borde)
2. [Parámetros numéricos](#2-parámetros-numéricos)
3. [Propiedades físicas del sistema](#3-propiedades-físicas-del-sistema)  
    3.1. [Tiempos característicos](#31-tiempos-característicos)  
4. [Determinación del umbral de convección natural](#4-determinación-del-umbral-de-convección-natural)
5. [Anexo](#5-anexo)  
    5.1. [Anexo I: Tabla de resultados](#51-anexo-i-tabla-de-resultados)  
    5.1. [Anexo II: Videos](#52-anexo-ii-videos)
6. [Bibliografía](#6-bibliografía)

## 1. Detalle del mallado y de las condiciones de borde

El recinto de la simulación consiste en un volumen de 9x1x2 m y la malla en 90x20x1 celdas. Esto se traduce en celdas de 10x5x1 cm que priorizan la obtención de información en la dirección vertical debido a que es en este eje en donde se esperan más cambios.  
La profundidad unitaria de las celdas da cuenta de la bidimensionalidad del problema.

La malla se generó con la utilidad `blockMesh`. La siguiente figura muestra la malla con sus respectivas dimensiones de recinto y de celda:

![Detalle de la malla](/B1900t10000/Detalle_mallado.png)

### 1.1. Influencia del tamaño de malla

Si bien un mallado más fino permite tener simulaciones más precisas, también aumenta el costo computacional y, por ende, el tiempo requerido para efectuar los cálculos así como el tamaño de los archivos. A modo de optimizar la malla, se hicieron 3 pruebas con distintos tamaños:

- Una malla gruesa de 900 celdas (90x10x1)
- Una malla media de 1800 celdas (90x20x1)
- Una malla fina de 4500 celdas (90x50x1)

Al [comparar](/analisis_mallado.ipynb "Cuaderno de cálculo de análisis del tamaño de malla") las [velocidades máximas obtenidas](/tabla_malla.csv "Tabla con velocidades para distintos tamaños de malla") luego de simular el mismo intervalo ($t = 10000 \\, s$) para distintos valores del número de Rayleigh se observa lo siguiente:

![Comparación tamaños de malla](/mallado_plot.png)

Para valores menores del parámetro adimensional en cuestión, la malla media proporciona resultados muy cercanos a los de la más fina mientras que la gruesa no es tan precisa. A medida que el $Ra$ aumenta la malla media comienza a diferir en mayor medida de la fina para, cerca de $Ra = 1900$ comportarse de forma muy similar a la gruesa. No obstante, debido a que el tiempo de simulación de la malla media es apenas el doble que el de la gruesa y aproximadamente 5 veces menor que el de la fina, resulta conveniente adoptarla para las simulaciones a efectuar. De esta manera, se concluye que la malla de 1800 celdas es la opción óptima.

Como análisis adicional, se puede inferir que para mallas más finas, la velocidad comienza a aumentar a valores de $Ra$ más altos. Se podría presuponer que el valor crítico determinado sea mayor a medida que la cantidad de celdas aumenta pero dadas las compliaciones ya mencionadas acerca de las simulaciones con este tipo de mallas, este análisis no será efectuado.

### 1.2. Condiciones de borde

Tanto las paredes superior e inferior como izquierda y derecha del recinto se consideran sólidos (`wall`) mientras que a las caras frontal y posterior se les atribuye la condición `empty` que denota la irrelevancia de la dirección en la solución numérica.

La temperatura en la pared inferior es $T_H=300\\,K$ y la de la pared superior $T_C=301\\,K$, ambas son uniformes y permanecen constantes. Para las paredes laterales se fija una condición de gradiente nulo (`zeroGradient`).  
El campo de velocidades es uniforme y vale $U(x, y, z) = (1e-4, 0, 0)\\, m/s$. Para las paredes se impone la condición de no deslizamiento.  
El campo de presiones es uniforme y nulo.

Si bien, explícitamente no hay condiciones de simetría, [los resultados de las simulaciones](#anexo-i-tabla-de-resultados) muestran que los vórtices son, efectivamente, estructuras simétricas ya que los valores máximos y mínimos en la dirección horizontal y en la vertical son siempre del mismo orden y coincidentes en, al menos, 2 dígitos.

## 2. Parámetros numéricos

Las simulaciones se efectuaron por medio del software [OpenFOAM (version 8)](https://www.openfoam.com/) y la visualización se llevó a cabo en [Paraview (5.6.2)](https://www.paraview.org/). Ambos se pueden obtener desde los sitios oficiales o a través de [blueCFD](https://bluecfd.github.io/Core/Downloads/).  
El solver empleado es `buoyantPimpleFoam` y es el más recomendado para simulaciones en régimen transitorio en casos de transferencia de calor por permitir considerar la aproximación de Boussinesq y simplificar, de esta manera, el término de las fuerzas volumétricas (flotación) en la ecuación de conservación de la cantidad de movimiento.

Como el regimen es claramente laminar, el modelo de turbulencia no merece consideración. No obstante, en el código se adopta el modelo *k-epsilon* del tipo RAS.

Los intervalos de tiempo simulados son de $t=10000 \\, s$ con un paso de $\\Delta T = 1 \\, s$. Los intervalos de escritura son de $50 \\, s$. Los mismos se encuentran especificados en [controlDict](/B1900t10000/system/controlDict).

Más detalles acerca de los parámetros numéricos se encuentran en los archivos [fvSchemes](/B1900t10000/system/fvSchemes) y [fvSolution](/B1900t10000/system/fvSolution).

## 3. Propiedades físicas del sistema

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

### 3.1. Tiempos característicos

Otra interpretación del número de Rayleigh es la comparación entre los tiempos difusivo, viscoso y de flotación según la relación $Ra = \\frac{\\tau_d}{\\tau_f} \\cdot \\frac{\\tau_v}{\\tau_f}$  
Calculando cada uno de los tiempos característicos considerando las propiedades determinadas y la gravedad terrestre, es decir, $Ra = 1960$, se obtienen los siguientes resultados:

$\\tau_v = \\frac{h^2}{\\nu} = \\frac{(1 \\, m)^{2}}{5 \times 10^{-3} \\: m^{2}/s} = 200 \\, s$

$\\tau_d = \\frac{h^2}{a} = \\frac{(1 \\, m)^{2}}{5 \\, m^{2}/s} = 1000 \\, s$

$\\tau_f = (\\frac{g \\, \\cdot \\, \\beta \\, \\cdot \\, \\Delta T}{h})^{-1/2} = (\\frac{9.8 \\, m/s^{2} \\, \\cdot \\, 1 \times 10^{-3} \\: 1/K \\, \\cdot \\, 1 \\, K}{1 \\, m})^{-1/2} \\simeq 10,10$

Además de comprobar que se verifica el valor del parámetro adimensional ($Ra = \\frac{\\tau_d}{\\tau_f} \\cdot \\frac{\\tau_v}{\\tau_f} = \\frac{1000}{10,10} \\cdot \\frac{200}{10,10} = 1960$), se puede apreciar que el tiempo de simulación adoptado $t=10000 \\, s$ es 10 veces mayor al valor del tiempo carácteristico más alto ($\\tau_d$). De esta manera, se asume que el intervalo simulado es lo suficientemente amplio para que se manifiesten los 3 fenómenos que intervienen en el análisis.

A modo de contraejemplo, modificando las propiedades de modo tal que $\\tau_d = \\tau_v = 1000 \\, s$ y simulando un tiempo de $t=1000 \\, s$ se obtiene [datos](/Ra2575/rb_t1000.csv) que permiten confeccionar la siguiente [curva](/Ra2575/rb_t1000.ipynb):

![Curva t1000](/Ra2575/rb_plot_t1000.png)

Como el tiempo de simulación apenas coincide con el tiempo característico viscoso y difusivo, los resultados implican que el valor crítico $Ra_c$ tiende a ser mayor del esperado (cerca de 2000). Esto puede explicarse justamente por la falta de tiempo para la correcta manifestación de los procesos implicados

## 4. Determinación del umbral de convección natural

Para hallar el valor crítico de $Ra$, se realizó una serie de simulaciones para distintos valores del mismo y se procuró determinar la velocidad máxima en la dirección vertical para el último instante ($t=10000$) a modo de obtener una curva a la obtenida por [Wesfreid et al. [1978]](https://www.researchgate.net/profile/Jose-Wesfreid/publication/43326017_Critical_effects_in_Rayleigh-Benard_convection/links/00463518264c70c91a000000/Critical-effects-in-Rayleigh-Benard-convection.pdf).

Se [ajustan](/rayleigh_benard4.ipynb) los puntos obtenidos de acuerdo a la ley potencial $V = V_0 \\cdot \\varepsilon ^{1/2}$ donde $\\varepsilon = \\frac{Ra-Ra_c}{Ra_c}$ (Bergé et al. [1975]) y se obtienen los siguientes gráficos:

![Ajustes_rb](/rb_plot3.png)

Se aprecia sin relleno el punto correspondiente al valor crítico, que corresponde a $Ra_c=1646.

Estudios experimentales (Jeffreys, H. [1928]) demostraron que el valor crítico corresponde a $Ra_c=1708$ lo que implica un error relativo de un $2,23 \\, \\%$ para el resultado obtenido en la simulación numérica y permite concluir que la misma valida satisfactoriamente la evidencia experimental.

Cabe aclarar que se consideraron para el ajuste los puntos correspondientes a los valores entre $Ra = 1662$ y $Ra = 1750$ de la [tabla de resultados](#anexo-i-tabla-de-resultados), esto se debe a que la validez del ajuste tiene un rango limitado alrededor del umbral de convección natural.

Sobre lo expresado en los [comentarios](#influencia-del-tamaño-de-malla) acerca de la influencia del tamaño de malla, se ve que el valor crítico efectivamente tiene un error en defecto. Si bien escapa al análisis pertinente, sería coherente suponer que al trabajar con mallas más finas el resultado se acerque más al valor experimental.

## 5. Anexo

## 5.1. Anexo I: Tabla de resultados

[Tabla de resultados](/tabla_resultados.md)

## 5.2. Anexo II: Videos

Las siguientes simulaciones corresponden a valores de $Ra=1960$, intervalos de tiempo $t=10000 \\, s$ y, para una mejor apreciación del fenómeno, se realizan escrituras cada $10 \\, s$.

### Temperatura
[![VideoT](/B1900t10000/Ra1960t10000_T_preview.png)](https://drive.google.com/file/d/1wK7DCpHmZBhm0HHUPW8WgpBd5w-wMazR/view?usp=sharing)

### Velocidad (valor absoluto)
[![VideoU](/B1900t10000/Ra1960t10000_U_preview.png)](https://drive.google.com/file/d/1wU70bRDix9Pj-kZfNyIOt5o7c9976dlI/view?usp=sharing)

### Velocidad horizontal
[![VideoUx](/B1900t10000/Ra1960t10000_Ux_preview.png)](https://drive.google.com/file/d/1wJ7gdjOLyfOpzFSEocXVVJOKu2eH8J-I/view?usp=sharing)

### Velocidad vertical
[![VideoUy](/B1900t10000/Ra1960t10000_Uy_preview.png)](https://drive.google.com/file/d/1wRRTLcBRNPjp-T7v4KlpV5BwzWFegG1H/view?usp=sharing)

### Vorticidad
[![VideoOmega](/B1900t10000/Ra1960t10000_omega_preview.png)](https://drive.google.com/file/d/1xLCPACeWyeALOYaZgiZjuH0hiTd936Wh/view?usp=sharing)

## 6. Bibliografía

Wesfreid, J., Pomeau, Y., Dubois, M., Normand, C., & Bergé, P. (1978). Critical effects in Rayleigh-Bénard convection. Journal de Physique, 39(7), 725-731.

Bergé, P. (1975). Rayleigh-Benard instability: experimental findings obtained by light scattering and other optical methods. Fluctuations, Instabilities, and Phase Transitions, 323-352.

Jeffreys, H. (1928). Some cases of instability in fluid motion. Proceedings of the Royal Society of London. Series A, Containing Papers of a Mathematical and Physical Character, 118(779), 195-208.

Greenshields, C. y Weller, H. (2022). Notes on Computational Fluid Dynamics: General Principles. Reading, UK: CFD Direct Ltd.

D'Adamo, J. (2022). Transferencia de Calor y Masa. Universidad de Buenos Aires.


