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
