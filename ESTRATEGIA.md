# InsiderPulse — Replanteamiento estratégico v3.2

> Documento de estrategia. Sustituye a toda planificación anterior.
> Escrito tras investigación de competencia, evidencia académica, marco
> regulatorio español/UE e infraestructura gratuita disponible.
>
> **v3.1** — corrige el supuesto de «coste 0 €»: varios free tier prohíben el
> uso comercial (§5.0) e incorpora el contexto de precio como parte del
> producto (§4.5), con su modelo de costes (§5.0.1) y de licencias (§5.5).
>
> **v3.2** — precios de licencia **verificados en fuente primaria**. El display
> comercial más barato es 399 €/mes, no 45-140 €/mes: se rectifica. Se resuelve
> con la **arquitectura de tres capas** (§5.5), que evita esa licencia y deja el
> coste en 35-70 €/mes. Se descartan con motivos el scraping (§5.5.1), IEX,
> Alpaca y los envoltorios open source. Se sustituye la fuente del backtest por
> **QuantConnect** para eliminar el **sesgo de supervivencia**.
>
> **Verificaciones pendientes antes de la Fase 1** (§5.5): leer las condiciones
> primarias de los widgets de TradingView; obtener por escrito si Marketstack
> permite display; calcular el coste real de Databento *pay-as-you-go*.

---

## 0. Punto de partida (restricciones reales)

| Variable | Valor | Consecuencia |
|---|---|---|
| Objetivo | Ingresos reales | Hay que optimizar por distribución, no por producto |
| Audiencia actual | **Cero** | El producto no puede ser el primer paso |
| Presupuesto | 0 €/mes hasta facturar + 100-300 € puntuales | El MVP validable cuesta ~1 €/mes; la producción con datos de mercado, 35-70 €/mes, **cubiertos con 5-9 suscriptores** (§5.0.1) |
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

Con cero audiencia, sólo dos canales componen sin gasto recurrente: **SEO
programático** y **una comunidad gratuita**. Ambos se alimentan del mismo
motor de datos que hay que construir de todos modos.

A ellos se añade un tercero con papel distinto: **paid social como instrumento
de validación puntual** (§3.5). No es un canal de crecimiento —la aritmética
lo impide— sino una forma de comprar en dos semanas una respuesta que al SEO
le costaría seis meses dar.

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

### 3.5 Paid social: instrumento de validación, no canal de crecimiento

**Meta Ads entra en el plan con un papel muy concreto y acotado: comprar
información, no clientes.**

#### Por qué no puede ser el motor de crecimiento

Benchmarks 2025-26 para el vertical financiero en España: CPC **0,95-1,60 €**,
CPL **35-50 €**. El embudo hasta cliente de pago sale así:

| | Optimista | Realista |
|---|---|---|
| CPC | 0,95 € | 1,20 € |
| Landing → registro gratuito | 20 % | 12 % |
| Coste por registro gratuito | 4,75 € | 10 € |
| Registro → pago | 5 % | 3 % |
| **CAC (cliente de pago)** | **95 €** | **333 €** |

Frente a un **LTV de ~65 €** (9 €/mes × ~8 meses de permanencia realista,
descontadas comisiones).

**Se pierde dinero en cada cliente incluso en el escenario optimista.** No es
un problema de creatividad ni de segmentación: es aritmética. Paid social
sobre suscripción de bajo ticket es una trampa conocida.

> **Corolario importante:** como el paid social se usa sólo para validar y no
> para escalar, **la presión sobre el precio desaparece**. Con adquisición
> orgánica el CAC tiende a 0 y los 9 €/mes vuelven a ser perfectamente
> viables. Si algún día se quisiera escalar con pago, habría que subir a
> 19-29 €/mes o convertir el plan anual por adelantado en la oferta principal.

#### Riesgo de plataforma (grave, verificar antes de construir)

En 2025-26 Meta trasladó **«Finance and Insurance» a Special Ad Categories**
en la UE:

- Exige **verificación del anunciante** y, en muchos casos, acreditar licencia
  regulatoria (CNMV o entidad MiFID) — que este proyecto no necesita
  legalmente, pero que Meta puede exigir igualmente.
- **Sin lookalike audiences**, sin segmentación por edad ni sexo, sólo
  ubicación amplia.
- **«Signal selling» figura explícitamente como contenido prohibido.**

Lo último es un impacto directo: esto es, literalmente, un servicio de
señales. El modo de fallo típico no es que rechacen un anuncio, sino
**restricción de la cuenta publicitaria**.

*Nota de fiabilidad: estas fuentes son blogs especializados, no documentación
primaria de Meta. La dirección es consistente entre ellas, pero debe
verificarse empíricamente con gasto real antes de depender del canal.*

#### Encuadre obligatorio del mensaje

El encuadre determina simultáneamente la aprobación en Meta y el cumplimiento
de MAR art. 20:

| ❌ Nunca | ✅ Así |
|---|---|
| «Gana dinero copiando a los directivos» | «Los directivos declaran sus compras a la SEC. Te lo traducimos.» |
| «Rentabilidad del X %» | «Qué dicen los datos de los últimos 5 años» |
| Señales, alertas de trading | Servicio de información y análisis no personalizado |

**La mejor creatividad disponible es el backtest**: «He analizado 50.000
compras de directivos de los últimos 5 años» es un gancho verdadero, que no
promete rentabilidades y por tanto sobrevive a la revisión automática. Una
razón más para que la Fase 0 vaya primero.

#### Optimización del gasto

Los datos muestran que **Reels tiene un CPM de 3 a 5 veces inferior al Feed**
(≈2-6 $ frente a 14-16 $). Para un test cuyo objetivo es maximizar señal por
euro, la colocación en Reels multiplica el tamaño de muestra con el mismo
presupuesto.

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

### 4.5 Contexto de precio *(decisión revisada)*

**Una versión anterior de este documento proponía prescindir de cotizaciones
para mantener el coste en 0 €. Se descarta.** El motivo es de producto, no
técnico: una alerta sin contexto de precio no es accionable. Si la acción ya
subió un 30 % desde que el directivo compró, la señal es información, no
oportunidad. El usuario necesita saber **si todavía puede entrar en condiciones
comparables a las del directivo**.

Esto tiene dos consecuencias que se asumen conscientemente: rompe el principio
de fuente única (§5.1) e introduce un coste recurrente (§5.5).

#### Las dos hipótesis están en conflicto — hay que medir, no asumir

| Hipótesis | Evidencia | Predicción |
|---|---|---|
| **Contrarian** | Rozeff & Zaman (1998); Jenter (2005) | Los directivos compran tras caídas y en *value*; esas compras anticipan reversiones. → **Cuanto más barato respecto a lo que pagó, mejor.** |
| **Momentum** | Estudio de microcaps (2018-2024) | Las compras tras subidas > 10 % también dieron 6,3 % de retorno anormal. → **Comprar caro también funcionó.** |

La regla intuitiva «más barato = mejor oportunidad» **puede ser falsa**. Es
exactamente el tipo de supuesto que hundió el algoritmo v1. Se trata como
**hipótesis a validar en el backtest (Fase 0)**, no como regla de negocio.

#### Factores candidatos

| Factor | Cálculo |
|---|---|
| Descuento / prima frente al directivo | `precio_actual / precio_pagado − 1` |
| Caída desde máximo de 52 semanas | contexto de castigo previo |
| Rentabilidad previa 3-6 meses | discrimina contrarian vs. momentum |
| Múltiplos P/B, P/E | señal *value* (Rozeff-Zaman) |

#### Límite regulatorio (MAR 596/2014, §7)

Se muestran **hechos**, nunca **veredictos**:

- ✅ «El consejero delegado pagó 47,20 $. Hoy cotiza a 44,10 $ (−6,6 %).»
- ❌ «Buena oportunidad de compra.»

La segunda formulación es recomendación de inversión y activa obligaciones
regulatorias que este proyecto no puede asumir.

---

## 5. Arquitectura técnica y costes reales

### 5.0 Corrección: la arquitectura «0 €» no era cierta para un producto de pago

Al verificar las condiciones de uso —no sólo los límites técnicos— aparecen
**tres restricciones que no son de cuota, sino contractuales**. Un free tier
puede aguantar la carga y aun así prohibir el uso:

| Hallazgo | Impacto |
|---|---|
| **Vercel Hobby prohíbe el uso comercial** | No sirve en cuanto hay un solo cliente de pago |
| **Los free tier de datos de mercado prohíben redistribución/display** | Finnhub, Tiingo, Polygon, Twelve Data, EODHD, Alpha Vantage |
| **Supabase Free pausa proyectos inactivos** | Mitigable: el worker escribe cada 10 min, nunca se pausa |

> ⚠️ Verificado con fuentes secundarias y comprobación empírica parcial. **Leer
> los ToS primarios del proveedor elegido antes de facturar al primer cliente.**

**Sustituciones:** Cloudflare Pages/Workers en lugar de Vercel (permite uso
comercial en el plan gratuito). Supabase Free se mantiene en MVP y se pasa a Pro
cuando haya clientes de pago, por los *backups*, no por la cuota.

### 5.0.1 Modelo de costes en dos etapas

| Concepto | MVP v0 | Producción v1 |
|---|---|---|
| Frontend (Cloudflare Pages) | 0 € | 0 € |
| Cron (GitHub Actions, repo público) | 0 € | 0 € |
| Base de datos (Supabase) | 0 € (Free) | ~23 € (Pro, backups) |
| **Datos de mercado** | **0 €** (widget TradingView, §5.5) | **11-46 €** (licencia *non-display*) |
| Telegram | 0 € | 0 € |
| Email (Resend, 3.000/mes) | 0 € | 0 € |
| Dominio | ~1 € | ~1 € |
| **Total mensual** | **~1 €** | **~35-70 €** |

Stripe aparte: 1,5 % + 0,25 € por transacción europea (≈ 0,39 € sobre 9 €).

**Punto de equilibrio a 9 €/mes (≈ 8,61 € netos): entre 5 y 9 suscriptores.**
Es un listón muy bajo, y es lo que convierte el coste en una decisión manejable
en lugar de un bloqueo.

**La cifra depende por completo de una decisión de diseño.** Si el producto
sirviera cotizaciones desde sus propios servidores necesitaría **licencia de
display: 399 €/mes** (EODHD, el precio publicado más barato) y el equilibrio se
iría a **~48 suscriptores**. Evitarlo con la arquitectura de tres capas (§5.5)
vale unos **350 €/mes** y es la diferencia entre viable y no viable en fase
temprana.

**MVP v0 — producción sin licencia de datos:** precio del Form 4 (EDGAR,
dominio público) + widget de TradingView incrustado para la cotización actual.
Coste cero y sin obligaciones de licencia, porque **el dato lo carga el
navegador del usuario y nunca pasa por el servidor**. Suficiente para validar si
el mercado quiere el producto antes de contratar nada. La licencia *non-display*
sólo se contrata si la Fase 0 demuestra que puntuar el contexto de precio añade
señal (§4.5).

### 5.1 La fuente primaria: SEC EDGAR (dominio público)

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
| Vercel Cron (Hobby) | 10 min | 5 jobs, 10 s ejecución | Buena, pero **no utilizable comercialmente** (§5.0) |
| Supabase pg_cron + pg_net | 1 min | pg_net 5.000 llamadas/mes | Insuficiente |

**Solución:** combinar **dos schedulers redundantes** (GitHub Actions +
Cloudflare) sobre un worker **idempotente**. Si uno falla o se retrasa, el
otro cubre; si ambos se ejecutan, no pasa nada porque la operación es
idempotente. Fiabilidad alta a coste cero.

Esto es viable precisamente porque la ventana real es de ~15 horas (§1.2).

### 5.3 Resto del stack

- **Base de datos**: Supabase Free en MVP (PostgreSQL 500 MB) → Pro al facturar.
  *Tras filtrar quedan ~15.000-25.000 filas/año ≈ 25 MB: la cuota sobra durante
  años. El motivo para pasar a Pro son los backups y el SLA, no el espacio.*
- **Frontend**: Next.js en **Cloudflare Pages** *(Vercel Hobby prohíbe el uso
  comercial; Vercel Pro son 20 $/mes por lo mismo que Cloudflare da gratis).*
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

### 5.5 Cotizaciones: fuentes, licencias y estrategia de compra

#### Fuentes descartadas

| Fuente | Motivo |
|---|---|
| **Stooq** | **Verificado empíricamente:** ya no sirve CSV. Devuelve un reto *proof-of-work* en JavaScript. Automatizarlo sería sortear un control de acceso. *(Recomendada en una versión previa de este documento por error, sin probarla.)* |
| **Yahoo Finance / `yfinance`** | Sin API oficial desde 2017. Endpoints internos no documentados; sus condiciones prohíben acceso automatizado y uso comercial. Se rompe varias veces al año. Inaceptable con clientes de pago. |
| **Scraping propio / Apify** | Apify es alojamiento, **no una licencia**: sus condiciones dejan toda la responsabilidad legal en el usuario. Y el precedente aplicable aquí no es el estadounidense (*hiQ*, *Meta v. Bright Data*) sino **el europeo: *Ryanair v. PR Aviation* (TJUE)**, donde el derecho *sui generis* de bases de datos **sí** puede bloquear el scraping comercial de datos públicos y factuales. Ver §5.5.1. |
| **Envoltorios open source** (OpenBB, `pandas-datareader`, `akshare`, `findatapy`) | **Ninguno aporta datos propios.** Son envoltorios de APIs de terceros y heredan sus restricciones. La licencia MIT/Apache cubre **el código, no el dato.** |
| **IEX Exchange HIST** (gratis) | IEX es ~2-3 % del volumen consolidado. Para microcaps ilíquidas **puede no haber ninguna operación en IEX en días enteros**, y su cierre no es el cierre oficial consolidado. Justo el segmento donde está tu alfa. Además: formato PCAP binario y sólo 12 meses de retención. |
| **Alpaca Markets** | El plan gratuito usa IEX (mismo problema). Y **las microcaps OTC no están disponibles ni en el plan de 99 $/mes**: exigen ser *broker partner*. Su uso en un SaaS ajeno a su correduría es zona gris. |
| **WIKI / Quandl (CC0)** | Licencia impecable —dominio público absoluto— pero **los datos terminan en abril de 2018**. Inservible para producción; residual para backtest. |

#### La distinción que determina el coste

| Tipo de dato | Régimen de licencia | Coste |
|---|---|---|
| **Tiempo real** | Acuerdos con cada mercado (NYSE, Nasdaq) + **tarifa por suscriptor** + auditorías | Prohibitivo |
| **Cierre diario / diferido** | Sólo contrato con el proveedor, tarifa plana | Asumible |

**InsiderPulse sólo necesita cierre diario.** La ventana real es de ~15 horas
(§1.2), así que el tiempo real no aporta nada y multiplica el coste. Esto no es
una renuncia: es coherente con la tesis del producto.

#### La distinción que lo cambia todo: *display* frente a uso interno

Éste es el hallazgo central de la investigación, y **vale unos 350 €/mes**:

| Régimen | Qué permite | Precio más barato verificado |
|---|---|---|
| **Display comercial** | Servir la cotización al usuario final desde tus servidores | **EODHD, 399 €/mes** *(precio publicado; los planes de 0-99 € son sólo uso personal)* |
| **Uso interno / *non-display*** | Calcular con el dato sin mostrarlo en crudo | **marketdata.app 12 $/mes**, Norgate ~25 $/mes, Tiingo 50 $/mes |

Mi estimación previa de «45-140 €/mes» era **optimista por un factor de 3-9** para
display. La corrección no invalida el proyecto, pero obliga a diseñarlo para
**no necesitar nunca licencia de display**.

> ⚠️ **Marketstack (9,99 $/mes) parece la ganga y no lo es:** su contrato,
> revisado directamente, **no menciona derechos de redistribución ni display**.
> No los concede ni los niega. Usarlo comercialmente sin confirmación escrita es
> riesgo legal sin cuantificar.

#### La arquitectura que evita la licencia cara

Tres capas, cada una con su propio régimen jurídico:

| Qué ve el usuario | Origen | Régimen | Coste |
|---|---|---|---|
| «El consejero pagó 47,20 $» | Form 4 (EDGAR) | **Dominio público** | 0 € |
| Gráfico con la cotización actual | **Widget de TradingView incrustado** | Licenciado por TradingView; **el dato lo carga el navegador del usuario, no tu servidor** → nunca redistribuyes nada | 0 € |
| «Cotiza por debajo del precio del directivo» | Calculado en tu servidor | Uso interno + dato derivado | 12-50 $/mes |

Los widgets gratuitos de TradingView **permiten expresamente su uso en webs
comerciales y de suscripción de pago**, a cambio de mantener su marca y su
enlace. La comparación la hace el ojo del usuario; tú no publicas ninguna
cotización.

> **Regla de diseño innegociable:** mostrar **banda categórica**, nunca el
> porcentaje exacto. «−6,6 %» junto al precio del Form 4 (que es público)
> permite reconstruir la cotización, y eso **sí es display**. «Por debajo del
> precio del directivo» no es reconstruible. Además es mejor producto: una ayuda
> a la decisión en lugar de un terminal de datos.

*(`lightweight-charts` de TradingView es Apache 2.0, pero exige aportar tu propio
feed: no resuelve el problema del dato.)*

#### Backtest: el sesgo de supervivencia es un riesgo reputacional

El backtest es **el activo de lanzamiento** (§3.1): un número público que hay que
poder defender. Si se calcula sólo sobre empresas que hoy siguen cotizando, el
resultado sale **inflado de forma sistemática**, porque las que quebraron han
desaparecido de la muestra. Publicar esa cifra y que alguien lo detecte
destruiría exactamente el activo que se pretende construir.

| Opción | Coste | Deslistadas | Profundidad |
|---|---|---|---|
| **QuantConnect** (datos AlgoSeek en su plataforma) | **0 €** | ✅ Sí | Desde 1998 |
| **Norgate Data Platinum** | ~25 $/mes + 208 $ inicial | ✅ Sí, desde 1990 | >30 años |
| **Sharadar** (Nasdaq Data Link) | ~30-100 $/mes | ✅ Sí | >20 años |
| WIKI/Quandl (CC0) | 0 € | Parcial | **Sólo hasta 2018** |

**Recomendación: QuantConnect para la Fase 0.** Coste cero, libre de sesgo de
supervivencia y suficiente para decidir si el proyecto sigue adelante. La
limitación —el backtest corre en su plataforma, no descargas el dato bruto— es
irrelevante para esta fase. *(Descartado el «Twelve Data free» que recomendaba la
versión anterior de este documento: su free tier prohíbe el uso comercial y,
sobre todo, no resuelve el sesgo de supervivencia.)*

#### 5.5.1 Por qué el scraping no es la salida barata

Es la vía que intuitivamente parece resolver el coste. No lo hace: **traslada el
riesgo en lugar de eliminarlo.**

| Precedente | Qué establece | ¿Aplica aquí? |
|---|---|---|
| *hiQ v. LinkedIn* (EEUU) | Raspar datos públicos no es delito informático… **pero hiQ perdió por incumplimiento contractual** | Parcialmente |
| *Meta v. Bright Data* (EEUU) | Sin login y con datos públicos, el riesgo baja | Parcialmente |
| ***Ryanair v. PR Aviation*** **(TJUE)** | **El derecho *sui generis* de bases de datos puede bloquear el scraping comercial de datos públicos y factuales en la UE** | **Sí. Es el aplicable.** |

El precedente favorable es estadounidense; el que rige aquí es el europeo, y es
el desfavorable.

Pero el argumento decisivo es de negocio, no jurídico: **el activo de este
proyecto es la credibilidad** (§3.4, track record público y auditable).
Publicar un histórico auditable construido sobre datos que no se tiene derecho a
usar es una contradicción interna. Y el modo de fallo típico no es un pleito: es
un bloqueo de IP o un requerimiento que **mata el servicio de un día para otro,
con clientes ya pagando**.

**Dónde sí es admisible:** en la Fase 0. Investigación interna, sin
redistribución ni publicación del dato bruto. Ahí el riesgo es marginal.

#### Serie histórica propia

Guardar siempre en base de datos el precio del Form 4 (dominio público, es del
propio filing). Con el tiempo genera una serie histórica **propia, gratuita e
incancelable** que ningún proveedor puede retirar.

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

**Dependencia con el canal de adquisición** *(ver §3.5)*: este precio sólo se
sostiene con **adquisición orgánica**, donde el CAC tiende a 0. Si en algún
momento se decidiera escalar con publicidad pagada, la aritmética obliga a
subir a 19-29 €/mes o a hacer del plan anual por adelantado la oferta
principal. **No se puede tener a la vez precio bajo y adquisición pagada.**

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

**Coste: 0 €** con **QuantConnect** (datos AlgoSeek desde 1998, incluidas
empresas deslistadas). No se contrata nada hasta la Fase 1.

> **Requisito innegociable: el backtest debe estar libre de sesgo de
> supervivencia.** Calcularlo sólo sobre empresas que hoy siguen cotizando
> infla el resultado de forma sistemática. Como esta cifra se va a publicar
> (§3.1), un sesgo detectable destruiría la credibilidad que el proyecto intenta
> construir. Por eso no vale cualquier fuente gratuita (ver §5.5).

Se responden **tres** preguntas, no una:

1. ¿El segmento oportunista bate al rutinario? *(la tesis, §2)*
2. ¿Qué pesos tienen realmente los factores? *(§4.3, en lugar de inventarlos)*
3. **¿El contexto de precio añade señal, y en qué dirección?** *(§4.5 —
   contrarian y momentum predicen lo contrario; aquí se decide cuál es cierto,
   y si justifica pagar por datos en producción.)*

**Criterio de continuación:** el segmento oportunista muestra un exceso de
retorno consistente y estadísticamente distinguible del ruido.

> **Si no lo muestra, el proyecto se cancela aquí.** Coste total: 3 semanas.
> Éste es el objetivo real de la fase.

### Fase 0.5 — Smoke test con Meta Ads *(2 semanas, 100-300 €)*

**Ataca directamente el riesgo nº 1** (§9): saber si el mercado hispanohablante
existe. Por vía orgánica esa respuesta tarda 6 meses; aquí se compra en dos
semanas. No se construye producto: sólo una landing y las creatividades salidas
del backtest de la Fase 0.

| Bloque | Gasto | Pregunta que responde |
|---|---|---|
| **A — Política** | ~30 € | ¿Meta aprueba los anuncios o restringe la cuenta? *(§3.5)* Binario y barato. **Si falla, para: el canal no existe.** |
| **B — Demanda** | ~150 € | ¿Le interesa a alguien? Landing + 3 creatividades en **Reels** (CPM 3-5× menor), optimizando a registro gratuito. |
| **C — Disposición a pagar** | ~50 € | Página de precio con *fake door*: se muestra el precio y se mide quién pulsa «suscribirme», declarando con honestidad que aún no está disponible. Nunca se cobra ni se engaña. |

**Criterios de decisión sobre el coste por registro gratuito:**

| Resultado | Lectura | Acción |
|---|---|---|
| **< 3 €** | Señal fuerte de demanda | Continuar; el paid social incluso podría reconsiderarse a futuro con precio más alto |
| **3-8 €** | Marginal | Continuar **sólo por vía orgánica**; el pago no es rentable |
| **> 8 €** | Sin demanda apreciable | Replantear el nicho o cancelar |

> **Salvedad estadística:** con 100-300 € la muestra es pequeña (decenas de
> registros). El resultado es **direccional, no concluyente**. Sirve para
> detectar un «no» rotundo o un «sí» claro; no para afinar previsiones.

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
| **Saber si el proyecto tiene sentido** *(Fases 0 y 0.5)* | **Mes 1-2** |
| Primer euro | Mes 4-6 |
| ~100 € / mes | Mes 8-12 |
| ~500 € / mes | Mes 12-18 *(si funciona)* |

La incorporación del smoke test no acelera los ingresos, pero **adelanta
enormemente el momento de saber si merece la pena seguir**: la decisión de
abandonar (o insistir) pasa del mes 6 al mes 2, por 100-300 €. Ése es el
verdadero retorno de ese gasto.

Cualquiera que prometa plazos de ingresos más cortos partiendo de cero
audiencia está vendiendo humo.

---

## 9. Riesgos, ordenados por gravedad

| # | Riesgo | Mitigación |
|---|---|---|
| 1 | **El mercado hispanohablante es demasiado pequeño.** España + LatAm son una fracción despreciable del tráfico de OpenInsider. Un nicho vacío puede estarlo por desatendido *o por no haber dinero.* No se puede distinguir con los datos disponibles. | **Fase 0.5 lo mide en 2 semanas por 100-300 €**, en vez de esperar 6 meses a que el SEO responda. Sigue siendo el riesgo principal, pero ahora es barato de acotar. |
| 1b | **Meta restringe la cuenta publicitaria.** «Finance and Insurance» es Special Ad Category en la UE y «signal selling» figura como prohibido (§3.5). | Bloque A de la Fase 0.5 lo prueba por ~30 €. El canal orgánico no depende de esto, por lo que un bloqueo retrasa la validación pero no mata el proyecto. |
| 2 | **El backtest sale plano.** | Fase 0 lo detecta en 3 semanas, antes de construir nada. |
| 2b | **La licencia de datos de mercado resulta cara.** Verificado: el display comercial más barato con precio publicado es **EODHD, 399 €/mes** — de 3 a 9 veces mi estimación inicial. | **Resuelto por diseño, no por presupuesto:** arquitectura de tres capas (§5.5) que evita la licencia de display. EDGAR + widget de TradingView + licencia *non-display* de 11-46 €/mes. Riesgo residual: que TradingView cambie las condiciones de sus widgets; mitigable volviendo al enlace externo. |
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
| Paid social | No contemplado | **Validación acotada (100-300 €), no canal de crecimiento** |
| Criterios de abandono | Ninguno | **Explícitos en cada fase** |

---

## 11. Siguiente paso concreto

**Fase 0, backtest.** Sigue siendo la única tarea que importa ahora, y el orden
no cambia por incorporar Meta Ads: el backtest **produce las creatividades y
las cifras** que hacen creíble el anuncio de la Fase 0.5. Sin él, el smoke test
no tiene nada que decir. Concretamente:

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
