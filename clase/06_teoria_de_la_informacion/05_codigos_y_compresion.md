---
title: "Códigos y compresión: pagar menos por lo frecuente"
---

# Códigos y compresión: pagar menos por lo frecuente

Ya tenemos una medida de “incertidumbre” en bits: $H(X\mid I)$.

Ahora viene una pregunta natural:

> Si $H$ es “bits esperados”, ¿eso se puede convertir en un método concreto para **codificar** mensajes?

Sí. Y este puente es una de las razones por las que teoría de la información es tan central.

---

## Problema: enviar símbolos con un alfabeto binario

Supón que una fuente emite símbolos (por ejemplo, palabras o tipos de eventos) con distribución $p(x\mid I)$.

Queremos asignar a cada símbolo $x$ un **código binario** $c(x)$ (una cadena de 0/1) para transmitirlo.

### Restricción esencial: decodificación sin ambigüedad

Una restricción práctica muy usada es **código prefijo**:

> Ningún código es prefijo de otro.

Eso permite decodificar leyendo bits de izquierda a derecha sin separadores.

---

## El principio: lo frecuente debe ser corto

Si un símbolo aparece mucho, conviene darle un código corto.

Esto no es solo “intuición”: si no lo haces, tu longitud promedio se dispara.

Definimos la **longitud** del código:

$$
\ell(x) = |c(x)|
$$

Y la longitud promedio:

$$
L = \mathbb{E}[\ell(X)] = \sum_x p(x\mid I)\,\ell(x)
$$

La pregunta es: ¿cuál es el menor $L$ que puedo lograr?

---

## La conexión con entropía (idea central)

Sin entrar en una prueba completa, el resultado (Shannon) dice:

> Para una fuente discreta, la longitud promedio óptima está acotada por la entropía:

$$
H(X\mid I) \le L < H(X\mid I) + 1
$$

![Longitud ideal vs probabilidad]({{ '/06_teoria_de_la_informacion/images/ideal_length_vs_prob.png' | url }})

*Visualización concreta de la idea: longitudes ideales se parecen a $-\log_2 p(x)$, pero los códigos reales suelen usar longitudes enteras. Aun así, el costo promedio puede acercarse al límite impuesto por $H(X\mid I)$.*

Lectura:

- **No puedes** comprimir, en promedio, por debajo de $H$ bits por símbolo (sin perder información).
- Puedes acercarte a $H$ con códigos bien diseñados (por ejemplo, Huffman).

Esto hace que $H$ sea más que una “fórmula bonita”: es un **límite de compresión**.

---

## Ejemplo acumulativo: “candidatas de palabra” como mensajes

Volvamos a una fuente que produce “palabras candidatas” con un prior sesgado.

Supón 4 símbolos:

| símbolo | $p(x)$ |
|---|---:|
| $w_1$ | 0.50 |
| $w_2$ | 0.25 |
| $w_3$ | 0.15 |
| $w_4$ | 0.10 |

Un código prefijo posible (no necesariamente óptimo) es:

| símbolo | código | $\ell(x)$ |
|---|---|---:|
| $w_1$ | 0 | 1 |
| $w_2$ | 10 | 2 |
| $w_3$ | 110 | 3 |
| $w_4$ | 111 | 3 |

Longitud promedio:

$$
L = 0.50\cdot 1 + 0.25\cdot 2 + 0.15\cdot 3 + 0.10\cdot 3
$$

Y puedes compararlo con $H(X)$ calculando:

$$
H(X)=\sum_i p(w_i)\log_2\frac{1}{p(w_i)}
$$

La idea UX aquí, si lo enseñamos en clase, es ver ambas cosas lado a lado:

- **Entropía**: “bits inevitables”
- **Longitud promedio**: “bits que tu diseño realmente paga”

El laboratorio (`lab_informacion.py`) va a generar gráficas donde se vea cómo cambia $L$ cuando cambias el prior.

---

## Analogía (útil pero incompleta): “precio por palabra”

Analogía:

> Cada bit cuesta dinero/tiempo. Si algo ocurre mucho, quieres pagarlo barato.

- **Qué captura bien**: el criterio de optimalidad promedio.
- **Qué es incompleto**: no dice nada sobre *cómo* construir el código, ni sobre errores, ni sobre canales con ruido.

---

:::exercise{title="Diseño de códigos y longitud promedio" difficulty="3"}

Tienes símbolos $\{a,b,c,d\}$ con:

- $p(a)=0.5$, $p(b)=0.25$, $p(c)=0.125$, $p(d)=0.125$

1. Propón un código prefijo (por ejemplo, asignando códigos cortos a los más probables).
2. Calcula $L$.
3. Calcula $H(X)$.
4. Compara $L$ contra $H$. ¿Qué tan “cerca” estás del límite?

:::

---

**Siguiente:** [Cross-entropy y KL →](06_cross_entropy_y_kl.md)  
**Volver:** [← Índice](00_index.md)

