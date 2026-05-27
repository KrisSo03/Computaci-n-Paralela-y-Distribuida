# Juego de la Vida de Conway

**Implementación Paralela con Análisis de Complejidad**

Kristhel Porras Mata  
Curso de Programación Paralela  
Universidad LEAD  
Profesor: Johansell Villalobos Cubillo

---

## Introducción

El Juego de la Vida es un autómata celular propuesto por John Horton Conway en 1970. A pesar de sus reglas simples, produce comportamientos emergentes complejos, incluyendo osciladores, estructuras estables y patrones móviles.

En este proyecto implementé el Juego de la Vida con énfasis en rendimiento computacional mediante Numba, diseño orientado a objetos y análisis empírico de complejidad computacional utilizando benchmarks y visualizaciones comparativas.

## Las reglas

El juego transcurre en una grilla bidimensional donde cada celda tiene ocho vecinos. En cada generación, todas las celdas se actualizan simultáneamente:

- **Supervivencia**: Una celda viva con 2 o 3 vecinos vivos permanece viva.
- **Reproducción**: Una celda muerta con exactamente 3 vecinos vivos se convierte en viva.
- **Muerte por soledad**: Una celda viva con menos de 2 vecinos muere.
- **Muerte por sobrepoblación**: Una celda viva con más de 3 vecinos muere.

Utilicé condiciones de borde toroidales, donde los bordes de la grilla se conectan entre sí para evitar efectos artificiales en los límites.

## Instalación

El proyecto puede ejecutarse localmente o en Google Colab.

Sube el archivo `.ipynb` a Google Colab y ejecuta las celdas en orden.  
La mayoría de dependencias ya vienen preinstaladas.

## Cómo ejecutar

```
python game_of_life.py
```

El script genera automáticamente:
- Tres animaciones GIF de patrones clásicos
- Tres gráficas de análisis de rendimiento
- Una tabla de benchmarks en la consola

Primera ejecución: 15–25 segundos (Numba compila el código)  
Ejecuciones posteriores: ~5 segundos (usa caché)

## Uso en el código

```python
from game_of_life import JuegoVida

# Crear una grilla aleatoria de 128×128
juego = JuegoVida(
    filas=128,
    columnas=128,
    inicializacion='aleatorio',
    semilla=42
)

# Ejecutar la simulación durante 100 generaciones
juego.ejecutar(100)

# Obtener el estado final de la grilla
estado = juego.obtener_estado()
```

La clase admite estados iniciales aleatorios, vacíos o personalizados. También permite colocar patrones clásicos fácilmente:

```python
juego = JuegoVida(64, 64, inicializacion='vacío')
juego.colocar('planeador', fila=10, columna=10)
juego.colocar('parpadeador', fila=30, columna=30)
juego.ejecutar(50)
```

Los patrones disponibles son: `'planeador'` (nave que se desplaza), `'parpadeador'` (oscilador simple), `'sapo'` (oscilador más complejo).

## Qué se genera

### Animaciones

Tres patrones clásicos evolucionan en grillas de diferentes tamaños:

**Planeador** — La nave más pequeña (5 celdas) que se desplaza diagonalmente. Viaja indefinidamente sin crecer, demostrando que la información puede "navegar" el universo. Período de 4 generaciones.

**Parpadeador** — El oscilador más simple (3 celdas en línea). Alterna entre horizontal y vertical cada 2 generaciones. Aparece frecuentemente dentro de patrones más grandes.

**Sapo** — Un oscilador más complejo (6 celdas) que rota. También tiene período 2 pero con estructura más interesante que el Parpadeador.

### Gráficas

**Gráfica lineal** — La gráfica muestra el tiempo de ejecución respecto al tamaño de la grilla en escala normal. Tanto la versión secuencial como la paralela presentan crecimiento cuadrático.

**Gráfica log-log** — El análisis logarítmico permite observar que los datos siguen un comportamiento cercano a O(n²), consistente con la complejidad teórica esperada.

**Speedup y Memoria** — Panel izquierdo muestra el factor de aceleración (secuencial dividido por paralelo) y eficiencia. Panel derecho muestra consumo de memoria, que escala como O(n²) como esperado para dos matrices de n×n.

## Análisis técnico

### Complejidad Temporal: O(n²)

En el Juego de la Vida, procesar una grilla de tamaño N×N requiere visitar cada celda una vez por generación. Por esta razón, la complejidad temporal es O(N²).

Los resultados experimentales obtenidos son consistentes con este comportamiento teórico. En la gráfica log-log, el crecimiento cuadrático aparece aproximadamente como una línea recta con pendiente cercana a 2.

**Complejidad de Memoria: O(n²)**

Mantuve dos matrices n×n: una para el estado actual y otra para la siguiente generación. Esto implica un consumo de memoria proporcional a O(n²). El uso de una matriz temporal evita conflictos durante la actualización simultánea de las celdas.


### Cuello de Botella Principal: Memory Bandwidth
El principal límite de rendimiento no proviene de las operaciones aritméticas, sino del acceso a memoria. Cada celda requiere múltiples lecturas de vecinos y una escritura del nuevo estado. Debido a esto, gran parte del tiempo de ejecución está dominado por transferencias de memoria más que por cálculo puro.


### Análisis de Paralelización con Numba

Utilicé Numba para compilar las funciones críticas a código máquina y reducir el overhead del intérprete de Python.
En algunos tamaños de grilla, la versión paralela mostró mejoras limitadas debido al overhead asociado a: creación y sincronización de hilos, uso de prange,y restricciones de memory bandwidth.

En sistemas con pocos núcleos, este overhead puede superar el beneficio de paralelización.

**Compilación JIT (Just-In-Time):**

La primera ejecución tarda más porque Numba compila dinámicamente el código Python. Después de esta compilación inicial, las siguientes ejecuciones son significativamente más rápidas gracias al caché JIT.

Comparado con una implementación puramente interpretada en Python, el rendimiento mejora considerablemente.

### Sincronización y Actualización Simultánea

Para garantizar que todas las celdas se actualicen simultáneamente, utilicé una matriz temporal `new_grid`. Esto evita que una celda modificada afecte el cálculo de otras dentro de la misma generación.

### Limitaciones Observadas

Uno de los principales cuellos de botella secundarios fue la visualización con matplotlib. En grillas grandes, renderizar cuadros puede tomar más tiempo que calcular la generación misma. También observé que el rendimiento comienza a estar limitado por el acceso a memoria conforme aumenta el tamaño de la grilla.

## Emergencia y Comportamiento Global

Se demostró cómo reglas locales simples en un autómata celular dan lugar a estructuras globales complejas:

- **Planeador:** Una nave que viaja indefinidamente, demostrando traslación ordenada
- **Osciladores:** Patrones que se repiten, demostrando ciclos limitados
- **Comportamiento emergente:** Estas estructuras surgen sin ser especificadas en las reglas

Esto valida la naturaleza de los sistemas emergentes: complejidad global de reglas locales simples.

## Conclusiones

### Sobre la Complejidad

Los resultados experimentales obtenidos fueron consistentes con la complejidad teórica O(n²), tanto en tiempo como en memoria.

### Sobre la Paralelización

La integración de Numba permitió mejorar significativamente el rendimiento respecto a Python puro y facilitó la exploración de paralelización mediante `prange`.
Sin embargo, el speedup obtenido depende fuertemente del hardware disponible y del ancho de banda de memoria.

### Sobre la Arquitectura

El diseño orientado a objetos facilitó la separación clara entre:
- La lógica del autómata (clase `JuegoVida`)
- La optimización computacional (funciones Numba `calcular_generacion_paralela` y `calcular_generacion_secuencial`)
- La visualización (funciones de animación y gráficas)

Esta separación permitió una transición fluida de ejecución secuencial a paralela mediante directivas de Numba, sin modificar la lógica central.

### Sobre Memory Wall

Aunque Numba reduce considerablemente el overhead del intérprete de Python, el acceso a memoria continúa siendo uno de los principales límites de rendimiento en grillas grandes.

---

**Autor:** Kristhel Porras Mata  
**Año:** 2026
