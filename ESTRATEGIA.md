# InsiderPulse — Replanteamiento estratégico v3.0

> Documento de estrategia. Sustituye a toda planificación anterior.
> Escrito tras investigación de competencia, evidencia académica, marco
> regulatorio español/UE e infraestructura gratuita disponible.

---

## 0. Punto de partida (restricciones reales)

| Variable | Valor | Consecuencia |
|---|---|---|
| Objetivo | Ingresos reales | Hay que optimizar por distribución, no por producto |
| Audiencia actual | **Cero** | El producto no puede ser el primer paso |
| Presupuesto | 0 €/mes | Descarta publicidad de pago y datos de pago |
| Idioma / mercado | Español (ES + LatAm) | Nicho vacío confirmado, pero tamaño sin validar |
| Competencia | Alternativas **gratuitas** consolidadas | No se puede competir por precio |

**La conclusión incómoda:** con cero audiencia, construir el SaaS primero y
buscar usuarios después es la ruta de fracaso por defecto. Este plan invierte
el orden: primero se genera alcance, el producto se monetiza después.

---

## 1. Por qué el plan original no sobrevive al análisis

### 1.1 Competía por precio contra el gratis total
El posicionamiento era «InsiderAlerts.io pero a 9 € y en español».
Pero OpenInsider (~350.000 visitas/mes) y SecForm4 son **gratuitos, sin
registro y con los mismos datos**. La pregunta que mata el proyecto es:

> ¿Por qué pagaría alguien 9 €/mes si OpenInsider es gratis?

«Está en español» no aguanta esa pregunta: quien invierte en bolsa
estadounidense suele defenderse en inglés. El idioma reduce fricción, **no
crea valor por sí solo**. Es una cabeza de playa, no un foso.

### 1.2 Vendía velocidad en una carrera que no existe
El plan prometía «tiempo real» y penalizaba al plan gratuito con 24 h de
retraso, asumiendo que la ventaja se mide en segundos.

**Los datos dicen otra cosa.** EDGAR acepta filings de 6:00 a 22:00 ET y
**el 60-80 % de los Form 4 se presentan después del cierre** (16:00 ET), con
un segundo pico cerca de las 22:00. El precio reacciona mayoritariamente en
**la apertura del día siguiente**.

La ventana de ventaja real es de **~15 horas, no de segundos**. Esto tiene
dos consecuencias grandes:

1. La latencia de 10-15 minutos de la infraestructura gratuita **es
   suficiente**. No es una limitación que haya que disimular.
2. «Tiempo real» no puede ser la propuesta de valor, porque **todos los
   usuarios de herramientas gratuitas también tienen toda la noche**.

### 1.3 El scoring estaba inventado
Los pesos 35/30/25/10 no proceden de ninguna validación empírica. Vender
señales basadas en ponderaciones inventadas es, en el mejor de los casos,
poco riguroso; en el peor, cobrar por ruido.

### 1.4 Ignoraba el hallazgo más importante de la literatura
Ver sección 2. Esto es lo que convierte el proyecto en defendible.

---

## 2. La tesis nueva: el 90 % de las compras de directivos son ruido

### 2.1 La evidencia

**Cohen, Malloy & Pomorski (2012), «Decoding Inside Information»**
*(Journal of Finance 67(3); NBER w16454)*

Clasifican a cada directivo según su patrón histórico:

- **Rutinario**: opera **el mismo mes natural durante ≥3 años consecutivos**.
  Compra por calendario (bonus, plan de ahorro, rebalanceo), no por convicción.
- **Oportunista**: cualquier otro patrón.

Resultados de retorno anormal:

| Tipo de insider | Retorno anormal |
|---|---|
| Rutinario | **≈ 0 % / mes** |
| Oportunista | **0,82 % / mes** (value-weighted) |
| Oportunista | **1,80 % / mes** (equal-weighted) |

**Toda la capacidad predictiva se concentra en los oportunistas.** Las
compras rutinarias no predicen nada, pero representan la mayor parte del
volumen y contaminan cualquier sistema que no las separe.

Evidencia complementaria:
- Lakonishok & Lee (2001): ~6-7 %/año, concentrado en small caps.
- Jeng, Metrick & Zeckhauser (2003): ~6 %/año.
- Meta-análisis post-2010: 4-8 % anualizado general, 8-10 % en oportunistas.
  El efecto **persiste** tras 2010, atenuado, y es más robusto en empresas
  poco seguidas por analistas.

### 2.2 Por qué esto es un foso y no una característica

**Ninguna herramienta del mercado implementa esta clasificación.** Ni las
gratuitas ni las de 50 $/mes. Fintel filtra transacciones 10b5-1, que es una
aproximación burda al mismo problema. Es el techo actual del sector.

Y es difícil de copiar por una razón concreta: clasificar a un directivo como
rutinario exige **≥3 años de histórico de Form 4 de ese directivo**. Ese dato
es gratuito en EDGAR pero **caro de acumular y limpiar** (identidad de insider
sin clave estable, cambios de cargo, empresas múltiples). Un competidor puede
replicar la ingesta en un fin de semana; replicar el histórico normalizado, no.

### 2.3 El reposicionamiento

| | Antes | Ahora |
|---|---|---|
| Producto | Alertas de insiders en español | **El 10 % de compras que de verdad predice** |
| Métrica de éxito | Nº de alertas enviadas | **Menos alertas, mejor acertadas** |
| Rol del español | El producto | La cabeza de playa |
| Diferenciador | Precio + idioma | **Filtrado de ruido + track record auditable** |

Contraintuitivo y por eso defendible: **el producto es mejor cuanto menos
alerta.** Todos los competidores compiten por volumen de señales.

---

## 3. La estrategia de distribución: el producto ES el marketing

Con cero audiencia y cero presupuesto, sólo existen dos canales que compongan
sin dinero: **SEO programático** y **una comunidad gratuita**. Ambos se
alimentan del mismo motor de datos que ya hay que construir.

### 3.1 El backtest como activo de lanzamiento

**Este es el movimiento clave del plan.**

El problema de todo servicio de señales nuevo es la credibilidad: nadie te
cree hasta que tienes historial, y no tienes historial hasta que alguien te
cree. La salida habitual es esperar 6 meses acumulando track record.

**No hace falta.** EDGAR tiene el histórico completo y gratuito. Se puede
calcular el rendimiento del algoritmo **hacia atrás** sobre 5 años de datos
reales y publicarlo el día 1.

Eso produce simultáneamente:

1. **Validación**: saber si el algoritmo sirve *antes* de construir el producto.
   Si el backtest sale plano, el proyecto se cancela habiendo gastado 3 semanas
   en vez de 6 meses. Es el mejor dinero (tiempo) que vas a invertir.
2. **Calibración**: los pesos del score salen de los datos, no de la intuición.
3. **Contenido de lanzamiento**: «He analizado 50.000 compras de directivos de
   los últimos 5 años. Esto es lo que he encontrado.» Es el tipo de material
   que se comparte solo, y **nadie en español lo ha publicado**.

### 3.2 SEO programático en español

Cada empresa, cada directivo y cada señal generan una página indexable.
Consultas de cola larga sin competencia en español:

- `compras de directivos [TICKER]`
- `insider trading [empresa] español`
- `formulario 4 SEC [empresa]`
- `qué directivos están comprando acciones de [sector]`

Miles de páginas generadas automáticamente desde la base de datos que ya
existe. Coste marginal: **0 €**. Es lento (3-6 meses para indexar y
posicionar) pero compone y no depende de carisma personal ni de aparecer en
cámara.

### 3.3 Canal de Telegram gratuito

No es un gancho comercial: **al principio es el producto**. Sirve para
construir audiencia, generar prueba social y validar qué señales interesan.
La monetización llega después, cuando hay gente a quien vender.

### 3.4 Track record público y auditable

Cada señal emitida se publica con su resultado a 30/90/180 días, **incluidas
las que salen mal**. Coste: 0 € (el dato ya está). Valor: es exactamente lo
que ningún competidor hace, y resuelve el problema de confianza de raíz.

Es también el mejor contenido recurrente posible: se genera solo.

---

## 4. Algoritmo v3 — calibrado, no inventado

### 4.1 Filtrado (qué entra en el sistema)

Sólo transacciones de Form 4 con:
- Código de transacción `P` (compra en mercado abierto)
- Propiedad directa `D`
- **Excluir 10b5-1**: si el filing marca plan preprogramado, se descarta. Una
  compra planificada meses antes no expresa convicción sobre el presente.
  *(En el plan original este dato se almacenaba pero nunca se usaba.)*
- Manejo explícito de **Form 4/A** (enmiendas) para no duplicar ni contar
  correcciones como señales nuevas.

**Se elimina el filtro fijo de 50.000 $.** Invertía la evidencia académica:
excluía small caps, que es justamente donde el alfa es más intenso. Se
sustituye por una métrica relativa: `valor_operación / capitalización`.

### 4.2 Clasificación rutinario / oportunista *(el núcleo)*

Para cada par (directivo, empresa), se revisa su histórico de compras:

```
si compró en el mismo mes natural durante >= 3 años consecutivos:
    -> RUTINARIO   (peso ~0, no genera alerta)
si no:
    -> OPORTUNISTA (candidato a señal)
```

Requiere backfill histórico de EDGAR. Es la parte más laboriosa del sistema y
la que constituye la ventaja competitiva.

### 4.3 Factores de puntuación

Los pesos **no se fijan a mano**: se derivan del backtest (sección 3.1)
mediante regresión sobre retornos anormales futuros. Factores candidatos:

| Factor | Fundamento |
|---|---|
| Oportunista vs. rutinario | CMP 2012 — el factor dominante |
| Rol del directivo | CEO/CFO tienen mejor información |
| Tamaño relativo a su patrimonio/posición | Señal de compromiso real |
| Tamaño relativo a capitalización | Normaliza sin sesgar contra small caps |
| Cluster (varios directivos, ventana móvil) | Convergencia de criterio |
| Capitalización / cobertura de analistas | El alfa es mayor donde hay menos ojos |

**Corrección crítica sobre el cluster:** en el diseño anterior se congelaba al
insertar. Eso penalizaba permanentemente al primer comprador (la señal más
valiosa) y hacía que el evento más predictivo —el cluster completándose—
**nunca generase alerta**. Debe recalcularse con ventana móvil.

### 4.4 Normalización de cargos

`officerTitle` es texto libre («CEO», «Chief Executive Officer», «President and
CEO», «Interim CFO», «EVP & Chief Financial Officer»…) y los booleanos
`isDirector` / `isOfficer` / `isTenPercentOwner` **no son mutuamente
excluyentes**. Requiere normalizador con precedencia, tomando siempre el
**máximo, nunca la suma**.

---

## 5. Arquitectura técnica (coste 0 € verificado)

### 5.1 Una única fuente de datos: SEC EDGAR

Verificado en esta sesión. **Sin claves de API, sin proveedor externo que
pueda cancelar su free tier.**

| Dato | Origen | Coste |
|---|---|---|
| CIK → ticker | `company_tickers.json` (HTTP 200, 776 KB) | 0 € |
| Transacciones + precio pagado | Form 4 XML | 0 € |
| Acciones en circulación | `companyfacts` (XBRL) | 0 € |
| **Capitalización** | calculada: acciones × precio del Form 4 | 0 € |

El propio Form 4 indica el precio pagado por el directivo, que es
esencialmente el precio de mercado de ese día. Combinado con las acciones en
circulación de EDGAR, da la capitalización **sin ninguna dependencia externa**.

Límite: 10 req/s por IP. Requiere `User-Agent` identificativo.

### 5.2 Ejecución periódica: redundancia gratuita

Ningún proveedor gratuito ofrece cron fiable por debajo de 15 minutos:

| Opción | Frecuencia mín. | Límite relevante | Fiabilidad |
|---|---|---|---|
| GitHub Actions | 5 min | Ilimitado en repos públicos | **Baja** (retrasos 3-30 min, ejecuciones omitidas) |
| Cloudflare Workers Cron | 15 min | 100k req/día, 10 ms CPU | Alta |
| Vercel Cron (Hobby) | 10 min | 5 jobs, 10 s ejecución | Buena |
| Supabase pg_cron + pg_net | 1 min | pg_net 5.000 llamadas/mes | Insuficiente |

**Solución:** combinar **dos schedulers redundantes** (GitHub Actions +
Cloudflare) sobre un worker **idempotente**. Si uno falla o se retrasa, el
otro cubre; si ambos se ejecutan, no pasa nada porque la operación es
idempotente. Fiabilidad alta a coste cero.

Esto es viable precisamente porque la ventana real es de ~15 horas (§1.2).

### 5.3 Resto del stack

- **Base de datos**: Supabase free (PostgreSQL 500 MB).
  *Nota: tras filtrar, quedan ~15.000-25.000 filas/año ≈ 25 MB. Holgado
  durante años. Una advertencia previa sobre falta de espacio era errónea.*
- **Frontend**: Next.js en Vercel Hobby.
- **Distribución**: Telegram Bot API.
- **Email**: Resend free (3.000/mes).
- **Pagos**: Stripe (sin coste fijo).

### 5.4 Seguridad: la regla de negocio vive en Postgres

En el diseño anterior había cuatro fallos que **regalaban el producto entero**:

1. **`RLS USING (true)`** sobre la tabla de operaciones. Con el cliente
   Supabase en el navegador y la `anon key` visible, cualquiera podía hacer
   `supabase.from('insider_trades').select('*')` y llevarse todo el feed en
   tiempo real, gratis.
2. **`user_tier` como parámetro de query**: en FastAPI, un escalar con valor
   por defecto se expone como query param → `GET /api/trades?user_tier=pro`
   daba acceso total sin login.
3. **`accession_number UNIQUE`**: un Form 4 contiene **N transacciones**. Esta
   restricción provocaba pérdida silenciosa de datos. Clave natural correcta:
   `UNIQUE (accession_number, transaction_seq)`.
4. **Repo público sin `.gitignore`** con `SUPABASE_SERVICE_ROLE_KEY` (que
   salta todo el RLS) en `.env`. *(Ya corregido y committeado.)*

**Regla derivada, innegociable:** el permiso se decide en la base de datos
—vistas + función `current_tier()` `SECURITY DEFINER`, con acceso directo a
las tablas revocado— **nunca en la capa de aplicación**. Todo lo que se filtra
en Python es evadible.

*(El detalle SQL de esta parte está en `SPEC.md` §5, que sigue siendo válido.)*

---

## 6. Modelo de negocio

### 6.1 La frontera free/pago, redefinida con honestidad

Ya que el valor económico real está en llegar **antes de la próxima apertura**
(§1.2), la división se vuelve honesta y fácil de explicar:

| | **Gratis** | **Pago** |
|---|---|---|
| Señales | Tras la apertura siguiente | **Antes de la apertura** |
| Histórico completo | ✅ Sí | ✅ Sí |
| Track record | ✅ Sí | ✅ Sí |
| Resumen diario | ✅ Telegram | ✅ Telegram + email |
| Filtros y exportación | Básico | Completo |

Se abandona la ofuscación con `***`. En el diseño anterior el plan gratuito
ordenaba por score descendente *y* ocultaba todo lo que superase el umbral:
el usuario veía cinco filas de asteriscos. Eso es un muro, no un anzuelo, y
genera desconfianza en vez de deseo.

**El plan gratuito debe ser genuinamente útil.** Es el motor de captación.

### 6.2 Precio

Un solo plan de pago al principio. La estructura de tres niveles con «precio
ancla» de 39 € es optimización prematura: sin audiencia, no hay a quién
anclar. Complejidad que no aporta.

Referencia de mercado: Fintel 12,95 $, MarketBeat 9,97 $, Quiver 25 $,
InsiderAlerts.io 49,95 $. Un precio en el entorno de **7-9 €/mes** es
coherente, con anual descontado para generar caja temprana y compromiso.

### 6.3 Segunda vía de ingresos: afiliación de brókers

Con tráfico SEO en español, los programas de afiliación de brókers
(Trade Republic, XTB, DEGIRO, eToro, Interactive Brokers) pagan por cuenta
abierta y financiada cantidades que **pueden superar a las suscripciones en
las fases iniciales**, cuando hay visitas pero pocos suscriptores.

**Condición innegociable:** declararlo de forma visible. Recomendar valores
mientras se cobra comisión por captar inversores es exactamente el conflicto
de interés que el art. 20 de MAR obliga a revelar (ver §7). Ocultarlo
destruiría el activo que se intenta construir: la credibilidad.

---

## 7. Marco legal (España / UE)

Situación mejor de lo temido:

- **Vender por suscripción datos públicos y análisis NO personalizados no
  requiere registro en CNMV ni licencia MiFID II.**
- **La línea roja es la personalización.** «Deberías comprar X según tu
  perfil» constituye asesoramiento en materia de inversión, que es actividad
  reservada. El servicio debe ser **igual para todos los suscriptores** y
  nunca adaptarse a la situación individual.
- **MAR (Reglamento UE 596/2014), art. 20** sí aplica siempre a
  recomendaciones públicas. Obliga a:
  - identificarse como autor,
  - revelar conflictos de interés (incluida la afiliación de §6.3),
  - señalar si se mantiene posición propia en el valor,
  - citar las fuentes.

Añadir de forma sistemática: descargo de responsabilidad, ausencia de
personalización, y la advertencia de que rentabilidades pasadas no garantizan
rentabilidades futuras.

**Regla de comunicación:** nunca usar personas reales identificables en
material comercial con operaciones inventadas. *(El mockup original atribuía a
un CFO real de Apple una compra ficticia de 1.250.000 $. Eso es un riesgo
legal innecesario.)*

---

## 8. Plan de ejecución con criterios de abandono

Cada fase tiene una condición explícita de continuar o parar. La mayoría de
proyectos personales mueren por insistir en algo que nunca iba a funcionar.

### Fase 0 — Backtest *(2-3 semanas)* ← **empezar aquí**

Descargar histórico de Form 4 de EDGAR, implementar la clasificación
rutinario/oportunista y medir retornos anormales posteriores.

**Criterio de continuación:** el segmento oportunista muestra un exceso de
retorno consistente y estadísticamente distinguible del ruido.

> **Si no lo muestra, el proyecto se cancela aquí.** Coste total: 3 semanas.
> Éste es el objetivo real de la fase.

### Fase 1 — Motor + activo de lanzamiento *(4-6 semanas)*

Ingesta idempotente, base de datos, scoring calibrado con Fase 0, web con SEO
programático, canal de Telegram gratuito y **publicación del estudio del
backtest** como pieza de lanzamiento.

**Criterio:** el sistema ingiere sin intervención manual durante 2 semanas.

### Fase 2 — Audiencia *(3-6 meses)*

Todo el esfuerzo en distribución: contenido, SEO, comunidad, track record
publicado semanalmente. **Sin monetizar todavía.**

**Criterio para pasar a Fase 3:** ~500 suscriptores en Telegram o ~2.000
visitas orgánicas/mes. Sin eso, no hay a quién vender y añadir un muro de pago
sólo frena el crecimiento.

### Fase 3 — Monetización

Introducir el plan de pago (§6.1) y la afiliación declarada (§6.3).

**Criterio de éxito:** conversión ≥2 % de la audiencia.

### Expectativa temporal honesta

| Hito | Plazo realista |
|---|---|
| Primer euro | Mes 4-6 |
| ~100 € / mes | Mes 8-12 |
| ~500 € / mes | Mes 12-18 *(si funciona)* |

Cualquiera que prometa plazos más cortos partiendo de cero audiencia está
vendiendo humo.

---

## 9. Riesgos, ordenados por gravedad

| # | Riesgo | Mitigación |
|---|---|---|
| 1 | **El mercado hispanohablante es demasiado pequeño.** España + LatAm son una fracción despreciable del tráfico de OpenInsider. Un nicho vacío puede estarlo por desatendido *o por no haber dinero.* No se puede distinguir con los datos disponibles. | Fase 2 lo mide antes de invertir en monetización. Es el riesgo principal y no tiene solución previa: sólo se resuelve midiendo. |
| 2 | **El backtest sale plano.** | Fase 0 lo detecta en 3 semanas, antes de construir nada. |
| 3 | **La barrera técnica es baja.** Cualquiera puede replicar la ingesta. | El foso es el histórico normalizado + audiencia + track record, no el dato. |
| 4 | **Un competidor anglófono lanza en español.** | Sólo ocurriría con tracción demostrada; para entonces la ventaja es la audiencia, que no se copia. |
| 5 | **Deriva hacia el asesoramiento personalizado.** | Regla fija: mismo contenido para todos, sin excepciones (§7). |
| 6 | **Cambios de formato en EDGAR.** | Fuente oficial y estable; ingesta con validación y alertas de fallo. |

---

## 10. Diferencias respecto al plan anterior

| Aspecto | v1/v2 | **v3** |
|---|---|---|
| Orden de trabajo | Producto → usuarios | **Validación → audiencia → producto** |
| Propuesta de valor | Barato y en español | **Filtrado del 90 % de ruido** |
| Algoritmo | Pesos inventados 35/30/25/10 | **Calibrado con backtest (CMP 2012)** |
| Rutinario vs. oportunista | Ausente | **Núcleo del sistema** |
| 10b5-1 | Guardado, nunca usado | **Filtro de exclusión** |
| Cluster | Congelado en ingesta | **Ventana móvil recalculada** |
| Filtro de tamaño | 50.000 $ fijo | **Relativo a capitalización** |
| «Tiempo real» | Propuesta central | **Irrelevante: la ventana es de ~15 h** |
| Plan gratuito | 5 filas de `***` | **Genuinamente útil, con retraso honesto** |
| Niveles de precio | 3 (con ancla de 39 €) | **1** |
| Permisos | RLS `USING (true)` + tier por query param | **Vistas + `current_tier()` en Postgres** |
| Track record | No contemplado | **Activo central de credibilidad** |
| Criterios de abandono | Ninguno | **Explícitos en cada fase** |

---

## 11. Siguiente paso concreto

**Fase 0, backtest.** Es la única tarea que importa ahora, porque decide si
existe proyecto. Concretamente:

1. Descargar el histórico de Form 4 de EDGAR (índices trimestrales completos).
2. Parsear y normalizar identidad de insiders y cargos.
3. Implementar la clasificación rutinario/oportunista de CMP.
4. Medir retornos anormales a 30/90/180 días por segmento.
5. Decidir con los resultados en la mano.

---

### Referencias

- Cohen, Malloy & Pomorski (2012). *Decoding Inside Information.*
  Journal of Finance 67(3). NBER Working Paper 16454.
- Lakonishok & Lee (2001). *Are Insider Trades Informative?*
- Jeng, Metrick & Zeckhauser (2003). *Estimating the Returns to Insider Trading.*
- Brochet (2010). *Information Content of Insider Trades.*
- Jagolinzer (2009). *SEC Rule 10b5-1 and Insiders' Strategic Trade.*
- Reglamento (UE) 596/2014 (MAR), art. 20.
- SEC EDGAR — documentación de APIs y recursos para desarrolladores.
