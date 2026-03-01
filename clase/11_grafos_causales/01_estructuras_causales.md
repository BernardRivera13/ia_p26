---
title: "Estructuras Causales"
---

# Estructuras Causales

> *"No amount of experimentation can ever prove me right; a single experiment can prove me wrong."*
> — Albert Einstein

---

## La Paradoja de Simpson

En 1973, la Universidad de California en Berkeley fue acusada de discriminación contra mujeres en sus admisiones a posgrado. Los números agregados parecían claros:

| | Solicitantes | Admitidos | Tasa |
|---|---:|---:|---:|
| **Hombres** | 8,442 | 3,714 | **44%** |
| **Mujeres** | 4,321 | 1,512 | **35%** |

La diferencia es notable: los hombres fueron admitidos a una tasa 9 puntos porcentuales mayor. ¿Discriminación?

Pero al desglosar por **departamento**, la historia cambia completamente:

| Departamento | Hombres (admitidos) | Mujeres (admitidas) |
|:---:|:---:|:---:|
| A | 62% | **82%** |
| B | 63% | **68%** |
| C | 37% | 34% |
| D | 33% | **35%** |
| E | 28% | 24% |
| F | 6% | **7%** |

Departamento por departamento, las mujeres tenían tasas de admisión **iguales o superiores**. ¿Cómo es posible que los datos agregados digan lo contrario?

![Paradoja de Simpson: Berkeley 1973]({{ '/11_grafos_causales/images/simpson_paradox.png' | url }})

La respuesta: **las mujeres se postularon en mayor proporción a departamentos más competitivos** (con tasas de admisión bajas para todos). El departamento al que aplicas actúa como una variable intermedia que distorsiona la comparación agregada.

```mermaid
graph TD
    G(("Género")) --> D(("Departamento"))
    D --> A(("Admisión"))
    G -.->|"¿efecto directo?"| A
```

El **Departamento** es un *confounder* (variable de confusión): influye tanto en la composición de los solicitantes como en la tasa de admisión. Si no lo tomamos en cuenta, llegamos a una conclusión incorrecta.

Esta paradoja no es un caso aislado. Aparece en:

- **Ensayos clínicos:** un tratamiento parece peor en el agregado pero mejor dentro de cada subgrupo
- **Eficacia de vacunas:** tasas de hospitalización que se invierten al estratificar por edad
- **Estadísticas deportivas:** un bateador con peor promedio general pero mejor en cada temporada individual

En todos los casos, la estructura es la misma: **una variable oculta invierte la conclusión cuando se ignora**. Para resolver esta paradoja, necesitamos un lenguaje formal para hablar de causa y efecto.

---

## Grafos causales como lenguaje

Un **grafo causal** es un grafo dirigido acíclico (DAG) donde:

- Cada **nodo** representa una variable
- Cada **flecha** $X \rightarrow Y$ significa: "$X$ es causa directa de $Y$"

Esto es diferente de decir "$X$ e $Y$ están correlacionados". La dirección importa: una flecha es una **afirmación sobre el mecanismo del mundo**, no solo un patrón en los datos.

:::example{title="Correlación vs. Causalidad"}
Estos dos grafos producen la misma correlación entre $X$ e $Y$, pero cuentan historias causales completamente distintas:

**Grafo A:** $X \rightarrow Y$ — "X causa Y"

**Grafo B:** $X \leftarrow Z \rightarrow Y$ — "X e Y están correlacionados, pero ninguno causa al otro"

Los datos solos no distinguen entre A y B. Necesitas **conocimiento del dominio** para decidir cuál es correcto.
:::

Para razonar sobre causalidad con grafos, necesitamos entender tres patrones fundamentales.

---

## Las tres estructuras fundamentales

Todo grafo causal, por complejo que sea, se compone de combinaciones de **tres estructuras básicas** de tres nodos. Cada una tiene un comportamiento distinto respecto a las correlaciones que genera.

### 1. Fork (bifurcación) — Confounding

```mermaid
graph TD
    Z(("Z")) --> X(("X"))
    Z --> Y(("Y"))
```

Una **causa común** $Z$ genera correlación entre $X$ e $Y$, aunque $X$ no cause $Y$ ni viceversa.

:::example{title="Fork: helado y ahogamientos"}
Cada verano, las ventas de helado y los ahogamientos en piscinas **suben juntos**. ¿El helado causa ahogamientos?

```mermaid
graph TD
    V(("Verano<br/>(calor)")) --> H(("Ventas de<br/>helado"))
    V --> A(("Ahogamientos"))
```

No. El **verano** (calor) es la causa común: la gente come más helado Y nada más. Si controlas por la temporada (solo miras datos de julio), la correlación entre helado y ahogamientos **desaparece**.

$Z$ es un **confounder** (variable de confusión): crea una correlación espuria entre $X$ e $Y$.
:::

**Regla del fork:** $X$ e $Y$ están correlacionados. Pero si condicionamos en $Z$ (fijamos su valor), la correlación **desaparece**:

$$X \not\perp Y \quad \text{pero} \quad X \perp Y \mid Z$$

**Intuición:** la correlación entre $X$ e $Y$ existe *solo* porque comparten la causa $Z$. Una vez que controlas por $Z$, no queda nada.

---

### 2. Chain (cadena) — Mediación

```mermaid
graph LR
    X(("X")) --> M(("M")) --> Y(("Y"))
```

$X$ causa $Y$, pero **a través de** un intermediario $M$.

:::example{title="Chain: fumar y cáncer"}
```mermaid
graph LR
    F(("Fumar")) --> A(("Alquitrán<br/>en pulmones")) --> C(("Cáncer<br/>pulmonar"))
```

Fumar causa cáncer, pero el mecanismo pasa por la acumulación de alquitrán. Si pudieras fijar el nivel de alquitrán (hipotéticamente), fumar ya no predeciría cáncer.
:::

**Regla de la cadena:** $X$ e $Y$ están correlacionados (porque $X$ causa $Y$ vía $M$). Pero si condicionamos en $M$, la correlación **se bloquea**:

$$X \not\perp Y \quad \text{pero} \quad X \perp Y \mid M$$

**Intuición:** $M$ es el *canal* por el que fluye la influencia de $X$ a $Y$. Si fijas $M$, cortas el canal.

---

### 3. Collider (colisionador) — Selection Bias

```mermaid
graph TD
    X(("X")) --> C(("C"))
    Y(("Y")) --> C
```

Dos causas independientes $X$ e $Y$ comparten un **efecto común** $C$.

:::example{title="Collider: Hollywood"}
```mermaid
graph TD
    T(("Talento")) --> E(("Éxito en<br/>Hollywood"))
    B(("Belleza")) --> E
```

En la **población general**, talento y belleza son independientes: una persona puede ser talentosa sin ser guapa y viceversa.

Pero si solo miras **actores exitosos en Hollywood** (condicionas en $C =$ éxito), parece que los más talentosos son menos guapos y viceversa. ¿Por qué?

Porque para tener éxito necesitas talento **o** belleza (o ambos). Si alguien exitoso no es guapo, *probablemente* es muy talentoso (y viceversa). Condicionar en el efecto común **crea una correlación que no existía**.

Esto se conoce como **selection bias** (sesgo de selección) o **explaining away**.
:::

**Regla del collider:** $X$ e $Y$ son independientes. Pero si condicionamos en $C$, **se crea** una correlación:

$$X \perp Y \quad \text{pero} \quad X \not\perp Y \mid C$$

**Intuición:** el collider es lo opuesto al fork. El fork crea correlación que se destruye al condicionar. El collider destruye independencia al condicionar.

---

## Resumen de las tres estructuras

| Estructura | Grafo | Sin condicionar | Condicionando en el nodo central |
|---|---|:---:|:---:|
| **Fork** | $X \leftarrow Z \rightarrow Y$ | Correlacionados | **Independientes** |
| **Chain** | $X \rightarrow M \rightarrow Y$ | Correlacionados | **Independientes** |
| **Collider** | $X \rightarrow C \leftarrow Y$ | **Independientes** | Correlacionados |

**Patrón clave:** En forks y chains, observar el nodo central **bloquea** el flujo de información. En colliders, observar el nodo central **abre** el flujo de información.

:::exercise{title="¿Qué estructura es?"}

Para cada situación, identifica si es un **fork**, **chain** o **collider**. Dibuja el grafo.

1. **Ejercicio causa sudoración.** Sudoración causa deshidratación. ¿Qué relación hay entre ejercicio y deshidratación?

2. **La lluvia causa tráfico.** La lluvia causa paraguas mojados. ¿Los paraguas mojados predicen tráfico?

3. **Habilidad atlética mejora la nota de admisión.** Buen expediente académico mejora la nota de admisión. ¿Qué pasa si solo observas a los alumnos admitidos?

4. **Presupuesto de marketing aumenta ventas.** Ventas altas aumentan el precio de la acción. ¿Marketing está correlacionado con el precio de la acción?

5. **Fumar causa tos.** Un resfriado causa tos. ¿Los fumadores se resfrían más?
:::

<details>
<summary><strong>Ver Respuestas</strong></summary>

1. **Chain:** Ejercicio → Sudoración → Deshidratación. Ejercicio causa deshidratación a través de la sudoración. Si controlas por sudoración, ejercicio y deshidratación se vuelven independientes.

2. **Fork:** Lluvia → Tráfico, Lluvia → Paraguas mojados. La lluvia es la causa común. Si sabes que llovió, saber que hay paraguas mojados no te dice nada nuevo sobre el tráfico.

3. **Collider:** Atletismo → Admisión ← Expediente. Entre la población general, atletismo y expediente son independientes. Pero entre los admitidos, parecen negativamente correlacionados (si no es atleta, probablemente tiene buen expediente).

4. **Chain:** Marketing → Ventas → Precio de acción. Marketing causa precio a través de ventas. Si controlas por ventas, marketing y precio se vuelven independientes.

5. **Collider:** Fumar → Tos ← Resfriado. Fumar y resfriarse son independientes. Pero si sabes que alguien tose y no fuma, es más probable que tenga resfriado.

</details>

---

**Anterior:** [Grafos Causales — Índice](00_index.md) | **Siguiente:** [Causalidad y el Operador do →](02_do_y_causalidad.md)
