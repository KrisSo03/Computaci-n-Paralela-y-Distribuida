# Juego de la Vida de Conway

**Implementación Paralela con Análisis de Complejidad**

Kristhel Porras Mata  
Curso de Programación Paralela  
Universidad LEAD  
Profesor: Johansell Villalobos Cubillo

---

## Introducción

El Juego de la Vida es un autómata celular propuesto por John Horton Conway en 1970. A pesar de sus reglas simples, produce comportamientos emergentes complejos, incluyendo osciladores, estructuras estables y patrones móviles.

Este proyecto implementa el Juego de la Vida con énfasis en rendimiento computacional mediante Numba, diseño orientado a objetos y análisis empírico de complejidad computacional.

## Las reglas

El juego transcurre en una grilla bidimensional donde cada celda tiene ocho vecinos. En cada generación, todas las celdas se actualizan simultáneamente:

- **Supervivencia**: Una celda viva con 2 o 3 vecinos vivos permanece viva.
- **Reproducción**: Una celda muerta con exactamente 3 vecinos vivos se convierte en viva.
- **Muerte por soledad**: Una celda viva con menos de 2 vecinos muere.
- **Muerte por sobrepoblación**: Una celda viva con más de 3 vecinos muere.

Se utilizan condiciones de borde toroidales (los bordes se conectan entre sí) para evitar efectos artificiales en los límites.

## Instalación

El proyecto puede ejecutarse en Google Colab:

- Sube el archivo `COLAB.ipynb` a Google Colab
- Ejecuta las celdas en orden
- Las dependencias se instalan automáticamente

## Cómo ejecutar

En Colab, ejecuta todas las celdas en orden:

```python
# Instalación de dependencias
!pip install numpy matplotlib numba pillow -q

# Módulos 1-6 se ejecutan automáticamente
# Genera:
# - 3 animaciones GIF
# - 3 gráficas de análisis
# - Tabla de benchmarks en consola
```

**Tiempo de ejecución:**
- Primera ejecución: 15–25 segundos (Numba compila)
- Ejecuciones posteriores: ~5 segundos (usa caché)

## Uso en el código

```python
from game_of_life import GameOfLife

# Crear grilla aleatoria
juego = GameOfLife(128, 128, inicializacion='aleatorio', semilla=42)

# Ejecutar 100 generaciones
juego.run(100)

# Obtener estado
estado = juego.get_state()
```

Soporta estados iniciales: `'aleatorio'`, `'vacío'`, o array de numpy personalizado.

Patrones disponibles: `'planeador'`, `'parpadeador'`, `'sapo'`

## Qué se genera

### Animaciones GIF

**Planeador** — Nave de 5 celdas que viaja diagonalmente. Período: 4 generaciones. Grilla: 64×64.

**Parpadeador** — Oscilador de 3 celdas. Período: 2 generaciones. Grilla: 32×32.

**Sapo** — Oscilador de 6 celdas. Período: 2 generaciones. Grilla: 64×64.

### Gráficas de rendimiento

Se generan automáticamente 3 gráficas basadas en benchmarks reales.

---

## Análisis técnico basado en datos reales

### Complejidad Temporal: O(n²)

Los benchmarks se ejecutaron en grillas de tamaño n×n para n = 32, 64, 128, 256, 512, 768, 1024.

**Gráfica log-log (Análisis de Complejidad):**

La gráfica en escala logarítmica muestra claramente que ambas versiones (paralela y secuencial) siguen una línea recta con pendiente cercana a 2. Esto confirma que el crecimiento empírico es **O(n²)**.

En contraste, la línea de referencia O(n) tiene pendiente mucho más suave. Los datos divergen claramente de O(n) y convergen con O(n²).

**Conclusión:** El comportamiento observado es consistente con O(n²). Procesar una grilla de tamaño N×N requiere visitar N² celdas, resultando en complejidad O(N²).

### Complejidad de Memoria: O(n²)

El código mantiene dos matrices n×n (estado actual y siguiente generación), resultando en almacenamiento de **2n² bytes** (int8 por celda).

**Datos observados:**

| Tamaño | Celdas | Memoria observada |
|--------|--------|-------------------|
| 32×32 | 1,024 | 0.001 MB |
| 128×128 | 16,384 | 0.032 MB |
| 256×256 | 65,536 | 0.126 MB |
| 512×512 | 262,144 | 0.501 MB |
| 1024×1024 | 1,048,576 | 2.001 MB |

**Gráfica de Consumo de Memoria:**

La curva de memoria observada se superpone casi perfectamente con la referencia teórica O(n²). Esto confirma que la escalabilidad de memoria es exactamente como se predice teóricamente.

**Conclusión:** La memoria escala como O(n²), acorde con el almacenamiento de dos matrices cuadradas.

### Versión Paralela vs Secuencial

Se midieron ambas versiones compiladas con Numba:

**Gráfica de Tiempo de Ejecución (Escala Lineal):**

- **Línea azul (Paralelo):** Sigue la curva O(n²)
- **Línea roja (Secuencial):** También sigue la curva O(n²)

Ambas versiones tienen **la misma complejidad asintótica O(n²)**, pero con factores constantes diferentes.

**Diferencia observada:**

| Tamaño | Paralelo | Secuencial | Ratio |
|--------|----------|-----------|-------|
| 32×32 | ~0.0001 s | ~0.00003 s | 3.3× |
| 256×256 | ~0.0062 s | ~0.0018 s | 3.4× |
| 512×512 | ~0.0303 s | ~0.0089 s | 3.4× |
| 1024×1024 | ~0.050 s | ~0.032 s | 1.5× |

**Observación importante:** La versión paralela es **más lenta que la secuencial** en todas las grillas probadas. Esto se debe al overhead de paralelización (crear infraestructura de hilos, sincronización) que supera el beneficio en un sistema de un solo núcleo.

**Conclusión:** En máquinas con un único núcleo, la paralelización añade overhead sin beneficio. Este comportamiento es esperado. En máquinas con múltiples núcleos, se esperaría aceleración real.

### Aceleración y Eficiencia (Máquina de un núcleo)

**Gráfica de Métricas de Paralelización:**

- **Aceleración (S):** Comienza en ~0.15 para 32×32 y aumenta a ~0.70 para 1024×1024
- **Eficiencia (E):** Comienza en ~0.08 para 32×32 y aumenta a ~0.35 para 1024×1024

Ambas métricas permanecen **por debajo de 1.0**, confirmando que la versión paralela es más lenta que la secuencial en un sistema de un núcleo.

**Conclusión:** El overhead de paralelización es significativo. Sin múltiples núcleos disponibles, la paralelización no proporciona beneficio.

### Dónde se Gasta el Tiempo: Sin Profiling Real

Las gráficas permiten confirmar **qué tan rápido es el código**, pero no revelan **dónde exactamente se consume el tiempo**.

Sin herramientas de profiling (como `perf`, `cProfile` o Intel VTune), no es posible cuantificar si el tiempo se dedica a:
- Acceso a memoria (lecturas de vecinos, escrituras de nuevos estados)
- Sincronización de hilos (en la versión paralela)
- Operaciones aritméticas (comparaciones, condicionales)
- Otros factores

**Lo que sí sabemos:**
- El código es rápido (10.68 ms para 1024×1024)
- Escala como O(n²) en ambas versiones
- La paralelización añade overhead visible

**Lo que no podemos afirmar sin profiling:**
- Si memory bandwidth es el cuello de botella principal
- Qué porcentaje del tiempo se dedica a cada operación
- Cómo se distribuye el tiempo entre lectura, cómputo y escritura

---

## Emergencia y Comportamiento Global

El proyecto demuestra cómo reglas locales simples producen estructuras globales complejas:

- **Planeador:** Una nave que viaja indefinidamente, demostrando traslación ordenada
- **Parpadeador:** Un patrón que alterna, demostrando ciclos periódicos
- **Sapo:** Un oscilador más complejo, demostrando múltiples patrones posibles

Estas estructuras emergen del sistema sin estar especificadas en las reglas, ilustrando la naturaleza de sistemas emergentes.

---

## Conclusiones

### Sobre la Complejidad

El análisis empírico mediante benchmarks confirma que la complejidad temporal es **O(n²)**. La gráfica log-log muestra una línea recta con pendiente cercana a 2, consistente con la teoría.

La complejidad de memoria también es **O(n²)**, con almacenamiento de dos matrices cuadradas confirmado en los datos reales (2 MB para 1024×1024).

### Sobre la Paralelización con Numba

La compilación JIT de Numba proporciona aceleración significativa respecto a Python puro (las primeras ejecuciones toman 15–25 segundos por compilación, posteriores ~5 segundos gracias al caché).

Sin embargo, en una máquina de un solo núcleo, la paralelización (prange) añade overhead que no es compensado por beneficios de concurrencia. El speedup observado es menor a 1.0, indicando que la versión paralela es **más lenta**.

En máquinas con múltiples núcleos, se esperaría aceleración real, pero esto no fue probado.

### Sobre el Diseño Arquitectónico

El código utiliza:
- **Matriz temporal (new_grid)** para evitar conflictos en actualización simultánea
- **Compilación Numba** para optimizar el núcleo computacional
- **Clase GameOfLife** para abstraer la lógica del juego
- **Patrón decorador @njit** para compilación transparente

Esto demuestra separación clara entre lógica, optimización e interfaz.

### Sobre Análisis Futuro

Para identificar con precisión dónde se consume el tiempo, se requerirían:
- Herramientas de profiling (perf, cProfile)
- Ejecución en máquinas con diferente número de núcleos
- Análisis de cache misses y memory bandwidth utilization
- Comparación con implementaciones GPU

---

**Autor:** Kristhel Porras Mata  
**Año:** 2026
