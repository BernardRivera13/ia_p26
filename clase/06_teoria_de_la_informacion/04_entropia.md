---
title: "Entropía: sorpresa promedio (Shannon) e información faltante (Jaynes)"
---

# Entropía: sorpresa promedio (Shannon) e información faltante (Jaynes)

Ya tenemos una forma de medir la sorpresa de un resultado específico:

$$
I(x) = -\log_2 p(x\mid I)
$$

Ahora queremos medir algo diferente:

> **Antes** de ver el resultado, ¿cuánta sorpresa espero, en promedio?

Eso es **entropía**.

---

## Definición (caso discreto)

Sea $X$ una variable aleatoria discreta con distribución $p(x\mid I)$.

La **entropía** de $X$ (en bits) es:

$$
H(X\mid I) \;=\; \mathbb{E}[I(X)] \;=\; \sum_x p(x\mid I)\,(-\log_2 p(x\mid I))
$$

Lectura:

> Entropía = sorpresa promedio de los resultados, bajo tu modelo $p(\cdot\mid I)$.

### Una aclaración crucial: “antes” vs “después” de observar

Esto evita otra confusión típica:

- **Después** de observar el resultado $X=x$, ya no hay incertidumbre sobre $X$: sabes qué ocurrió.
- La entropía $H(X\mid I)$ es una medida **ex ante**: describe cuánta incertidumbre/sorpresa esperas **antes** de mirar, cuando solo tienes $I$.

Por eso $H(X\mid I)$ se interpreta como:

- el número esperado (idealizado) de “preguntas binarias” necesarias para identificar $X$, o
- el costo promedio mínimo (en bits) para codificar los resultados de la fuente,

conectando directamente con [Bits y preguntas](02_bits_y_preguntas.md) y con [Códigos y compresión](05_codigos_y_compresion.md).

### Nota (por qué escribimos $H(X\mid I)$)

En el espíritu de Jaynes, la entropía no es una propiedad “mística” del universo: es una propiedad de **tu distribución** condicionada a lo que sabes $I$.

---

## Ejemplo acumulativo (no-uniforme, con interpretación)

Retomemos el prior de 5 palabras:

- $p(w_1)=0.60$
- $p(w_2)=0.20$
- $p(w_3)=0.10$
- $p(w_4)=0.05$
- $p(w_5)=0.05$

Calculemos $H$ (aprox):

$$
H \approx 0.60\log_2\frac{1}{0.60}
      + 0.20\log_2\frac{1}{0.20}
      + 0.10\log_2\frac{1}{0.10}
      + 0.05\log_2\frac{1}{0.05}
      + 0.05\log_2\frac{1}{0.05}
$$

No importa que el número salga “feo”: lo importante es la lectura:

- Si una opción domina (0.60), la entropía baja.
- Si las probabilidades se distribuyen más uniforme, la entropía sube.

---

## Propiedad clave 1: máximo en la uniforme

Si $X$ tiene $N$ valores posibles y es uniforme:

$$
p(x)=\frac{1}{N}
\quad\Rightarrow\quad
H(X)=\log_2 N
$$

Interpretación:

> Si todo es igual de plausible, necesitas $\log_2 N$ bits en promedio para identificar el resultado.

Esto conecta perfectamente con “bits como preguntas” de la sección 2.

---

## Propiedad clave 2: concentración reduce entropía

Si tu distribución se vuelve más “picuda” (más masa en pocas opciones), la entropía baja.

Esto no es una metáfora: es una afirmación matemática sobre la forma de la suma $\sum p\log(1/p)$.

Lectura operacional:

> Menos entropía = menos bits esperados para descubrir el valor (si haces buenas preguntas/estrategias).

![Entropía vs concentración]({{ '/06_teoria_de_la_informacion/images/entropia_concentracion.png' | url }})

*La gráfica muestra cómo $H$ cae cuando pasas de un prior uniforme a uno concentrado. Esto es la intuición “operativa” que usaremos en Wordle/password: si el prior está concentrado, adivinar es más fácil en promedio.*

---

## La perspectiva de Jaynes: “entropía = información faltante”

En el enfoque de E.T. Jaynes (máxima entropía), la entropía se interpreta como:

> **cuánta información te falta** para especificar el estado, *dado* lo que ya sabes.

Esto es útil pedagógicamente porque te obliga a decir:

- ¿Cuál es el espacio de posibilidades?
- ¿Qué restricciones (información) ya tienes?
- ¿Qué distribución estás justificando?

### Entropía de Jaynes (en el caso discreto)

En este módulo, cuando digamos “entropía de Jaynes” nos referiremos a:

1) Entropía escrita **condicionada** al contexto $I$: $H(X\mid I)$.  
2) (Más adelante) la idea de **elegir** $p$ maximizando $H$ sujeto a restricciones — “no inventar información”.

Hoy solo necesitamos (1) para el capstone; (2) lo mencionaremos como “lo que sigue”.

### (Lo que sigue) Máxima entropía en una frase

Jaynes propone un principio para elegir $p(x\mid I)$ cuando solo conoces ciertas restricciones:

> Elige la distribución que **maximiza** $H(X\mid I)$ sujeta a las restricciones, para no introducir suposiciones extra.

En este módulo no vamos a desarrollar todo el método, pero sí vamos a usar su filosofía: **siempre declarar qué sabes** y qué estás asumiendo.

---

## Analogías comunes (y por qué son incompletas)

### “Entropía = desorden”

Esto puede ayudar como intuición en física, pero aquí puede confundir.

- **Qué captura bien**: cuando hay muchas configuraciones plausibles, “hay más entropía”.
- **Qué NO captura bien**: en información, entropía no es “caos” sino **incertidumbre cuantificada** bajo un modelo.

### “Entropía = aleatoriedad”

También es peligroso si se toma literal.

- Un proceso puede ser determinista para el mundo, pero si tú no lo conoces, tu $p(\cdot\mid I)$ puede tener alta entropía.
- Por eso Jaynes insiste en separar “estado del mundo” vs “estado de conocimiento”.

---

## Entropía y Wordle: la idea de “ganancia esperada”

Antes de jugar un guess $g$, tienes una distribución sobre posibles palabras $p(x\mid I)$.

Después del feedback $F$, tu distribución cambia a $p(x\mid F,I)$.

La cantidad de incertidumbre restante (en bits) después de ver $F$ es:

$$
H(X\mid F, I)
$$

Como $F$ es aleatorio (depende del secreto), la incertidumbre restante esperada es:

$$
\mathbb{E}_{F}[H(X\mid F,I)]
$$

Entonces la **ganancia esperada de información** del guess $g$ es:

$$
\text{IG}(g) \;=\; H(X\mid I) - \mathbb{E}_{F}[H(X\mid F,I)]
$$

Esta fórmula será el corazón del solver del capstone.

![Top guesses por ganancia esperada]({{ '/06_teoria_de_la_informacion/images/wordle_top_info_gain.png' | url }})

*Ejemplo (a escala pequeña): guesses que “parten” mejor el espacio de candidatas maximizando $\text{IG}(g)$. La lista exacta depende del lexicón y del prior.*

![Wordle: distribución de patrones (por guess)]({{ '/06_teoria_de_la_informacion/images/wordle_pattern_mass.png' | url }})

*Otra vista complementaria: un buen guess induce muchos patrones posibles con masas no demasiado concentradas. Si casi toda la masa cae en 1–2 patrones, el guess “dice poco” en promedio.*

---

:::exercise{title="Entropía como costo promedio de sorpresa" difficulty="3"}

Considera dos priors sobre 4 palabras:

- Prior A: $[0.25, 0.25, 0.25, 0.25]$
- Prior B: $[0.70, 0.10, 0.10, 0.10]$

1. Calcula $H_A$ y $H_B$.
2. Interpreta: ¿en cuál caso te conviene más “apostar” por la palabra más probable sin preguntar nada?
3. Relación con passwords: ¿qué prior se parece más a la realidad humana al escoger contraseñas?

:::

---

**Siguiente:** [Códigos y compresión →](05_codigos_y_compresion.md)  
**Volver:** [← Índice](00_index.md)

