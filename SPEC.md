# INSIDERPULSE — ESPECIFICACIÓN TÉCNICA MAESTRA

> ## ⚠️ DOCUMENTO SUPERADO — LEER PRIMERO `ESTRATEGIA.md`
>
> Esta v2.0 fue escrita **antes** del replanteamiento estratégico del proyecto.
> El documento vigente es **[`ESTRATEGIA.md`](./ESTRATEGIA.md)**, que sustituye
> las decisiones de producto, algoritmo, precio y orden de ejecución.
>
> **Qué de este documento sigue vigente:**
> - §0 — tabla de correcciones de seguridad y lógica sobre la v1.0.
> - §5 — **esquema SQL completo**: `current_tier()`, vistas, revocación de
>   acceso directo. Es la referencia técnica de la regla «el permiso se decide
>   en Postgres, nunca en la aplicación».
> - §5.2 — tests de verificación de RLS.
> - §6.1 — derivación del tier en servidor a partir del JWT.
> - §11 — cumplimiento legal.
>
> **Qué ha quedado obsoleto:**
> - El algoritmo de scoring con pesos fijos 35/30/25/10 → sustituido por un
>   modelo calibrado con backtest y centrado en la clasificación
>   rutinario/oportunista (Cohen-Malloy-Pomorski 2012).
> - Los tres niveles de precio y el «precio ancla» de 39 € → un solo plan.
> - La ofuscación con `***` en el plan gratuito.
> - «Tiempo real» como propuesta de valor → la ventana real es de ~15 h.
> - El plan de fases → sustituido por uno con criterios de abandono explícitos.

---

> **Versión 2.0** · Documento de arquitectura, ingeniería de datos, lógica algorítmica, modelo de negocio y plan de ejecución.
>
> Esta revisión corrige cuatro fallos críticos de seguridad y cuatro de lógica detectados en la v1.0. Los cambios están marcados con **`[FIX v2]`** para trazabilidad.

---

## 0. RESUMEN DE CORRECCIONES RESPECTO A LA v1.0

| # | Severidad | Problema en v1.0 | Corrección en v2.0 |
| :-- | :-- | :-- | :-- |
| 1 | 🔴 Crítico | RLS `USING (true)` sobre `insider_trades` + cliente Supabase en navegador → cualquiera leía el feed completo con la `anon key` | Acceso directo revocado. Se expone únicamente la vista `trades_feed`, que aplica retraso y ofuscación **en SQL** |
| 2 | 🔴 Crítico | `user_tier` era un parámetro de query string → `?user_tier=pro` daba acceso total sin login | El tier se deriva del JWT verificado en servidor mediante `current_tier()` |
| 3 | 🔴 Crítico | `accession_number UNIQUE` → un Form 4 con varias transacciones perdía filas | Clave natural `UNIQUE (accession_number, transaction_seq)` |
| 4 | 🔴 Crítico | Sin `.gitignore` en repositorio público con `SERVICE_ROLE_KEY` en `.env` | `.gitignore` creado como primer artefacto del proyecto |
| 5 | 🟠 Alto | `is_10b5_1` se almacenaba pero el algoritmo lo ignoraba | Las operaciones bajo plan preestablecido se excluyen del scoring |
| 6 | 🟠 Alto | `cluster_count_15d` congelado en la ingesta → los primeros compradores de un cluster nunca puntuaban | Recálculo de la ventana móvil en cada ciclo + alerta por *cluster completado* |
| 7 | 🟠 Alto | Free sufría retraso de 24 h **y** ofuscación → 5 filas de `***`, producto inservible | Mecanismos separados: feed histórico completo a 24 h + franja *live* bloqueada |
| 8 | 🟠 Alto | Cuatro umbrales distintos (50 / 60 / 70 / 75) | Umbral único de alerta: **75** |
| 9 | 🟠 Alto | Sin cobertura legal (RGPD, MAR, IVA) y mockup con directivo real identificable | Sección 11 de cumplimiento + ejemplos ficticios obligatorios |
| 10 | 🟠 Alto | `ticker` e `insider_title` como `NOT NULL` → la ingesta rompe en producción | Columnas `NULL`-ables con resolución diferida |
| 11 | 🟡 Medio | Pesos del algoritmo afirmados sin validación empírica | Fase 0 de backtesting previa a la comercialización |

---

## 1. FUNDAMENTACIÓN FINANCIERA Y PROPUESTA DE VALOR

### 1.1. Ineficiencia de los precios objetivo del *sell-side*

* **Baja tasa de acierto:** los precios objetivo del consenso de analistas a 12 meses se cumplen entre el **35 % y el 40 %** de las veces (Bradshaw, Brown, Asquith), con error medio absoluto del **40-46 %**.
* **Sesgo comercial estructural:** menos del 6 % de las recomendaciones son de venta, por conflicto de interés con las divisiones de banca corporativa.
* **Efecto rezagado:** ante un deterioro operativo, los analistas tardan meses en recortar valoraciones, inflando artificialmente el *upside* publicado en agregadores.

### 1.2. La señal: compras de directivos en mercado abierto

* **Asimetría de motivaciones:** un directivo vende por razones diversas (impuestos, liquidez, diversificación), pero **compra con dinero propio por una sola razón**.
* **Marco regulatorio:** la Sección 16 de la *Securities Exchange Act* y la *Sarbanes-Oxley Act* obligan a declarar la operación en el **Formulario 4** dentro de **2 días hábiles (T+2)**.
* **Misión:** ingerir cada Form 4, descartar el ruido administrativo y aplicar un **score determinista 0-100** que priorice las operaciones de mayor convicción.

### 1.3. `[FIX v2]` Advertencia de validación pendiente

> Los pesos del algoritmo (35 / 30 / 25 / 10) son una **hipótesis razonada, no un resultado validado**. La literatura académica (Lakonishok & Lee 2001; Jeng, Metrick & Zeckhauser 2003) respalda la existencia de alfa en el *insider buying*, pero **no** esta ponderación concreta.
>
> **La Fase 0 (§13) debe completarse antes de comercializar el producto.** Vender ventaja cuantitativa sin backtest es un riesgo reputacional y potencialmente regulatorio.

### 1.4. `[FIX v2]` Sesgo conocido del filtro de importe

El suelo de 50 000 $ excluye sistemáticamente *small caps*, donde la literatura sitúa el alfa **más intenso**. Es una decisión consciente de reducción de ruido, pero introduce sesgo hacia *large caps*, donde el efecto es más débil.

**Mitigación prevista (post-MVP):** sustituir el umbral absoluto por uno relativo — `total_value / market_cap` — cuando exista fuente gratuita y fiable de capitalización.

---

## 2. ARQUITECTURA TÉCNICA DE COSTE 0 €

```
                        ┌────────────────────────────┐
                        │      SEC EDGAR Feed        │
                        │   Atom RSS + XML Form 4    │
                        └─────────────┬──────────────┘
                                      │ polling
                                      ▼
                        ┌────────────────────────────┐
                        │   Worker de ingesta        │
                        │   (ver §2.1 — decisión)    │
                        └─────────────┬──────────────┘
                                      │
              ┌───────────────────────┼───────────────────────┐
              ▼                       ▼                       ▼
   ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐
   │ Supabase Postgres  │  │   Telegram Bot     │  │  Meta CAPI server  │
   │  + Auth + RLS      │  │  Canal Free / VIP  │  │  (dedup con Pixel) │
   └─────────┬──────────┘  └────────────────────┘  └────────────────────┘
             │
             │  SOLO vistas — acceso directo a tablas revocado  [FIX v2]
             ▼
   ┌────────────────────┐        ┌────────────────────┐
   │  Next.js / Vercel  │───────▶│   Stripe Billing   │
   └────────────────────┘        └────────────────────┘
```

| Capa | Servicio | Coste |
| :-- | :-- | :-- |
| Base de datos + Auth | Supabase Free (PostgreSQL 500 MB, RLS, JWT) | 0 € |
| Fuente de datos | SEC EDGAR (dominio público) | 0 € |
| Backend | Python 3.11+, FastAPI, `httpx`, `defusedxml`, `FastMCP` | 0 € |
| Frontend | Next.js App Router + Tailwind + shadcn/ui en Vercel Hobby | 0 € |
| Pagos | Stripe Checkout + Portal + Webhooks | 0 € fijo |
| Alertas | Telegram Bot API | 0 € |
| Email | Resend Free (3 000/mes) | 0 € |
| Tracking | Meta Pixel + CAPI | 0 € |

**Estimación de volumen:** ~450 000 Form 4/año en EDGAR. Tras filtrar `P` + `D` + ≥ 5 $ + ≥ 50 000 $ quedan **15 000-25 000 filas/año ≈ 25 MB**. Los 500 MB de Supabase son holgados durante años.

### 2.1. `[FIX v2]` Decisión sobre el worker de ingesta

La v1.0 daba por sentado GitHub Actions cron cada 10 min. **Esto entra en conflicto directo con la promesa de "tiempo real":**

* Los `schedule:` de GitHub Actions son **best-effort, sin SLA**. Retrasos habituales de 5-30 min bajo carga, y ejecuciones omitidas.
* GitHub **desactiva automáticamente los workflows programados tras 60 días sin commits** en el repositorio. Un servicio en producción se apagaría solo.
* El repositorio es **público** (requisito para minutos ilimitados), lo que implica que `scoring.py` es copiable por cualquiera.

**Opciones evaluadas:**

| Opción | Latencia | Coste | Observaciones |
| :-- | :-- | :-- | :-- |
| **A** · GitHub Actions cron | 10-40 min, sin garantía | 0 € | Requiere repo público; riesgo de autodesactivación |
| **B** · Supabase `pg_cron` + `pg_net` | ~1 min, fiable | 0 € | Incluido en Free Tier; lógica de ingesta en la BD |
| **C** · Cloud Run + Cloud Scheduler | < 1 min, con SLA | 0 € dentro de free tier | Más piezas, pero es el único con garantía real |

**Decisión para el MVP: opción A, con copy honesto.** El material comercial dirá **"alertas en menos de 15 minutos"**, no "tiempo real instantáneo". Sigue siendo una ventaja enorme frente a las 24 h del plan gratuito, y es una afirmación defendible.

**Migración a opción C** cuando exista el primer cliente de pago recurrente. La promesa comercial no debe superar nunca a la garantía técnica.

**Mitigación de la autodesactivación:** workflow adicional `keepalive.yml` mensual que hace *commit* de un fichero de latido, o `workflow_dispatch` manual documentado en el runbook.

---

## 3. VARIABLES DE ENTORNO

> ⚠️ **El repositorio es público.** `.gitignore` excluye `.env` y `.env.*` salvo `.env.example`. **Nunca** sustituyas los marcadores de posición por valores reales en este fichero.
>
> `SUPABASE_SERVICE_ROLE_KEY` **salta todas las políticas RLS**. Solo puede vivir en los *secrets* del worker, jamás en el frontend ni en variables con prefijo `NEXT_PUBLIC_`.

```env
# --- SEC EDGAR (formato de User-Agent obligatorio: la SEC devuelve 403 sin él) ---
SEC_USER_AGENT="InsiderPulse admin@tudominio.com"
SEC_MAX_RPS=8                       # límite oficial 10 req/s; margen de seguridad

# --- SUPABASE ---
NEXT_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=****  # público por diseño; solo accede a vistas
SUPABASE_SERVICE_ROLE_KEY=****      # SOLO worker. Salta RLS. Nunca en el cliente
DATABASE_URL=****

# --- STRIPE ---
STRIPE_SECRET_KEY=sk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...     # verificación de firma obligatoria
STRIPE_PRICE_ID_VIP_MONTHLY=price_...
STRIPE_PRICE_ID_VIP_YEARLY=price_...
STRIPE_PRICE_ID_PRO_MONTHLY=price_...
STRIPE_PRICE_ID_PRO_YEARLY=price_...

# --- TELEGRAM ---
TELEGRAM_BOT_TOKEN=****
TELEGRAM_FREE_CHANNEL_ID=-100...
TELEGRAM_VIP_CHANNEL_ID=-100...

# --- EMAIL ---
RESEND_API_KEY=****
EMAIL_FROM="InsiderPulse <alertas@tudominio.com>"

# --- META ADS ---
NEXT_PUBLIC_META_PIXEL_ID=****
META_CAPI_ACCESS_TOKEN=****
META_TEST_EVENT_CODE=TEST12345

# --- GENERAL ---
NEXT_PUBLIC_APP_URL=http://localhost:3000
ALERT_SCORE_THRESHOLD=75            # umbral único [FIX v2]
FREE_TIER_DELAY_HOURS=24
CLUSTER_WINDOW_DAYS=15
```

---

## 4. ALGORITMO DETERMINISTA DE SCORING (0-100)

### 4.1. Filtros de admisión

Se procesa una transacción **solo si cumple todas** estas condiciones:

| Filtro | Condición | Motivo |
| :-- | :-- | :-- |
| Código | `transactionCode == 'P'` | Compra en mercado abierto. Se descartan `A`, `M`, `G`, `S`, `F`, `C` |
| Propiedad | `directOrIndirectOwnership == 'D'` | Propiedad directa; se excluyen vehículos y trusts |
| Precio | `transactionPricePerShare >= 5.00` | Evita *penny stocks* y microcaps manipulables |
| Importe | `transactionShares * price >= 50 000` | Elimina compras testimoniales |
| **Plan 10b5-1** | **`is_10b5_1 == FALSE`** | **`[FIX v2]`** Ver §4.2 |

### 4.2. `[FIX v2]` Exclusión de planes 10b5-1 — justificación

Una compra ejecutada bajo un **plan preestablecido 10b5-1** se programó con meses de antelación. **No refleja convicción sobre la valoración actual**: es un calendario automático.

Incluirlas contradice la tesis central del producto (*"solo compran cuando está infravalorada"*) y contamina la señal. El Form 4 las identifica mediante la casilla del pie del formulario.

En la v1.0 la columna `is_10b5_1` existía en el esquema pero **el algoritmo nunca la consultaba** — se almacenaba dato muerto mientras estas operaciones puntuaban igual que una compra discrecional.

Las operaciones 10b5-1 **se persisten** (valor analítico e histórico) con `score = NULL` y `excluded_reason = '10b5-1'`, pero **no puntúan ni generan alertas**.

### 4.3. Fórmula

```
Score = P_Rol + P_Volumen + P_Cluster + P_Incremento        (máx. 100)
```

**1 · `P_Rol` — Rol del directivo (máx. 35)**

Los indicadores del Form 4 (`isDirector`, `isOfficer`, `isTenPercentOwner`) **no son excluyentes** y `officerTitle` es **texto libre**. Se aplica normalización con precedencia y **se asigna la puntuación mayor, nunca la suma**.

| Rol normalizado | Puntos |
| :-- | --: |
| CEO / Chief Executive Officer | 35 |
| CFO / Chief Financial Officer | 35 |
| COO / President / Vice-Chair / Director General | 25 |
| Director (consejero) / VP / General Counsel | 15 |
| Accionista > 10 % sin cargo ejecutivo | 5 |

**Normalización obligatoria de `officerTitle`** — el título llega como texto libre y es la primera fuente de error de clasificación. Variantes reales frecuentes:

```
"CEO" · "Chief Executive Officer" · "President and CEO" · "CEO & Chairman"
"Interim CFO" · "EVP & Chief Financial Officer" · "Principal Financial Officer"
```

Se implementa en `scoring.py` mediante lista ordenada de patrones regex insensibles a mayúsculas, evaluada de mayor a menor precedencia. **Todo título no reconocido se registra en `unmapped_titles` para revisión manual** y puntúa según los booleanos (`isDirector` → 15, `isTenPercentOwner` → 5).

**2 · `P_Volumen` — Importe (máx. 30)**

| Importe | Puntos |
| :-- | --: |
| ≥ 500 000 $ | 30 |
| 100 000 – 499 999 $ | 20 |
| 50 000 – 99 999 $ | 10 |

**3 · `P_Cluster` — Compras agrupadas (máx. 25)**

Número de directivos **únicos** (por `insider_cik`) con compras `P` admitidas sobre el **mismo emisor** en una ventana móvil de **15 días naturales** centrada en `transaction_date`.

| Directivos únicos | Puntos |
| :-- | --: |
| ≥ 3 | 25 |
| 2 | 15 |
| 1 | 0 |

**4 · `P_Incremento` — Aumento de posición (máx. 10)**

```
prevShares = sharesOwnedFollowingTransaction − transactionShares
ratio      = transactionShares / prevShares          (solo si prevShares > 0)
```

| Condición | Puntos |
| :-- | --: |
| `ratio ≥ 0.20` | 10 |
| `0.05 ≤ ratio < 0.20` | 5 |
| `ratio < 0.05` | 0 |
| `prevShares == 0` (primera compra) | 0 — ver nota |

> **Nota para la Fase 0:** asignar 0 puntos a la primera compra absoluta de un directivo es contraintuitivo — suele considerarse una señal potente. Se mantiene en 0 en el MVP por prudencia, pero **es una de las hipótesis prioritarias a contrastar en el backtest**. Si el histórico lo respalda, pasará a 10 puntos.
>
> `prevShares` puede ser negativo por datos inconsistentes de EDGAR: tratar como `0` y registrar anomalía.

### 4.4. `[FIX v2]` Recálculo del cluster — el score es dinámico

En la v1.0 `cluster_count_15d` se fijaba en el momento de la inserción. Como **un cluster se forma con el tiempo**, esto infravalora sistemáticamente a los compradores más tempranos, que son precisamente la señal más valiosa:

| Día | Evento | Cluster v1.0 (congelado) | Cluster v2.0 (recalculado) |
| :-- | :-- | :-- | :-- |
| 1 | Compra el CFO | **0 pts para siempre** | 0 → 15 → **25** |
| 6 | Compra el CEO | 15 pts | 15 → **25** |
| 12 | Compra un consejero | 25 pts | **25** |

Peor aún: el evento de mayor poder predictivo —**el cluster completándose**— nunca generaba alerta.

**Procedimiento en cada ciclo de ingesta:**

1. Insertar las transacciones nuevas con score provisional.
2. Recalcular `cluster_count_15d` y `score` de **todas** las operaciones cuyo `transaction_date` esté dentro de la ventana de 15 días de cualquier emisor afectado.
3. Actualizar `score_updated_at`.
4. **Emitir alerta de tipo `cluster_completed`** para toda operación que cruce el umbral de 75 como consecuencia del recálculo, siempre que no exista ya un registro en `sent_alerts` para ese `(trade_id, target)`.

Esto convierte el sistema en detector de clusters emergentes, no solo de compras individuales.

### 4.5. `[FIX v2]` Umbral único

La v1.0 manejaba cuatro cortes distintos (badges en 75/50, Telegram Free en 70, VIP en 60), de modo que el canal gratuito anunciaba como *"oportunidad destacada"* operaciones que la propia interfaz etiquetaba como `MODERADA`.

**Un único umbral de alerta: `score >= 75`.**

| Rango | Etiqueta | Color | Comportamiento |
| :-- | :-- | :-- | :-- |
| 75-100 | `ALTA CONVICCIÓN` | `#10B981` esmeralda | Alerta instantánea VIP + resumen diario Free |
| 50-74 | `MODERADA` | `#F59E0B` ámbar | Visible en la web; sin alerta push |
| 0-49 | `BAJA` | `#71717A` gris | Visible en la web |

---

## 5. ESQUEMA DE BASE DE DATOS

`supabase/migrations/001_initial_schema.sql`

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- ═══════════════════════════════════════════════════════════════════
-- 1. PERFILES
-- ═══════════════════════════════════════════════════════════════════
CREATE TABLE public.profiles (
    id                        UUID PRIMARY KEY REFERENCES auth.users ON DELETE CASCADE,
    email                     TEXT UNIQUE NOT NULL,

    telegram_chat_id          BIGINT UNIQUE,
    telegram_link_token       TEXT UNIQUE,
    telegram_token_expires_at TIMESTAMPTZ,          -- [FIX v2] token con caducidad
    telegram_invite_link      TEXT,                 -- [FIX v2] enlace de un solo uso, revocable

    stripe_customer_id        TEXT UNIQUE,
    stripe_subscription_id    TEXT UNIQUE,
    plan_tier                 TEXT NOT NULL DEFAULT 'free'
                              CHECK (plan_tier IN ('free','vip','pro')),
    billing_period            TEXT CHECK (billing_period IN ('monthly','yearly')),
    subscription_status       TEXT NOT NULL DEFAULT 'inactive'
                              CHECK (subscription_status IN
                                    ('inactive','active','trialing','canceled','past_due')),
    subscription_end_date     TIMESTAMPTZ,

    -- [FIX v2] Consentimiento RGPD granular (§11)
    tracking_consent          BOOLEAN NOT NULL DEFAULT FALSE,
    marketing_consent         BOOLEAN NOT NULL DEFAULT FALSE,
    consent_updated_at        TIMESTAMPTZ,

    created_at                TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at                TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ═══════════════════════════════════════════════════════════════════
-- 2. OPERACIONES DE DIRECTIVOS
-- ═══════════════════════════════════════════════════════════════════
CREATE TABLE public.insider_trades (
    id                   BIGSERIAL PRIMARY KEY,

    -- [FIX v2] Un Form 4 contiene N transacciones. La clave natural es compuesta;
    -- con UNIQUE solo sobre accession_number se perdían filas silenciosamente.
    accession_number     TEXT     NOT NULL,
    transaction_seq      SMALLINT NOT NULL DEFAULT 0,

    -- [FIX v2] Enmiendas Form 4/A: corrigen o anulan operaciones previas
    is_amendment         BOOLEAN  NOT NULL DEFAULT FALSE,
    amends_accession     TEXT,
    is_superseded        BOOLEAN  NOT NULL DEFAULT FALSE,

    -- [FIX v2] ticker NULL-able: el Form 4 entrega CIK, no ticker. La resolución
    -- falla en emisores extranjeros, deslistados o sin mapeo en company_tickers.json
    issuer_cik           TEXT NOT NULL,
    ticker               TEXT,
    company_name         TEXT NOT NULL,

    insider_cik          TEXT NOT NULL,
    insider_name         TEXT NOT NULL,
    insider_title        TEXT,                       -- [FIX v2] vacío en >10 % owners
    role_normalized      TEXT,                       -- resultado de la normalización §4.3
    is_officer           BOOLEAN NOT NULL DEFAULT FALSE,
    is_director          BOOLEAN NOT NULL DEFAULT FALSE,
    is_ten_percent_owner BOOLEAN NOT NULL DEFAULT FALSE,

    transaction_date     DATE        NOT NULL,
    filing_date          TIMESTAMPTZ NOT NULL,       -- EDGAR opera en horario del Este (ET)

    transaction_shares   NUMERIC(20,4) NOT NULL CHECK (transaction_shares > 0),
    transaction_price    NUMERIC(20,4) NOT NULL CHECK (transaction_price  > 0),
    total_value          NUMERIC(20,2) NOT NULL,
    shares_owned_after   NUMERIC(20,4),
    pct_increase         NUMERIC(10,4),

    is_10b5_1            BOOLEAN NOT NULL DEFAULT FALSE,
    cluster_count_15d    SMALLINT NOT NULL DEFAULT 1,

    -- [FIX v2] score NULL = operación persistida pero excluida del scoring
    score                SMALLINT CHECK (score BETWEEN 0 AND 100),
    score_breakdown      JSONB,
    score_updated_at     TIMESTAMPTZ,
    excluded_reason      TEXT,

    sec_xml_url          TEXT NOT NULL,
    created_at           TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_trade UNIQUE (accession_number, transaction_seq)
);

-- ═══════════════════════════════════════════════════════════════════
-- 3. ALERTAS DESPACHADAS
-- ═══════════════════════════════════════════════════════════════════
CREATE TABLE public.sent_alerts (
    id         BIGSERIAL PRIMARY KEY,
    trade_id   BIGINT NOT NULL REFERENCES public.insider_trades(id) ON DELETE CASCADE,
    target     TEXT   NOT NULL CHECK (target IN ('telegram_free','telegram_vip','email')),
    alert_kind TEXT   NOT NULL DEFAULT 'new_trade'
               CHECK (alert_kind IN ('new_trade','cluster_completed')),
    sent_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- [FIX v2] Sin esta restricción, cada reejecución del cron duplicaba alertas
    CONSTRAINT uq_alert UNIQUE (trade_id, target, alert_kind)
);

-- ═══════════════════════════════════════════════════════════════════
-- 4. TÍTULOS SIN MAPEAR (control de calidad del normalizador)
-- ═══════════════════════════════════════════════════════════════════
CREATE TABLE public.unmapped_titles (
    id         BIGSERIAL PRIMARY KEY,
    raw_title  TEXT NOT NULL UNIQUE,
    seen_count INT  NOT NULL DEFAULT 1,
    first_seen TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_seen  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ═══════════════════════════════════════════════════════════════════
-- 5. ÍNDICES
-- ═══════════════════════════════════════════════════════════════════
CREATE INDEX idx_trades_score_date ON public.insider_trades (score DESC, transaction_date DESC)
    WHERE score IS NOT NULL AND is_superseded = FALSE;
CREATE INDEX idx_trades_ticker     ON public.insider_trades (ticker) WHERE ticker IS NOT NULL;
CREATE INDEX idx_trades_filing     ON public.insider_trades (filing_date DESC);
CREATE INDEX idx_trades_cluster    ON public.insider_trades (issuer_cik, transaction_date);
CREATE INDEX idx_profiles_stripe   ON public.profiles (stripe_customer_id);
CREATE INDEX idx_profiles_telegram ON public.profiles (telegram_chat_id);
```

### 5.1. `[FIX v2]` Control de acceso — la corrección más importante

En la v1.0, `CREATE POLICY ... USING (true)` sobre `insider_trades`, combinado con el cliente Supabase en el navegador, hacía que **toda la lógica de negocio fuese evadible desde la consola del navegador**:

```js
// Con la anon key visible en el bundle, esto devolvía el feed COMPLETO en tiempo real
await supabase.from('insider_trades').select('*').gte('score', 75)
```

**Principio de la v2.0: la regla de negocio vive en Postgres, no en Python.** El acceso directo a la tabla se revoca por completo y solo se exponen vistas que aplican retraso y ofuscación en SQL.

```sql
-- ═══════════════════════════════════════════════════════════════════
-- 6. TIER EFECTIVO DERIVADO DEL JWT — nunca del cliente  [FIX v2]
-- ═══════════════════════════════════════════════════════════════════
CREATE OR REPLACE FUNCTION public.current_tier()
RETURNS TEXT
LANGUAGE sql STABLE SECURITY DEFINER SET search_path = public
AS $$
    SELECT COALESCE(
        (SELECT plan_tier FROM public.profiles
          WHERE id = auth.uid()
            AND subscription_status IN ('active','trialing')
            AND (subscription_end_date IS NULL OR subscription_end_date > NOW())),
        'free');
$$;

-- ═══════════════════════════════════════════════════════════════════
-- 7. RLS Y REVOCACIÓN DE ACCESO DIRECTO
-- ═══════════════════════════════════════════════════════════════════
ALTER TABLE public.insider_trades ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.sent_alerts    ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.profiles       ENABLE ROW LEVEL SECURITY;

-- Sin políticas SELECT para anon/authenticated => acceso denegado por defecto.
-- El worker usa SERVICE_ROLE_KEY, que opera al margen de RLS.
REVOKE ALL ON public.insider_trades  FROM anon, authenticated;
REVOKE ALL ON public.sent_alerts     FROM anon, authenticated;
REVOKE ALL ON public.unmapped_titles FROM anon, authenticated;

-- El usuario solo accede a su propio perfil
CREATE POLICY "perfil propio: lectura"      ON public.profiles
    FOR SELECT USING (auth.uid() = id);
CREATE POLICY "perfil propio: actualización" ON public.profiles
    FOR UPDATE USING (auth.uid() = id)
    WITH CHECK (auth.uid() = id);

-- ═══════════════════════════════════════════════════════════════════
-- 8. VISTA PRINCIPAL — la regla de tier se aplica aquí  [FIX v2]
-- ═══════════════════════════════════════════════════════════════════
-- security_invoker = off (valor por defecto): la vista se ejecuta con los
-- privilegios de su propietario, que sí puede leer la tabla base.
CREATE OR REPLACE VIEW public.trades_feed AS
SELECT
    t.id,
    t.ticker,
    t.company_name,
    t.insider_name,
    t.insider_title,
    t.role_normalized,
    t.transaction_date,
    t.filing_date,
    t.transaction_shares,
    t.transaction_price,
    t.total_value,
    t.pct_increase,
    t.cluster_count_15d,
    t.score,
    t.score_breakdown,
    t.sec_xml_url,
    FALSE AS is_locked
FROM public.insider_trades t
WHERE t.score IS NOT NULL
  AND t.is_superseded = FALSE
  AND (
        public.current_tier() <> 'free'
        OR t.filing_date <= NOW() - INTERVAL '24 hours'   -- retraso solo para Free
      );

-- ═══════════════════════════════════════════════════════════════════
-- 9. FRANJA "LIVE" — el mecanismo de FOMO real  [FIX v2]
-- ═══════════════════════════════════════════════════════════════════
-- En la v1.0 el plan Free sufría retraso de 24 h Y ofuscación simultáneos.
-- Como el listado ordenaba por score descendente y se ofuscaba todo lo ≥75,
-- el usuario gratuito veía cinco filas de "***": un muro, no un teaser.
--
-- v2.0 separa ambos mecanismos:
--   · trades_feed  → histórico COMPLETO y legible, con 24 h de retraso
--   · trades_live  → lo que está ocurriendo AHORA, ofuscado para Free
CREATE OR REPLACE VIEW public.trades_live AS
SELECT
    t.id,
    CASE WHEN public.current_tier() = 'free' THEN '***' ELSE t.ticker END       AS ticker,
    CASE WHEN public.current_tier() = 'free'
         THEN '*** Desbloquear con VIP' ELSE t.company_name END                 AS company_name,
    t.role_normalized,          -- "CEO" se muestra siempre: es el gancho
    t.total_value,              -- el importe también: refuerza la urgencia
    t.score,
    t.filing_date,
    (public.current_tier() = 'free') AS is_locked
FROM public.insider_trades t
WHERE t.score >= 75
  AND t.is_superseded = FALSE
  AND t.filing_date > NOW() - INTERVAL '24 hours';

GRANT SELECT ON public.trades_feed TO anon, authenticated;
GRANT SELECT ON public.trades_live TO anon, authenticated;

-- ═══════════════════════════════════════════════════════════════════
-- 10. TRIGGER updated_at
-- ═══════════════════════════════════════════════════════════════════
CREATE OR REPLACE FUNCTION public.touch_updated_at()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN NEW.updated_at = NOW(); RETURN NEW; END; $$;

CREATE TRIGGER trg_profiles_touch BEFORE UPDATE ON public.profiles
    FOR EACH ROW EXECUTE FUNCTION public.touch_updated_at();
```

### 5.2. Verificación obligatoria antes de desplegar

```sql
-- Ejecutar como rol `anon`. Las dos primeras DEBEN fallar con "permission denied".
SET ROLE anon;
SELECT * FROM public.insider_trades LIMIT 1;              -- ❌ debe denegar
SELECT ticker FROM public.insider_trades WHERE score>=75; -- ❌ debe denegar
SELECT * FROM public.trades_feed LIMIT 5;                 -- ✅ solo filas de +24 h
SELECT * FROM public.trades_live LIMIT 5;                 -- ✅ ticker = '***'
RESET ROLE;
```

> Añadir estas comprobaciones como test de integración en CI. Una regresión aquí regala el producto completo.

---

## 6. CAPA DE API

### 6.1. `[FIX v2]` Derivación del tier en servidor

La v1.0 contenía un fallo de autenticación crítico:

```python
# ❌ v1.0 — `user_tier` es un parámetro de QUERY STRING en FastAPI.
#          GET /api/trades?user_tier=pro daba acceso total sin autenticación.
async def get_trades(user_tier: str = "free", db: AsyncSession = Depends(get_db)):
```

```python
# ✅ v2.0 — el tier procede exclusivamente del JWT verificado en servidor
from fastapi import APIRouter, Depends, Query
from app.core.security import get_current_profile, Profile

router = APIRouter()

VALID_STATUSES = {"active", "trialing"}

def effective_tier(profile: Profile | None) -> str:
    """Fuente única de verdad del tier. Jamás se lee de la petición."""
    if profile is None:
        return "free"
    if profile.subscription_status not in VALID_STATUSES:
        return "free"
    if profile.subscription_end_date and profile.subscription_end_date < utcnow():
        return "free"
    return profile.plan_tier


@router.get("/api/trades")
async def get_trades(
    profile: Profile | None = Depends(get_current_profile),
    limit: int = Query(50, le=200),
):
    tier = effective_tier(profile)
    # La vista aplica retraso y ofuscación en SQL; el límite de filas es
    # la única regla que permanece en la capa de aplicación.
    row_cap = 5 if tier == "free" else limit
    return await fetch_trades_feed(as_user=profile, limit=row_cap)
```

> **Defensa en profundidad:** aunque `routes_trades.py` tuviera un fallo, la vista SQL seguiría impidiendo que un usuario Free viese datos de menos de 24 h. Ninguna de las dos capas es suficiente por sí sola.

### 6.2. Webhooks de Stripe

* **Verificar siempre la firma** con `STRIPE_WEBHOOK_SECRET` antes de procesar. Un webhook sin verificar permite a cualquiera concederse plan Pro.
* `subscription_status` y `plan_tier` en `profiles` **solo** se escriben desde el manejador del webhook, nunca desde el cliente.
* Eventos: `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`, `invoice.payment_failed`.
* Idempotencia: registrar `event.id` procesados; Stripe reintenta entregas.

---

## 7. MODELO DE PRECIOS

| Parámetro | **Free (Radar)** | **Investor VIP** | **Quant Pro** |
| :-- | :-- | :-- | :-- |
| **Precio** | 0 € | **9 €/mes** · 69 €/año | 39 €/mes · 299 €/año |
| **Rol** | Captación y SEO | **Objetivo de conversión** | Ancla de precio |
| **Histórico** | Completo, **retraso 24 h** | Inmediato | Inmediato |
| **Franja live** | Bloqueada (`$***`) | Desbloqueada | Desbloqueada |
| **Filas por consulta** | 5 | Sin límite | Sin límite |
| **Telegram** | Resumen diario diferido | **Canal VIP inmediato** | Canal VIP + bot 1-a-1 |
| **Herramientas** | — | Filtros y búsqueda | Export CSV/JSON + API |

### 7.1. `[FIX v2]` Rediseño del mecanismo de conversión

En la v1.0 el plan Free ordenaba por score descendente **y** ofuscaba todo lo ≥ 75, de modo que las cinco filas visibles eran precisamente las bloqueadas: **una pantalla completa de asteriscos**. Eso no genera curiosidad, genera rebote.

La v2.0 separa los dos mecanismos:

* **`trades_feed`** — el usuario Free ve el histórico **completo y legible** con 24 h de retraso. El producto gratuito es genuinamente útil, lo que sostiene el SEO programático y la credibilidad.
* **`trades_live`** — franja superior que muestra lo que está ocurriendo **en las últimas 24 h**, con el ticker oculto pero **el rol y el importe visibles**.

```
🔴 EN VIVO · hace 14 min
   Un CEO acaba de comprar 1 840 000 $ · Score 92/100 · $***
   → Desbloquea el ticker por 9 €/mes
```

El usuario comprueba la utilidad real del producto en el histórico *y* percibe con precisión qué se está perdiendo ahora. La aversión a la pérdida opera sobre una carencia concreta, no sobre un muro opaco.

### 7.2. Sesgos aplicados

* **Anclaje:** Quant Pro a 39 € fija el marco; VIP a 9 € se percibe como ganga (< 0,30 €/día).
* **Brecha de curiosidad:** rol e importe visibles, ticker oculto — el dato que falta es exactamente el accionable.
* **Aversión a la pérdida:** *"Esta operación se publicó hace 14 minutos. Tu plan la verá mañana."*

---

## 8. INGESTA SEC EDGAR

### 8.1. Requisitos del cliente HTTP

* **User-Agent obligatorio:** `InsiderPulse admin@tudominio.com`. La SEC responde **403** sin cabecera identificativa con contacto real.
* **Límite:** 10 req/s oficial; el sistema opera a **8 req/s** con retroceso exponencial ante `429`/`403`.
* Feed: `https://www.sec.gov/cgi-bin/browse-edgar?action=getcurrent&type=4&output=atom`
* Parseo con **`defusedxml.ElementTree`** — protección frente a XXE y *billion laughs* al procesar XML de terceros.

### 8.2. `[FIX v2]` Casos que la v1.0 no contemplaba

| Caso | Tratamiento |
| :-- | :-- |
| **N transacciones por filing** | Iterar `<nonDerivativeTransaction>`; asignar `transaction_seq` incremental |
| **Form 4/A (enmienda)** | Marcar `is_amendment`; poner `is_superseded = TRUE` en las filas del `accession_number` corregido |
| **CIK sin ticker** | Resolver vía `company_tickers.json`; si falla, `ticker = NULL` y reintentar en ciclos posteriores |
| **Zona horaria** | EDGAR opera en ET. Normalizar todo a UTC en la ingesta; mezclar husos corrompe la ventana de 15 días y el retraso de 24 h |
| **`officerTitle` desconocido** | Registrar en `unmapped_titles` y puntuar por booleanos |
| **`prevShares` negativo** | Tratar como 0 y registrar anomalía |
| **Reejecución del cron** | `INSERT ... ON CONFLICT (accession_number, transaction_seq) DO UPDATE` |

### 8.3. Servidor FastMCP

`backend/app/mcp_server.py` — expone el motor a agentes de IA:

* `fetch_latest_sec_trades(limit)` — consulta el feed oficial y devuelve Form 4 procesados.
* `score_insider_trade(trade_data)` — cálculo determinista con desglose.
* `query_database_trades(min_score, limit)` — consulta filtrada por convicción.
* `dispatch_telegram_alert(trade_id, channel_type)` — despacho manual de alerta.

---

## 9. TELEGRAM

### 9.1. Canal gratuito

Una publicación diaria a las 22:30 CET con la operación de mayor score del día (≥ 75), ya fuera de la ventana de retraso. **El ticker se muestra**: a 24 h de antigüedad no compromete la ventaja del plan VIP y funciona como demostración de valor.

```text
🔥 OPERACIÓN DESTACADA DE AYER

🏢 Empresa: ACME Robotics Corp. ($DEMO)
👤 Directivo: J. Doe (CFO)
💰 Compra: 1 250 000 $ (5 500 acciones a 227,27 $)
📈 Incremento de posición: +24,5 %
👥 Cluster: 2 directivos en los últimos 15 días

⭐️ Score: 95/100 — ALTA CONVICCIÓN

⏱️ Los suscriptores VIP recibieron esta alerta hace 24 horas.
👉 https://tudominio.com/pricing
```

> **`[FIX v2]` Los ejemplos deben usar emisores y personas ficticios.** La v1.0 empleaba *"Apple Inc. — Luca Maestri (CFO) — 1 250 000 $"*: un directivo real, identificable por nombre, con una operación inventada, en material comercial. Riesgo legal y reputacional innecesario.

### 9.2. Canal VIP

Alerta inmediata para toda operación con `score >= 75`, incluidas las de tipo `cluster_completed`. Incluye enlace al Form 4 original y desglose completo de puntos.

### 9.3. Altas y bajas

* Vinculación mediante `/start <token>`, con token de **un solo uso y caducidad de 15 minutos** (`telegram_token_expires_at`).
* **`[FIX v2]` Enlaces de invitación individuales:** `createChatInviteLink` con `member_limit=1` por usuario, almacenado en `telegram_invite_link`. Un enlace estático compartido permitiría reentrar tras la expulsión, invalidando el control de acceso.
* Al pasar la suscripción a `canceled` o `past_due`: `revokeChatInviteLink` + `banChatMember` seguido de `unbanChatMember` (permite reincorporación futura si vuelve a suscribirse).
* El bot requiere permisos de administrador en ambos canales.

---

## 10. MARKETING Y CRM

### 10.1. Meta CAPI

* Envío dual Pixel (navegador) + Graph API v19.0 (servidor) con **`event_id` idéntico** para deduplicación exacta.
* Hasheo SHA-256 de email, IP y user-agent para EMQ > 8,0/10.
* Eventos: `PageView`, `ViewContent`, `Lead`, `InitiateCheckout`, `Purchase` (disparado desde el webhook de Stripe, no desde el navegador).

> **`[FIX v2]` Condicionado a consentimiento.** Ni el Pixel ni CAPI pueden ejecutarse antes de consentimiento explícito de usuarios del EEE. Ver §11.

### 10.2. Secuencias de email (Resend)

| Secuencia | Disparador | Contenido |
| :-- | :-- | :-- |
| Bienvenida | Registro | Guía del terminal + acceso al canal Free |
| Curiosity trigger | +24 h | Operación con score 90+ y ticker oculto, con el retorno posterior |
| Carrito abandonado | +2 h sin completar checkout | Recordatorio con enlace directo |
| Digest semanal | Domingos 20:00 | Tres mayores compras de CEO de la semana |

> Solo a perfiles con `marketing_consent = TRUE`. Enlace de baja obligatorio en todos los envíos.

---

## 11. `[FIX v2]` CUMPLIMIENTO LEGAL — BLOQUEANTE DE LANZAMIENTO

> La v1.0 no contemplaba ningún aspecto legal. **Ninguno de estos puntos es opcional antes de cobrar el primer euro a un usuario del EEE.**

### 11.1. Naturaleza del servicio

InsiderPulse es un **servicio de tratamiento y presentación de datos públicos**, no un servicio de asesoramiento en materia de inversión.

* El Reglamento (UE) 596/2014 (**MAR**) art. 20 y el Reglamento Delegado 2016/958 regulan las *recomendaciones de inversión*. Una etiqueta como `ALTA CONVICCIÓN` puede interpretarse como recomendación implícita.
* **MiFID II / CNMV:** el asesoramiento en inversión remunerado es actividad reservada a entidades autorizadas.

**Medidas obligatorias:**

1. Disclaimer permanente y visible en landing, aplicación, canales de Telegram y pie de todos los emails:
   > *InsiderPulse presenta información pública procedente de la SEC con fines exclusivamente informativos y educativos. No constituye asesoramiento financiero ni recomendación de compra o venta. Las rentabilidades pasadas no garantizan resultados futuros.*
2. Evitar en todo el material comercial: *"señal de compra"*, *"garantizado"*, *"rentabilidad asegurada"*.
3. **Revisión por abogado especializado en mercados de valores antes del lanzamiento comercial.**

### 11.2. RGPD

* **CMP de consentimiento previo.** El Pixel de Meta **no puede dispararse** antes del consentimiento (la v1.0 lanzaba `PageView` en la landing sin control → incumplimiento directo).
* Rutas obligatorias: `/privacidad`, `/terminos`, `/cookies`.
* Base jurídica documentada para el envío de datos a Meta (EE. UU.) — Data Privacy Framework.
* Derechos ARCO-POL: mecanismo de exportación y supresión de cuenta.
* Consentimiento granular y revocable en `profiles.tracking_consent` / `marketing_consent`.

### 11.3. Fiscalidad

* Servicios digitales B2C en la UE → **ventanilla única OSS**; el IVA se aplica según el país del consumidor.
* Indicar con claridad si 9 € y 39 € son **con o sin IVA**. Stripe Tax automatiza el cálculo.
* Facturación y conservación conforme a normativa española.

---

## 12. BRANDING Y FRONTEND

### 12.1. Identidad

* **Arquetipo:** El Sabio / El Gobernante. Terminal institucional, sobrio, sin promesas.
* **Paleta:** fondo `zinc-950` `#09090b` · tarjetas `zinc-900` `#18181b` · bordes `zinc-800` `#27272a` · acento `emerald-500` `#10b981` · aviso `amber-500` `#f59e0b`.

### 12.2. Landing (`/`)

1. **Barra live:** número de compras de CEO detectadas en 24 h, con pulso.
2. **Hero:** *"Deja de perseguir opiniones de analistas. Sigue el dinero real de los directivos."*
3. **Franja `trades_live`:** rol e importe visibles, ticker `$***` — el motor de conversión (§7.1).
4. **Mockup de alerta:** con datos **ficticios** (§9.1).
5. **Tabla `trades_feed`:** histórico completo y legible; desenfoque a partir de la fila 6 para usuarios Free.
6. **Precios:** tres niveles con toggle anual (−35 %).
7. **CTA móvil fija.**
8. **Pie:** disclaimer §11.1 y enlaces legales.

### 12.3. SEO programático

* Ruta `frontend/src/app/insider-trading/[ticker]/page.tsx` con ISR `revalidate = 3600`.
* OpenGraph dinámico por ticker.
* JSON-LD: `Dataset`, `FAQPage`.
* `sitemap.ts` con todos los tickers registrados.

> Canal de adquisición principal: coste marginal cero y cola larga de miles de tickers.

---

## 13. ESTRUCTURA DEL REPOSITORIO

```text
insiderpulse/
├── .gitignore                        # [FIX v2] PRIMER artefacto — repo público
├── .github/
│   ├── copilot-instructions.md
│   └── workflows/
│       ├── sec_cron.yml              # ingesta periódica
│       ├── keepalive.yml             # [FIX v2] evita autodesactivación a 60 días
│       └── ci.yml                    # tests + verificación RLS (§5.2)
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── routes_auth.py
│   │   │   ├── routes_trades.py      # [FIX v2] tier desde JWT
│   │   │   ├── routes_billing.py
│   │   │   └── routes_telegram.py
│   │   ├── core/
│   │   │   ├── config.py             # validación Pydantic del entorno
│   │   │   ├── security.py           # [FIX v2] get_current_profile / effective_tier
│   │   │   └── supabase.py
│   │   ├── models/{trade,user}.py
│   │   ├── services/
│   │   │   ├── sec_parser.py
│   │   │   ├── title_normalizer.py   # [FIX v2] normalización de officerTitle
│   │   │   ├── scoring.py
│   │   │   ├── cluster_recalc.py     # [FIX v2] recálculo ventana 15 días
│   │   │   ├── telegram.py
│   │   │   ├── stripe_svc.py
│   │   │   ├── meta_capi.py
│   │   │   └── email_service.py
│   │   ├── mcp_server.py
│   │   └── ingestor.py
│   ├── backtest/                     # [FIX v2] Fase 0
│   │   ├── download_history.py
│   │   ├── evaluate_weights.py
│   │   └── README.md
│   ├── tests/
│   │   ├── test_scoring.py
│   │   ├── test_sec_parser.py
│   │   ├── test_title_normalizer.py
│   │   ├── test_cluster_recalc.py
│   │   └── test_rls_isolation.py     # [FIX v2] anon NO puede leer la tabla
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   └── src/
│       ├── app/
│       │   ├── api/{checkout,webhooks/stripe}/route.ts
│       │   ├── insider-trading/[ticker]/{page.tsx,opengraph-image.tsx}
│       │   ├── (legal)/{privacidad,terminos,cookies}/page.tsx   # [FIX v2]
│       │   ├── {sitemap.ts,robots.ts,layout.tsx,page.tsx}
│       ├── components/
│       │   ├── {HeroSection,LiveTeaserBar,TradesTable}.tsx
│       │   ├── {BlurPaywall,PricingCards,StickyMobileCTA}.tsx
│       │   ├── ConsentBanner.tsx     # [FIX v2] CMP previo al Pixel
│       │   └── MetaPixel.tsx
│       └── lib/{supabaseClient,stripe,metaPixel}.ts
├── supabase/migrations/001_initial_schema.sql
├── .env.example
├── SPEC.md
└── README.md
```

---

## 14. PLAN DE IMPLEMENTACIÓN

### `[FIX v2]` Fase 0 — Blindaje y validación *(nueva, previa a todo lo demás)*

> `@workspace Lee SPEC.md. Ejecuta la Fase 0: (1) crea supabase/migrations/001_initial_schema.sql exactamente como aparece en §5, incluidas la revocación de acceso directo, la función current_tier() y las vistas trades_feed y trades_live; (2) crea backend/tests/test_rls_isolation.py que verifique que el rol anon NO puede leer public.insider_trades y que trades_live devuelve ticker '***' para usuarios free; (3) crea backend/backtest/ con el script de descarga de históricos de EDGAR y el evaluador de pesos.`

**Criterio de salida:** los tests de aislamiento RLS pasan **y** existe un informe de backtest con el retorno anormal a 30/90/180 días por tramo de score.

> Si el backtest no respalda los pesos, **ajustarlos antes de construir el producto**. Es más barato corregir la hipótesis ahora que reeducar a clientes de pago después.

### Fase 1 — Motor de scoring

> `@workspace Ejecuta la Fase 1 de SPEC.md: crea backend/app/services/title_normalizer.py con la normalización de officerTitle descrita en §4.3, backend/app/services/scoring.py con el algoritmo determinista de §4 (excluyendo operaciones 10b5-1) y backend/app/services/cluster_recalc.py con el recálculo de la ventana móvil de §4.4. Añade tests unitarios exhaustivos para cada subpuntuación y para los casos límite: prevShares = 0, prevShares negativo, roles múltiples y títulos no mapeados.`

### Fase 2 — Ingesta SEC

> `@workspace Ejecuta la Fase 2 de SPEC.md: implementa backend/app/services/sec_parser.py y backend/app/ingestor.py con el User-Agent reglamentario, límite de 8 req/s con retroceso exponencial, parseo con defusedxml, iteración sobre las N transacciones de cada filing asignando transaction_seq, manejo de enmiendas Form 4/A, resolución CIK→ticker con company_tickers.json y upsert idempotente ON CONFLICT.`

### Fase 3 — API y control de acceso

> `@workspace Ejecuta la Fase 3 de SPEC.md: implementa backend/app/core/security.py con get_current_profile y effective_tier según §6.1, y backend/app/api/routes_trades.py consumiendo las vistas trades_feed y trades_live. El tier NO puede provenir en ningún caso de la petición del cliente.`

### Fase 4 — Telegram, MCP y CAPI

> `@workspace Ejecuta la Fase 4 de SPEC.md: bot de Telegram con canales Free y VIP, enlaces de invitación individuales con member_limit=1 y revocación al cancelar la suscripción; servidor FastMCP; y despachador Meta CAPI condicionado a tracking_consent.`

### Fase 5 — Stripe

> `@workspace Ejecuta la Fase 5 de SPEC.md: stripe_svc.py y las rutas de checkout y webhook, con verificación obligatoria de firma, idempotencia por event.id y sincronización de plan_tier y subscription_status en profiles. Al cancelar, revocar el acceso al canal VIP de Telegram.`

### Fase 6 — Landing y conversión

> `@workspace Ejecuta la Fase 6 de SPEC.md: landing en Next.js con LiveTeaserBar consumiendo trades_live, TradesTable consumiendo trades_feed, BlurPaywall a partir de la fila 6, PricingCards, StickyMobileCTA y ConsentBanner que bloquee el Pixel hasta obtener consentimiento. Todos los mockups deben usar datos ficticios.`

### Fase 7 — SEO, legal y lanzamiento

> `@workspace Ejecuta la Fase 7 de SPEC.md: páginas pSEO por ticker con ISR, opengraph-image, sitemap, robots, las páginas legales de §11 con el disclaimer obligatorio, y los workflows sec_cron.yml, keepalive.yml y ci.yml.`

---

## 15. CHECKLIST PREVIO AL LANZAMIENTO

**Seguridad**

- [ ] `anon` no puede ejecutar `SELECT` sobre `public.insider_trades` (§5.2)
- [ ] `trades_live` devuelve `'***'` para usuarios sin suscripción activa
- [ ] Ningún endpoint acepta el tier como parámetro de entrada
- [ ] `SUPABASE_SERVICE_ROLE_KEY` ausente del bundle del frontend
- [ ] `.env` ignorado por git y verificado con `git check-ignore -v .env`
- [ ] Firma de webhooks de Stripe verificada
- [ ] Secret scanning activado en el repositorio

**Datos**

- [ ] Un Form 4 con varias transacciones genera varias filas
- [ ] Las enmiendas Form 4/A marcan `is_superseded` en las filas corregidas
- [ ] Reejecutar el cron no duplica filas ni alertas
- [ ] Las operaciones 10b5-1 se persisten con `score IS NULL`
- [ ] El recálculo de cluster actualiza los scores de las operaciones previas

**Legal**

- [ ] Disclaimer visible en landing, app, Telegram y emails
- [ ] `/privacidad`, `/terminos` y `/cookies` publicadas
- [ ] El Pixel no se dispara antes del consentimiento
- [ ] IVA configurado en Stripe Tax
- [ ] Revisión por abogado completada
- [ ] Ningún material comercial usa personas o empresas reales en ejemplos

**Producto**

- [ ] Informe de backtest completado y pesos validados o ajustados
- [ ] El copy dice "menos de 15 minutos", no "tiempo real instantáneo"
- [ ] Un usuario Free ve datos útiles y legibles, no una pantalla de asteriscos
