---
title: "IDDFS y comparación"
---

| Notebook | Colab |
|---------|:-----:|
| Notebook 02 — Búsqueda (BFS, DFS e IDDFS en Python) | <a href="https://colab.research.google.com/github/sonder-art/ia_p26/blob/main/clase/13_simple_search/notebooks/02_busqueda.ipynb" target="_blank"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"></a> |

---

# IDDFS y comparación de algoritmos

> *"The best of both worlds."*

---

IDDFS es `busqueda_generica` con `frontera = PilaConLimite(d)`, ejecutada en un bucle que incrementa `d` desde 0 hasta encontrar la solución. Eso es todo. El resto de este capítulo es entender por qué esa idea tan simple produce un algoritmo que tiene las garantías de BFS con la eficiencia de memoria de DFS.

---

## 1. Intuición: búsqueda con radio creciente

**¿Para qué sirve IDDFS?** Úsalo cuando necesitas las mismas garantías que BFS — encontrar el camino más corto, no perderte en ramas infinitas — pero la memoria disponible no es suficiente para BFS. IDDFS es la solución cuando el problema es grande y profundo y no puedes permitirte guardar toda la frontera en memoria.

La intuición es la siguiente. Imagina que perdiste las llaves en algún lugar de tu ciudad. Estrategia BFS: examina simultáneamente *todos* los lugares a 100 metros, luego *todos* a 200 metros, etc. El problema: tienes que recordar todos los lugares que estás examinando a la vez — para distancias grandes, eso es una cantidad enorme de lugares en mente al mismo tiempo.

Estrategia IDDFS: en la primera vuelta, explora todo lo que está a menos de 100 metros usando la estrategia DFS (ve profundo por una calle, llega al límite, vuelve, prueba la siguiente). En la segunda vuelta, amplía el radio a 200 metros y repite DFS. En cada vuelta, solo tienes en mente *un camino* desde tu posición actual — mucha menos memoria.

El truco que hace esto funcionar: cuando el radio llega a la distancia real de las llaves, DFS **garantizará** encontrarlas antes de terminar esa vuelta — exactamente como BFS, porque exploró todas las posibilidades hasta ese radio.

---

## 2. En lenguaje natural

IDDFS responde la misma pregunta que BFS: *"¿cuál es el camino de menos pasos del nodo A al nodo B?"* La diferencia es cómo llega a la respuesta: en lugar de mantener toda una frontera nivel a nivel, hace **múltiples pasadas DFS** con un límite de profundidad creciente.

En cada pasada:
1. Fija un límite de profundidad $d$ (empieza en 0).
2. Ejecuta DFS normalmente, pero **ignora cualquier vecino** cuya profundidad superaría $d$.
3. Si DFS encuentra la meta: devuelve el camino. Fin.
4. Si DFS agota todos los nodos dentro del límite sin encontrar la meta: incrementa $d$ en 1 y repite desde el inicio.

La clave está en el paso 2: el límite de profundidad convierte a DFS en un explorador exhaustivo dentro de un radio fijo. Cuando $d$ iguala la profundidad de la solución, DFS la encontrará — y lo hará recorriendo exactamente los mismos nodos que hubiera explorado BFS.

---

## 3. Pseudocódigo

IDDFS tiene dos capas: el **bucle externo** que incrementa el límite, y el **DFS con límite** que es idéntico al DFS estándar con una sola diferencia.

### Capa externa: el bucle IDDFS

```
function IDDFS(problema):

    for d = 0, 1, 2, 3, ...:                    # [O1] incrementar límite de profundidad en cada pasada

        resultado ← DFS-WITH-LIMIT(problema, d)  # [O2] DFS que no pasa de profundidad d
                                                  #      internamente: busqueda_generica + PilaConLimite(d)

        if resultado ≠ FAILURE:                  # [O3] ¿encontró la meta con este límite?
            return resultado                      #      sí → devolver el camino encontrado
                                                  #      no → probar con límite d+1
```

### Capa interna: DFS con límite de profundidad

DFS-WITH-LIMIT es `busqueda_generica` con `PilaConLimite`. El pseudocódigo es **idéntico al DFS** con una sola diferencia en `push`:

```
function DFS-WITH-LIMIT(problema, limite):

    frontera ← new DepthLimitedStack(limite)    # pila con memoria de profundidad
    frontera.push(problema.inicio, padre=null)  # nodo raíz tiene profundidad 0

    explorado ← empty set
    padre ← { problema.inicio: null }

    while frontera is not empty:

        nodo ← frontera.pop()                   # [L1] LIFO → el más reciente, igual que DFS

        if problema.is_goal(nodo):              # [L2] ¿es la meta?
            return reconstruct(padre, nodo)

        explorado.add(nodo)                     # [L3] marcar como procesado

        for each vecino in problema.neighbors(nodo):  # [L4]

            if vecino not in explorado
            and vecino not in frontera:

                prof_vecino ← depth(nodo) + 1
                if prof_vecino <= limite:       # [L5] ← ÚNICA DIFERENCIA CON DFS
                    padre[vecino] ← nodo        #       ¿cabe dentro del límite? → añadir
                    frontera.push(vecino, padre=nodo)
                # si no cabe: se descarta silenciosamente (poda)

    return FAILURE
```

La línea `[L5]` es la única diferencia respecto a DFS. Todo lo demás — el bucle, el conjunto explorado, la tabla de padres, la expansión de vecinos — es idéntico.

---

## 4. Ejemplo paso a paso

Usamos un árbol binario con factor de ramificación $b = 2$ y objetivo en profundidad 3. El objetivo es el nodo `K`.

```
              A
            /   \
          B       C
        /   \   /   \
       D     E F     G
      / \
     H   K   ← objetivo (profundidad 3)
```

IDDFS ejecutará cuatro pasadas DFS con límites 0, 1, 2 y 3.

![IDDFS iteraciones]({{ '/13_simple_search/images/12_iddfs_levels.png' | url }})

### Pasada 0 — límite = 0

Solo se puede explorar el nodo raíz. Todos los hijos superan el límite y se podan.

| Paso | Nodo | Prof. | Pila (tope→) | Explorado | Qué pasó |
|------|------|-------|-------------|-----------|----------|
| 0 | — | — | `[A]` | `{}` | Inicialización. |
| 1 | A | 0 | `[]` | `{A}` | **L1**: pop A. **L2**: no es K. **L5**: hijos B, C tienen prof 1 > 0 → **podados**. |

Resultado: FAILURE. (Se produjeron podas → hay más nodos, vale la pena ampliar el límite.)

### Pasada 1 — límite = 1

Ahora se puede llegar hasta profundidad 1: los hijos directos de A.

| Paso | Nodo | Prof. | Pila (tope→) | Explorado | Qué pasó |
|------|------|-------|-------------|-----------|----------|
| 0 | — | — | `[A]` | `{}` | Inicialización. |
| 1 | A | 0 | `[B, C]` | `{A}` | **L1**: pop A. **L5**: B (prof 1 ≤ 1), C (prof 1 ≤ 1) → push ambos. |
| 2 | C | 1 | `[B]` | `{A, C}` | **L1**: pop C. **L5**: hijos F, G tienen prof 2 > 1 → **podados**. |
| 3 | B | 1 | `[]` | `{A, C, B}` | **L1**: pop B. **L5**: hijos D, E tienen prof 2 > 1 → **podados**. |

Resultado: FAILURE. (Podas → ampliar.)

### Pasada 2 — límite = 2

Ahora llegamos hasta profundidad 2. Llegamos a D, que es el padre de K, pero K está en profundidad 3 — todavía fuera de alcance.

| Paso | Nodo | Prof. | Pila (tope→) | Explorado | Qué pasó |
|------|------|-------|-------------|-----------|----------|
| 0 | — | — | `[A]` | `{}` | Inicialización. |
| 1 | A | 0 | `[B, C]` | `{A}` | Push B (1), C (1). |
| 2 | C | 1 | `[B, F, G]` | `{A, C}` | Push F (2), G (2). |
| 3 | G | 2 | `[B, F]` | `{A, C, G}` | Hoja. Sin hijos. |
| 4 | F | 2 | `[B]` | `{A, C, G, F}` | Hoja. Sin hijos. |
| 5 | B | 1 | `[D, E]` | `{A, C, G, F, B}` | Push D (2), E (2). |
| 6 | E | 2 | `[D]` | `{..., E}` | Hoja. Sin hijos. |
| 7 | D | 2 | `[]` | `{..., D}` | **L5**: hijos H, K tienen prof 3 > 2 → **podados**. |

Resultado: FAILURE. (Podas → ampliar.)

### Pasada 3 — límite = 3

Ahora el límite alcanza la profundidad real de K. DFS encuentra K exactamente cuando llega a la rama correcta.

| Paso | Nodo | Prof. | Pila (tope→) | Explorado | Qué pasó |
|------|------|-------|-------------|-----------|----------|
| 0 | — | — | `[A]` | `{}` | Inicialización. |
| 1 | A | 0 | `[B, C]` | `{A}` | Push B (1), C (1). |
| 2 | C | 1 | `[B, F, G]` | `{A, C}` | Push F (2), G (2). |
| 3 | G | 2 | `[B, F]` | `{A, C, G}` | Hoja. |
| 4 | F | 2 | `[B]` | `{A, C, G, F}` | Hoja. |
| 5 | B | 1 | `[D, E]` | `{..., B}` | Push D (2), E (2). |
| 6 | E | 2 | `[D]` | `{..., E}` | Hoja. |
| 7 | D | 2 | `[H, K]` | `{..., D}` | **L5**: H (3 ≤ 3), K (3 ≤ 3) → push ambos. |
| 8 | K | 3 | — | — | **L2**: K **es la meta** → reconstruir camino. |

**Camino encontrado:** `A → B → D → K` (profundidad 3). ✓

Observa que IDDFS encontró K exactamente cuando el límite igualó la profundidad de la solución — igual que BFS habría hecho al llegar al nivel 3.

---

## 5. Implementación Python

`PilaConLimite` es la única pieza nueva. Todo lo demás — `busqueda_generica`, `reconstruir_camino` — es idéntico al DFS.

```python
class PilaConLimite(Frontera):
    """
    Frontera tipo pila con límite de profundidad.
    Produce DFS-con-límite cuando se usa con busqueda_generica.
    La única diferencia respecto a PilaDeFrontera: push rechaza nodos muy profundos.
    """

    def __init__(self, limite):
        self.pila = []
        self.miembros = {}     # nodo → profundidad (dict, no set — necesitamos guardar el número)
        self.limite = limite
        self.poda = False      # ¿se descartó algún nodo por superar el límite?

    def push(self, nodo, padre=None):
        # Calculamos la profundidad del nodo a partir de la profundidad del padre
        prof_padre = self.miembros.get(padre, -1) if padre is not None else -1
        prof_nodo = prof_padre + 1   # el nodo está un nivel más abajo que su padre

        if prof_nodo <= self.limite:
            self.pila.append(nodo)
            self.miembros[nodo] = prof_nodo  # registrar profundidad para calcular la de sus hijos
        else:
            self.poda = True  # hay algo más allá del límite — vale la pena ampliar en la siguiente pasada

    def pop(self):
        nodo = self.pila.pop()       # LIFO — igual que PilaDeFrontera
        # NO eliminamos de miembros: seguimos necesitando la profundidad de nodos ya procesados
        # para calcular la profundidad de sus hijos cuando los expandimos
        return nodo

    def contains(self, nodo):
        return nodo in self.miembros and nodo in self.pila   # solo si aún está en la pila activa

    def is_empty(self):
        return len(self.pila) == 0


def dfs_con_limite(problema, limite):
    """DFS que nunca expande nodos más allá de 'limite' niveles de profundidad."""
    return busqueda_generica(problema, PilaConLimite(limite))


def iddfs(problema, max_depth=1000):
    """IDDFS: DFS-con-límite en bucle con límite creciente."""
    for d in range(max_depth + 1):
        resultado = dfs_con_limite(problema, d)
        if resultado is not None:
            return resultado
    return None
```

Comparación directa de las tres fronteras:

| | `ColaDeFrontera` (BFS) | `PilaDeFrontera` (DFS) | `PilaConLimite` (IDDFS) |
|---|---|---|---|
| Estructura interna | `deque` | `list` | `list` |
| Conjunto sombra | `set` | `set` | `dict` (guarda profundidad) |
| `pop` | `popleft()` — FIFO | `pop()` — LIFO | `pop()` — LIFO |
| Diferencia clave | — | — | `push` comprueba profundidad |

---

## 6. ¿En qué se diferencia de BFS? Parecen hacer lo mismo

Esta es la pregunta más importante. BFS e IDDFS son **funcionalmente equivalentes** en lo que garantizan (ambos son completos y óptimos sin pesos), pero son muy distintos en **cómo usan la memoria**.

### BFS: guarda toda la frontera

BFS expande nivel a nivel. Para procesar el nivel $d$, tiene que guardar en la cola **todos los nodos del nivel $d-1$** hasta terminarlos, y simultáneamente va acumulando los del nivel $d$. En el peor caso, la cola contiene todo el último nivel explorado: $b^d$ nodos.

```
BFS en nivel 3 (b=2):
  Cola: [D, E, F, G]   ← todos los nodos de nivel 2 más los que se van generando
        ↑ 2² = 4 nodos en memoria a la vez
```

Para $b = 10$, $d = 10$: la cola necesita hasta $10^{10}$ nodos. Eso son miles de millones de entradas — impracticable.

### IDDFS: solo guarda un camino

IDDFS usa DFS internamente. DFS solo necesita mantener el camino desde la raíz hasta el nodo actual, más los hermanos pendientes de cada nodo en ese camino. Eso es como máximo $b \times d$ nodos — uno por profundidad, más los hermanos.

```
IDDFS en pasada 3 (b=2, límite=3):
  Pila (en el peor momento): [B, D, H, K]   ← un camino + un hermano
                               ↑ b×d = 2×3 = 6 nodos como máximo
```

Para $b = 10$, $d = 10$: la pila nunca supera $10 \times 10 = 100$ nodos. La diferencia es de **mil millones a cien**.

### La tabla de diferencias

| | BFS | IDDFS |
|---|---|---|
| **Estructura de datos** | Cola (FIFO) | Pila con límite, en bucle |
| **Memoria (frontera)** | $O(b^d)$ — todo un nivel | $O(bd)$ — un camino |
| **Nodos expandidos** | Cada nodo una vez: $O(b^d)$ | Con redundancia: $O(b^d)$ asintóticamente |
| **¿Re-expande nodos?** | No | Sí — en cada pasada |
| **Completo** | Sí | Sí |
| **Óptimo (sin pesos)** | Sí | Sí |
| **¿Cuándo es mejor?** | Grafos poco profundos, memoria abundante | Grafos profundos, memoria limitada |

---

## 7. Complejidad de tiempo y espacio

### Tiempo: $O(b^d)$ — igual que BFS

A primera vista parece que IDDFS hace mucho trabajo extra: en la pasada con límite $d$, re-explora todos los nodos de las pasadas anteriores. Hagamos las cuentas para $b = 3$ y solución a profundidad $d = 4$:

| Pasada | Nodos en esa pasada | Acumulado |
|--------|--------------------|-----------|
| 0 | $1$ | $1$ |
| 1 | $1 + 3 = 4$ | $5$ |
| 2 | $1 + 3 + 9 = 13$ | $18$ |
| 3 | $1 + 3 + 9 + 27 = 40$ | $58$ |
| 4 | $1 + 3 + 9 + 27 + 81 = 121$ | $179$ |

BFS habría expandido exactamente $1 + 3 + 9 + 27 + 81 = 121$ nodos. IDDFS expandió $179$ — solo un **48% más**.

¿Por qué el sobrecosto es tan pequeño? Porque la última pasada domina el total. Los nodos del nivel $d$ son $b^d$; todos los niveles anteriores juntos suman $b^d/(b-1)$, que es una fracción constante. En general:

$$N_{\text{IDDFS}} = \sum_{i=0}^{d} \sum_{j=0}^{i} b^j \approx \frac{b}{b-1} \cdot b^d = O(b^d)$$

**La complejidad asintótica de tiempo de IDDFS es idéntica a la de BFS.**

### Espacio: $O(bd)$ — mucho mejor que BFS

La pila de IDDFS en cualquier momento contiene a lo sumo el camino desde la raíz hasta el nodo actual, más los hermanos pendientes de cada nivel. Eso es como máximo $b$ nodos por nivel, durante $d$ niveles:

$$S_{\text{IDDFS}} = O(bd)$$

Compara con $O(b^d)$ de BFS. La diferencia en la práctica es enorme: para $b = 10$, $d = 10$:

- **BFS**: hasta $10^{10}$ nodos en la frontera.
- **IDDFS**: hasta $10 \times 10 = 100$ nodos en la pila.

![Comparación de complejidad de tiempo y espacio]({{ '/13_simple_search/images/13_complexity_comparison.png' | url }})

---

## 8. Completitud y optimalidad

### Completitud: sí

IDDFS es **completo**. Para cualquier profundidad de solución $d^{∗}$, IDDFS llegará eventualmente a la pasada con límite $d^{∗}$ y encontrará la solución, siempre que el grafo sea finito.

**¿Y si el grafo tiene ciclos?** Igual que DFS, cada pasada de IDDFS usa un conjunto `explorado` para no repetir nodos dentro de esa pasada. Los ciclos no causan bucles infinitos.

**¿Y si el grafo es infinito?** Si la solución existe a profundidad finita $d^{∗}$, IDDFS la encontrará en la pasada $d^{∗}$. Si no existe ninguna solución, IDDFS no termina — igual que BFS.

### Optimalidad: sí (sin pesos)

IDDFS es **óptimo** para grafos sin pesos: siempre encuentra el camino con menos aristas.

**¿Por qué?** En la pasada con límite $d-1$, IDDFS no encontró la solución — esto significa que no existe ninguna solución a profundidad $\leq d-1$. En la pasada con límite $d$, IDDFS encuentra la solución — y como DFS la encuentra antes de explorar profundidades mayores que $d$, la solución encontrada tiene exactamente $d$ pasos. Eso es el mínimo posible.

:::example{title="¿Qué pasa si hay pesos distintos en las aristas?"}
Al igual que BFS, IDDFS solo cuenta aristas, no pesos. Si una arista corta vale 100 y un camino largo vale 10 en total, IDDFS encontraría el camino corto (menos aristas) aunque sea el más caro. Para pesos arbitrarios se necesita una variante con costos: IDA* (Iterative Deepening A*), que usa $g(n) + h(n)$ como criterio en lugar de la profundidad.
:::

---

## 9. Aplicaciones de IDDFS

### Puzzles con profundidad desconocida

IDDFS es ideal cuando no sabes cuántos pasos necesita la solución. Por ejemplo, el **puzzle del 15** (el juego de tiles deslizables): la solución mínima puede estar entre 1 y 80 movimientos. BFS tendría que guardar en memoria todos los estados a 40 movimientos antes de llegar a 41 — un número astronómico. IDDFS va aumentando el límite de uno en uno, usando solo la memoria de un camino a la vez.

### Motores de ajedrez (antes de A*)

Los primeros motores de ajedrez usaban IDDFS para buscar el mejor movimiento. Dado que no se sabe de antemano cuántos movimientos hay que calcular, IDDFS empieza buscando a profundidad 1, luego 2, etc. — y si se acaba el tiempo, devuelve el mejor movimiento encontrado hasta ahora. Esto se llama *iterative deepening with time control*.

### Sistemas con memoria muy limitada

En dispositivos embebidos (microcontroladores, robots con memoria restringida), BFS puede ser impracticable. IDDFS ofrece las mismas garantías con una fracción del uso de memoria.

---

## 10. Resumen de IDDFS

| Propiedad | Valor | Justificación |
|---|---|---|
| Frontera | Pila con límite (LIFO + poda) | DFS que no supera profundidad $d$ |
| Tiempo | $O(b^d)$ | Mismo orden que BFS — el sobrecosto es una constante |
| Espacio | $O(bd)$ | Solo un camino raíz→nodo en memoria |
| Completo | Sí | Prueba todos los límites hasta $d^{∗}$ |
| Óptimo | Sí (sin pesos) | Nunca termina antes de agotar profundidad $d-1$ |

---

## 11. Tabla comparativa final: BFS vs DFS vs IDDFS

| Propiedad | BFS | DFS | IDDFS |
|---|---|---|---|
| **Frontera** | Cola (FIFO) | Pila (LIFO) | Pila con límite $d$, en bucle |
| **Tiempo** | $O(b^d)$ | $O(b^m)$ | $O(b^d)$ |
| **Espacio** | $O(b^d)$ | $O(bm)$ | $O(bd)$ |
| **Completo** | Sí | Sí (finito + explorado) | Sí |
| **Óptimo** | Sí (sin pesos) | No | Sí (sin pesos) |
| **Re-expande nodos** | No | No | Sí (constante por pasada) |
| **Implementación** | `popleft()` | `pop()` | bucle + `pop()` con poda |

Donde: $b$ = factor de ramificación, $d$ = profundidad de la solución, $m$ = profundidad máxima del grafo.

---

## 12. ¿Cuándo usar cada algoritmo?

La elección depende de dos preguntas: *¿necesito el camino más corto?* y *¿tengo suficiente memoria?*

```
¿Necesitas el camino MÁS CORTO?
├── No  → DFS   (más rápido en práctica, menos memoria, backtracking)
└── Sí  → ¿Tienes memoria suficiente?
          ├── Sí  → BFS   (simple, directo, no re-expande nodos)
          └── No  → IDDFS (garantías de BFS, memoria de DFS)
```

### Usa BFS cuando:

- Necesitas el **camino más corto** garantizado y la memoria no es problema.
- El grafo es **poco profundo** — si $d$ es pequeño, $O(b^d)$ es manejable.
- Quieres una implementación **simple** sin el overhead del bucle de IDDFS.
- **Ejemplos reales**: camino mínimo en un laberinto de videojuego ($d$ pequeño), grados de separación en LinkedIn, flood fill en una imagen pequeña.

### Usa DFS cuando:

- **No te importa** si el camino es el más corto — solo necesitas cualquier solución.
- El grafo es **profundo** y la solución probablemente está lejos del inicio.
- Necesitas explorar **todas** las ramas posibles (backtracking: Sudoku, N-reinas, permutaciones).
- La memoria es limitada y sabes que DFS no se perderá en ramas infinitas.
- **Ejemplos reales**: resolver un Sudoku probando valores, listar todos los archivos en un directorio, detectar ciclos en dependencias de paquetes.

### Usa IDDFS cuando:

- Necesitas el **camino más corto** (como BFS) pero la memoria es un problema real.
- **No sabes de antemano** cuán profunda está la solución — IDDFS prueba todas las profundidades.
- El **factor de ramificación es grande** — con $b$ grande, el overhead del 48% de IDDFS es insignificante frente al ahorro en memoria.
- **Ejemplos reales**: puzzles de tiles ($d$ hasta 80, $b \approx 3$), motores de juegos con control de tiempo, búsqueda en espacios de estados grandes donde BFS no cabe en RAM.

---

## 13. Preview: búsqueda informada

Los tres algoritmos que hemos visto — BFS, DFS, IDDFS — son **no informados** (*uninformed* o *blind*): exploran el espacio de estados sin ninguna pista sobre cuán cerca está la meta. Prueban todas las posibilidades dentro de su estrategia, sin importar en qué dirección está la solución.

El siguiente paso es añadir **información heurística**: una función $h(n)$ que estima cuán lejos está la meta desde el nodo $n$. Con esta guía, los algoritmos pueden evitar explorar zonas del grafo que claramente no llevan a la meta:

- **Búsqueda voraz por mejor primero** (*greedy best-first*): frontera = cola de prioridad ordenada por $h(n)$. Rápido pero no garantiza optimalidad.
- **A\***: frontera = cola de prioridad ordenada por $f(n) = g(n) + h(n)$ (costo real + heurística). Con una heurística admisible, es completo, óptimo, y en la práctica mucho más eficiente que BFS e IDDFS.
- **IDA\*** (*Iterative Deepening A\**): como IDDFS pero usando $f(n)$ como criterio de poda en lugar de la profundidad. Combina el ahorro de memoria de IDDFS con la guía heurística de A\*.

---

**Inicio:** [Volver al índice →](00_index.md)
