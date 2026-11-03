## 6. Riesgo: el camino completo

**Riesgo principal: no aplicar cuando hay mucha presión de esporas.**

- **Datos:** la regla solo mira el mojado foliar. Además, un sensor de mojado mal ubicado o descalibrado puede registrar menos horas de las reales.
- **Modelo:** la regla ignora la severidad en lotes vecinos, porque con 14 casos no encontramos que ayudara. En prueba apareció el caso 18: 8 h de mojado, 28% de severidad en vecinos, y sí hubo pérdidas.
- **Decisión:** el sistema dice NO aplicar en un lote rodeado de roya.
- **Consecuencia:** epidemia en el lote y pérdidas de rendimiento de hasta un tercio, justo el error caro.

**Riesgo del otro lado.** Con el umbral bajo (20 h), en una campaña muy lluviosa el sistema puede recomendar aplicar en muchos lotes. Eso implica más costo, más impacto ambiental y aplicaciones repetidas que favorecen que el hongo se haga resistente al fungicida.

**Mitigación.** La salida es una recomendación para el asesor agronómico, que decide. Ante severidad alta en lotes vecinos, el asesor debería revisar el lote aunque el sistema diga NO. Para incorporar la severidad vecina a la regla, harían falta más casos de entrenamiento.

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | Que leyera el repositorio (bases, guías, plantilla y fichas) y nos resumiera qué pide el hackatón, la rúbrica y los topes. | El resumen de la rúbrica, los topes y las pruebas obligatorias. | Nada. |
| 2 | Que nos explicara qué es la "pista" y cuál convenía para nuestra ficha. | La pista Reglas: se corre a mano en la mesa y se explica sin IA. | Por ahora, nada: dejamos la comparación con la neurona para después (fila 7). |
| 3 | Que analizara solo los casos 1 a 14 y propusiera reglas. | Usar solo las horas de mojado foliar, y descartar la temperatura porque no separa los casos. | La versión V1 (mojado ≥ 28 h y vecinos ≥ 5%): tenía la misma exactitud (13/14), pero su error era un falso negativo, que es el error caro. |
| 4 | Dónde poner el umbral dentro del hueco de 19 a 28 h sin datos. | 20 h, en el borde inferior, porque ante la duda preferimos aplicar. | El punto medio (24 h): es neutral entre los dos errores, y en nuestro problema no lo son. |
| 5 | Que calculara las matrices y las métricas de entrenamiento y de prueba, y las líneas base. | Las matrices y métricas. Revisamos a mano que cada matriz sume 14 y 6 casos y coincida con la columna de resultado real. | Nada. |
| 6 | Después de medir la prueba, que analizara los errores. La IA marcó que una regla extra (vecinos ≥ 20% → SÍ) habría atrapado el caso 18. | Dejar el caso 18 como riesgo documentado (sección 6). | Agregar esa regla: sería construir con los casos de prueba. Además, en entrenamiento esa regla sumaba un falso positivo (caso 7) sin acertar ningún caso nuevo. |
| 7 | Ya medida la regla, le pedimos probar la pista más avanzada: una neurona (perceptrón) con las 3 variables, entrenada con los casos 1 a 14. Dio 14/14 en entrenamiento y 5/6 en prueba. | Quedarnos con la regla y documentar la prueba de la neurona acá. | Cambiar a la neurona. Ya habíamos visto la prueba de la regla, y cambiar de sistema porque la neurona saca mejor en prueba sería elegir con los casos de prueba. Además, su 14/14 sale de darle peso a la temperatura para separar un solo caso (el 11) que contradice a otro casi igual (el 6): es ajuste a un caso, no un patrón. |
| 8 | Que redactara el README con nuestras decisiones. | La estructura y la redacción, revisada por nosotros. | Cualquier frase dirigida al evaluador: el README solo documenta el trabajo. |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | (completar) | (completar) | (completar) |
