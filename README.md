# T13 · ¿Aplicamos fungicida contra la roya de la soja?

Hackatón IIA · UADE · 3.4.257

**Integrantes:** Juan Pedro Moldes · Facundo Iturbe · **IA declarada:** Claude · **Pista:** Reglas · **Publicación en LinkedIn:** (se agrega el link al publicar)

## 1. Formulación y PEAS

**La pregunta.** Para cada lote de soja en una semana, ¿la roya asiática causa pérdidas si no se aplica fungicida? **SÍ** significa: la roya causa pérdidas si no se aplica, entonces hay que aplicar. **NO** significa: no hace falta aplicar esa semana.

**Cuándo se decide.** Al cierre de cada semana de monitoreo, con los datos ya registrados de esa semana (horas de mojado foliar del sensor o la estación, temperatura media, y severidad relevada en lotes vecinos). El sistema no usa nada que se conozca después de decidir.

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | Que los lotes que iban a tener pérdidas por roya se traten a tiempo (sensibilidad), sin aplicar en lotes que no lo necesitaban (especificidad). De fondo: rendimiento protegido contra costo de producto e impacto ambiental. |
| Entorno | Lotes de soja durante la campaña, semana a semana. Clima (lluvia, rocío, temperatura) y presión de esporas desde lotes vecinos. Parcialmente observable, estocástico y secuencial. |
| Actuadores | La recomendación al asesor agronómico: "aplicar fungicida en este lote" o "no aplicar esta semana" (orden de pulverización). |
| Sensores | Sensor de mojado foliar o estación meteorológica (horas de mojado, temperatura media) y monitoreo de lotes vecinos (% de hojas afectadas). |

**Objetivo y métrica.** El **objetivo** es proteger el rendimiento de la soja frente a la roya aplicando fungicida solo cuando hace falta: el menor costo total entre pérdidas de cosecha, producto y daño ambiental. La **métrica principal** es la **sensibilidad** (de los lotes que iban a tener pérdidas, cuántos detectamos), porque el falso negativo es el error caro. La controlamos con la **especificidad**. **No son lo mismo:** se puede lograr 100% de sensibilidad aplicando fungicida en todos los lotes, y eso no cumple el objetivo porque gasta producto y daña el ambiente en todos los lotes sanos. Por eso la sensibilidad sola no alcanza y siempre la miramos junto con la especificidad y contra la línea base.

## 2. El sistema

**Cómo decide.** Sistema de reglas si‑entonces:

- **R1.** Si las **horas de mojado foliar en la semana son 20 o más** → **SÍ** (aplicar fungicida).
- **R2.** Si no → **NO** (no aplicar esta semana).

**Variables que usa y por qué.**

- **Usa: horas de mojado foliar.** El hongo necesita la hoja mojada para infectar. En los 14 casos de entrenamiento, todos los lotes con pérdidas tuvieron 28 h o más de mojado, y todos los lotes sin pérdidas salvo uno (caso 11) tuvieron 19 h o menos. Es la única variable que separa los casos.
- **Descarta: temperatura media.** En entrenamiento hay lotes con y sin pérdidas en todo el rango (de 15,4 °C a 28,6 °C, mezclados). No separa los casos. **También la probamos combinada con el mojado.** Entre los lotes de entrenamiento con 20 h o más, la mejor combinación es "mojado ≥ 20 h y temperatura ≥ 22,7 °C". Esa regla saca el falso positivo del caso 11 (22,5 °C), pero deja afuera al caso 5 (15,4 °C, con pérdidas): cambia el error barato por el caro. Solo combinando las tres variables se llega a 14/14 (bitácora, fila 10), y eso es ajustarse a un solo caso.
- **Descarta: severidad en lotes vecinos.** En entrenamiento no ayudó: el caso 6 tuvo pérdidas con 3% en los vecinos y el caso 7 no tuvo pérdidas con 22%. Agregarla como condición (vecinos ≥ 5%) sacaba el falso positivo del caso 11 pero creaba un falso negativo en el caso 6 (ver sección 4).
- **Dónde va el umbral.** Entre 19 h (el lote sin pérdidas con más mojado, salvo el caso 11) y 28 h (el lote con pérdidas con menos mojado) no hay ningún caso de entrenamiento. Pusimos el corte en el borde inferior de ese hueco, **20 h**, porque el error caro es no aplicar: ante un mojado intermedio, preferimos aplicar.

Las tres variables se conocen al momento de decidir (se registran durante la semana que se evalúa).

**Cómo se corre en vivo.**

1. Del caso nuevo, mirar solo la columna **Horas de mojado foliar (en la semana)**.
2. Si es **20 o más** → respuesta **SÍ** (aplicar).
3. Si es **menos de 20** → respuesta **NO** (no aplicar).

Se puede hacer a mano. En una planilla, con las horas de mojado en la celda C2: `=SI(C2>=20;"SÍ";"NO")`.

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | Mojado 45 h · Temperatura 24 °C · Vecinos 10% | SÍ (45 ≥ 20) |
| B | Mojado 12 h · Temperatura 22 °C · Vecinos 15% | NO (12 < 20) |

## 3. Medición

Reglas aplicadas a los datos de la ficha (`datos/T13.csv`), sin cambiar la división: casos 1 a 14 para construir, casos 15 a 20 solo para medir.

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| Real SÍ | VP = 7 | FN = 0 |
| Real NO | FP = 1 | VN = 6 |

Total: 7 + 0 + 1 + 6 = 14 casos. Reales SÍ = 7 (casos 1, 3, 5, 6, 8, 13, 14). Reales NO = 7. El único error es el caso 11 (69 h de mojado, sin pérdidas): falso positivo.

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| Real SÍ | VP = 2 | FN = 1 |
| Real NO | FP = 1 | VN = 2 |

Total: 2 + 1 + 1 + 2 = 6 casos. Reales SÍ = 3 (casos 16, 18, 19). Reales NO = 3 (casos 15, 17, 20). Errores: caso 17 (21 h, sin pérdidas) falso positivo; caso 18 (8 h, con pérdidas) falso negativo.

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | 13/14 = 92,9% | 4/6 = 66,7% |
| Precisión | 7/8 = 87,5% | 2/3 = 66,7% |
| Sensibilidad | 7/7 = 100% | 2/3 = 66,7% |
| Especificidad | 6/7 = 85,7% | 2/3 = 66,7% |

**Entrenamiento contra prueba.** Sí hay diferencia: la exactitud baja de 92,9% a 66,7% y la sensibilidad de 100% a 66,7%. Hay dos razones:

- **Pocos casos.** Con 6 casos de prueba, cada error mueve 16,7 puntos. La diferencia son dos casos, así que los números de prueba tienen mucha incertidumbre.
- **Patrones que el entrenamiento no tenía.** El caso 17 (21 h) cae justo en la zona de 20 a 27 h donde no había ningún caso de entrenamiento. Ahí el umbral era una apuesta, y la decisión de aplicar ante la duda nos dio un falso positivo. El caso 18 (8 h de mojado pero 28% de severidad en los vecinos) muestra un patrón que no aparece en entrenamiento: mucha presión de esporas con poco mojado. Nuestra regla no lo puede ver.

No es memorización: es una sola regla con un solo umbral. Es generalización limitada por tener 14 casos. **No cambiamos la regla después de ver la prueba**, porque eso sería usar los casos de prueba para construir y la medición dejaría de valer.

## 4. El error más caro

**El falso negativo es peor.** La ficha T13 (`problematicas/tarde.md`, apartado "Si el sistema se equivoca") dice textualmente:

> - Falso positivo (dice SÍ y era NO): Aplicar sin necesidad: costo del producto e impacto ambiental.
> - Falso negativo (dice NO y era SÍ): No aplicar y que haya epidemia: pérdidas de rendimiento de hasta un tercio.

Perder hasta un tercio de la cosecha cuesta mucho más que una aplicación. Aun sin mirar la cifra, la comparación da lo mismo: el FN es una epidemia en el lote (cosecha perdida) y el FP es el costo de una aplicación más su impacto ambiental.

**Qué cambiamos por él.** Partimos de una primera versión con 13/14 en entrenamiento y la cambiamos:

| Versión | Regla | Entrenamiento | Error en entrenamiento |
|---|---|---|---|
| V1 (primera versión) | SÍ si mojado ≥ 28 h **y** vecinos ≥ 5% | 13/14 | 1 FN (caso 6) |
| **V2 (final)** | SÍ si mojado ≥ 20 h | 13/14 | 1 FP (caso 11) |

- **Sacamos la condición de vecinos ≥ 5%.** Eliminaba el falso positivo del caso 11, pero a cambio creaba un falso negativo en el caso 6. Con la misma exactitud, cambiamos el error caro por el barato. La exactitud sola no distinguía las dos versiones: el costo sí.
- **Bajamos el umbral de 28 h a 20 h,** al borde inferior del hueco sin datos, para que un lote con mojado intermedio se trate.

En entrenamiento, los falsos negativos pasaron de 1 a 0. Elegimos V2 antes de mirar la prueba. En prueba, las dos versiones tienen la misma sensibilidad (2/3, las dos fallan el caso 18). V1 no habría tenido el falso positivo del caso 17. No volvimos a V1 por ese resultado, porque elegir la versión según la prueba es usar la prueba para construir.

**Cuántos falsos positivos aceptamos por cada falso negativo evitado.** La ficha no da el costo por hectárea de una aplicación ni de una epidemia, así que no podemos fijar una tasa exacta. Lo que decidimos, con números de entrenamiento:

- **De V1 a V2 aceptamos 1 FP para evitar 1 FN.** Es una tasa de 1 a 1, y la aceptamos porque la ficha deja claro que el FN es mucho más caro.
- **No aceptamos "aplicar siempre".** En entrenamiento suma 6 FP más que nuestra regla (7 contra 1) sin evitar ningún FN, porque ya teníamos 0. Ahí es donde entra el costo del producto y del ambiente: es lo que nos hace rechazar aplicar en todos los lotes.
- **Regla de decisión:** aceptamos un falso positivo más solo si evita un falso negativo en entrenamiento.
- **Dónde entra el costo del producto en la regla: en el umbral.** El costo total en entrenamiento es C_FN × FN + C_FP × FP, donde C_FP es el costo del producto y el ambiente por aplicación de más. Con un umbral entre 19 y 28 h queda FN = 0 y FP = 1, que es el mínimo para cualquier relación de costos:
  - **Si bajamos el umbral a 19 h,** se suma un FP (caso 10, 19 h): se paga una aplicación más (C_FP) sin evitar ningún FN.
  - **Si lo subimos por encima de 28 h,** se suma un FN (caso 14, 28 h).
  - **El costo del producto es lo que frena el umbral en 20 h y no más abajo.** Dentro del hueco entre 19 y 28 h, como C_FN es mayor que C_FP, elegimos el borde que aplica.
- **El límite, en prueba:** pasar de nuestra regla a "aplicar siempre" habría evitado 1 FN (caso 18) a cambio de 2 FP (casos 15 y 20). Si una epidemia cuesta más que dos aplicaciones, convendría aplicar en más lotes. Esa cuenta se tiene que hacer con costos reales antes de usar el sistema en el campo.

## 5. ¿Comparado contra qué?

**Línea base trivial.**

| Sistema | Exactitud entrenamiento | Exactitud prueba | Sensibilidad prueba | Especificidad prueba |
|---|---|---|---|---|
| Siempre NO (nunca aplicar) | 7/14 = 50% | 3/6 = 50% | 0% | 100% |
| Siempre SÍ (aplicar siempre) | 7/14 = 50% | 3/6 = 50% | 100% | 0% |
| **Nuestra regla** | 13/14 = 92,9% | 4/6 = 66,7% | 66,7% | 66,7% |

- La regla le gana a las dos líneas base en exactitud: 92,9% contra 50% en entrenamiento y 66,7% contra 50% en prueba.
- En prueba la ventaja es de un solo caso de seis. Es una mejora real pero con poca evidencia.
- "Aplicar siempre" tiene 100% de sensibilidad. Si solo miráramos la sensibilidad, ganaría el sistema trivial, que fumiga todos los lotes. Por eso la sensibilidad se lee junto con la especificidad (0% en la línea base).
- **En prueba, "aplicar siempre" nos gana en sensibilidad** (100% contra 66,7%). Con nuestra regla se escapó 1 lote con pérdidas (caso 18) y a cambio se evitaron 2 aplicaciones innecesarias (casos 15 y 20). Si ese intercambio conviene depende del costo por hectárea de la aplicación y de la pérdida, que la ficha no da.
- Las clases están balanceadas (7 y 7, 3 y 3), así que el 50% de la línea base es el piso real.

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | Los 20 casos inventados de la ficha T13: 14 de entrenamiento y 6 de prueba, con la división fija. Son pocos, no son datos reales de campo y no sabemos de qué campaña o región representan. |
| ¿Qué métrica? | Sensibilidad como principal (el falso negativo es el error caro), junto con especificidad y precisión. La exactitud la reportamos comparada contra la línea base trivial. |
| ¿Comparado contra qué? | Contra las dos líneas base triviales: "siempre SÍ" y "siempre NO" (50% de exactitud cada una). |
| ¿Quién lo midió? | Nosotros mismos, el mismo grupo que construyó la regla, sobre los 6 casos de prueba que no usamos para construir. No hay una medición independiente con datos nuevos de campo; la mesa lo prueba con casos propios de la cátedra. |

## 6. Riesgo: el camino completo

**Riesgo principal: no aplicar cuando hay mucha presión de esporas.**

- **Datos:** la regla solo mira el mojado foliar. Además, un sensor de mojado mal ubicado o descalibrado puede registrar menos horas de las reales.
- **Modelo:** la regla ignora la severidad en lotes vecinos, porque con 14 casos no encontramos que ayudara. En prueba apareció el caso 18: 8 h de mojado, 28% de severidad en vecinos, y sí hubo pérdidas.
- **Decisión:** el sistema dice NO aplicar en un lote rodeado de roya.
- **Consecuencia:** epidemia en el lote y pérdidas de rendimiento de hasta un tercio, justo el error caro.

**Riesgo del otro lado.** Con el umbral bajo (20 h), en una campaña muy lluviosa el sistema puede recomendar aplicar en muchos lotes. Eso implica más costo, más impacto ambiental y aplicaciones repetidas que favorecen que el hongo se haga resistente al fungicida.

**Mitigación.** La salida es una recomendación para el asesor agronómico, que decide.

- **Si el sistema dice NO con severidad alta en lotes vecinos:** el asesor debería revisar el lote igual.
- **Si en una campaña lluviosa el sistema pide aplicar en muchos lotes:** antes de pulverizar, el asesor revisa en el campo los lotes marcados SÍ, mirando las hojas en busca de pústulas de roya. Empieza por los que tienen entre 20 y 28 h de mojado, que es la zona donde el entrenamiento no tenía casos y la regla es menos confiable. Cada lote revisado se anota con su resultado (hubo o no pérdidas) para recalibrar el umbral con datos propios.
- **Para incorporar la severidad vecina a la regla:** harían falta más casos de entrenamiento.

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | Que leyera el repositorio (bases, guías, plantilla y fichas) y nos resumiera qué pide el hackatón, la rúbrica y los topes. | El resumen de la rúbrica, los topes y las pruebas obligatorias. | Nada. |
| 2 | Que nos explicara qué es la "pista" y cuál convenía para nuestra ficha. | La pista Reglas: se corre a mano en la mesa y se explica sin IA. | Por ahora, nada: dejamos la comparación con la neurona para después (fila 7). |
| 3 | Que analizara solo los casos 1 a 14 y propusiera reglas. | Usar solo las horas de mojado foliar, y descartar la temperatura porque no separa los casos. | La versión V1 (mojado ≥ 28 h y vecinos ≥ 5%): tenía la misma exactitud (13/14), pero su error era un falso negativo, que es el error caro. |
| 4 | Dónde poner el umbral dentro del hueco de 19 a 28 h sin datos. | 20 h, en el borde inferior, porque ante la duda preferimos aplicar. | El punto medio (24 h): es neutral entre los dos errores, y en nuestro problema no lo son. |
| 5 | Que calculara las matrices y las métricas de entrenamiento y de prueba, y las líneas base. | Las matrices y métricas, después de verificarlas: Facundo las recalculó por separado con su IA desde `datos/T13.csv`, sin ver nuestros números, y coincidieron caso por caso (mismos errores: 11 en entrenamiento, 17 y 18 en prueba). | Nada. |
| 6 | Después de medir la prueba, que analizara los errores. La IA marcó que una regla extra (vecinos ≥ 20% → SÍ) habría atrapado el caso 18. | Dejar el caso 18 como riesgo documentado (sección 6). | Agregar esa regla: sería construir con los casos de prueba. Además, en entrenamiento esa regla sumaba un falso positivo (caso 7) sin acertar ningún caso nuevo. |
| 7 | Ya medida la regla, le pedimos probar la pista más avanzada: una neurona (perceptrón) con las 3 variables, entrenada con los casos 1 a 14. Dio 14/14 en entrenamiento y 5/6 en prueba. Pesos aprendidos (regla del perceptrón, pesos iniciales en 0, tasa 1, entradas divididas por 10, 196 pasadas): **SÍ si 0,58 · mojado (h) + 1,401 · temperatura (°C) + 1,85 · vecinos (%) − 79 ≥ 0**. | Quedarnos con la regla y documentar la prueba de la neurona acá. | Cambiar a la neurona. Ya habíamos visto la prueba de la regla, y cambiar de sistema porque la neurona saca mejor en prueba sería elegir con los casos de prueba. Además, su 14/14 sale de darle peso a la temperatura para separar un solo caso (el 11) que contradice a otro casi igual (el 6): con esos pesos el caso 6 da +0,10 y el 11 da −0,06. Es un margen mínimo, un ajuste a un caso y no un patrón. |
| 8 | Que redactara el README con nuestras decisiones. | La estructura y la redacción, revisada por nosotros. | Cualquier frase dirigida al evaluador: el README solo documenta el trabajo. |
| 9 | Facundo le pidió a su IA 10 preguntas de defensa sobre el README, con respuestas para corregir. | 9 de las 10 respuestas. | La respuesta 6: citaba una versión con temperatura que no es la nuestra (nuestra V1 es mojado ≥ 28 h y vecinos ≥ 5%). Se había escrito sobre otro borrador. |
| 10 | Facundo le pidió a su IA revisar el README contra el CSV. | Tres correcciones. V1 no era "de máxima exactitud": la regla "mojado ≥ 28 y (vecinos ≥ 5% o temperatura ≥ 25 °C)" saca 14/14, así que la renombramos "primera versión". Agregamos en la sección 5 que en prueba "aplicar siempre" nos gana en sensibilidad. Corregimos el rango de temperatura (15,4 a 28,6 °C). | No usar esa regla de 14/14. Con dos condiciones encadenadas para acertar un solo caso (el 11), es el mismo ajuste a un caso que la neurona. |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | 9,5/10 (A 2 · B 1,5 · C 3 · D 1 · E 1 · F 0,5 · G 0,5). Sin topes. Habilitados para la mesa. | **Agujero principal:** decíamos que el objetivo es el menor costo total, pero no explicábamos cómo entra el costo del producto y del ambiente ni cuántos FP aceptamos por cada FN evitado. **Otros:** si probamos la temperatura combinada con el mojado o solo sola, y cómo verificamos la recomendación en una campaña lluviosa. Además, la fila de corridas estaba sin completar. | Sección 4: agregamos la tasa FP por FN (1 a 1 de V1 a V2; rechazo de "aplicar siempre", que suma 6 FP sin evitar FN) y el límite de esa decisión en prueba. Sección 2: mostramos la temperatura combinada con el mojado ("mojado ≥ 20 y temperatura ≥ 22,7 °C" saca el FP del caso 11 pero crea el FN del caso 5). Sección 6: verificación en campo antes de pulverizar en campañas lluviosas. Completamos esta fila. También entraron las correcciones de la fila 10 de la bitácora. |
| 2 | 9,5/10 (A 2 · B 1,5 · C 3 · D 0,5 · E 1 · F 0,5 · G 1). Sin topes. Habilitados para la mesa. | **Agujero principal:** según el evaluador, la cifra "hasta un tercio" no aparece en la ficha. **Otro:** dónde entra el costo del producto en la regla, y no solo en el texto. | Verificamos la ficha: la cifra sí está, en el falso negativo del apartado "Si el sistema se equivoca" de T13 (`problematicas/tarde.md`, línea 534), igual en el repositorio original de la cátedra. La citamos textual en la sección 4 y aclaramos que la conclusión no depende de la cifra. En la sección 4 agregamos la cuenta de costo total en entrenamiento: el costo del producto es lo que frena el umbral en 20 h, porque bajarlo a 19 h suma una aplicación (caso 10) sin evitar ningún FN. |
