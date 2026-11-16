# CORRECCIÓN DE FUENTES — Rama de neumonía (AICLINIC)

## Contexto
Prueba en producción 2026-11-16 (caso nuevo de neumonía, «Sombras en el pecho de un veterano otoñal», NAC grave):
de 7 fuentes, solo 2 llevan a un documento real y específico de neumonía. La corrección de Fuentes de ayer (521aa73 + 5489add)
funcionó para epilepsia, pero la rama de neumología NO.

## Causa raíz (verificada por Frida en el código, `services/geminiService.ts`, `buildPrecisionMedicalSources()`)
La rama 5 «NEUMOLOGÍA / NEUMONÍA / EPOC / ASMA» mete en UN solo grupo neumonía, EPOC y asma, con un solo regex
(`/neumon[ií]a|epoc|asma|pulmonar|bronquial|insuficiencia respiratoria|derrame pleural/i`).
Por eso un caso de NEUMONÍA recibe fuentes de EPOC (GOLD) y de asma (GINA), y URLs genéricas.

## Lo que hay que hacer
1. **Dividir la rama 5 en tres ramas separadas**, cada una con sus propias fuentes y su propio regex:
   - Rama NEUMONÍA: solo regex de neumonía (`neumon[ií]a`, NAC, consolidación, etc.).
   - Rama EPOC: solo regex de EPOC.
   - Rama ASMA: solo regex de asma.
   - Las fuentes de EPOC y asma NO deben aparecer nunca en un caso de neumonía, y viceversa.
2. **Reescribir la rama NEUMONÍA con estas URLs, TODAS verificadas por Frida (HTTP 200 o página real confirmada):**
   - ATS/IDSA 2019, Diagnosis and Treatment of Adults with Community-acquired Pneumonia — `https://pubmed.ncbi.nlm.nih.gov/31573350/` (SE QUEDA, ya estaba bien).
   - NICE NG250, Pneumonia (community-acquired): antimicrobial prescribing (guía vigente; reemplaza a NG138) — `https://www.nice.org.uk/guidance/ng250`
   - StatPearls, Community-Acquired Pneumonia (NCBI Bookshelf) — `https://www.ncbi.nlm.nih.gov/books/NBK430749/`
   - Guía de Práctica Clínica para el diagnóstico y tratamiento de la neumonía adquirida en la comunidad, Infectio 2026;30(2):185-223 (Varón-Vega et al.) — `https://www.revistainfectio.org/P_OJS/index.php/infectio/article/download/1293/1454/4411`
   - Cochrane, Antibiotics for community-acquired pneumonia in adult outpatients (CD002109) — `https://www.cochranelibrary.com/cdsr/doi/10.1002/14651858.CD002109.pub5/full` (curl devuelve 412 por protección anti-bot; VERIFICAR en tu navegador integrado que abre la revisión completa).
   - NICE NG138 — `https://www.nice.org.uk/guidance/ng138` (solo como sexta fuente si hace falta; la propia página indica que fue reemplazada por NG250).
3. **ELIMINAR de la rama de neumonía (y no usarlas para neumonía):**
   - `https://www.gob.mx/salud/dgmoss` — página genérica de DGMOSS, no es la guía; además devuelve Access Denied de Akamai.
   - `http://www.cenetec.salud.gob.mx/contenidos/gpc/catalogoMaestroGPC.html` — catálogo general, y el dominio ya NO resuelve DNS. La GPC CENETEC IMSS-234-09 (2009) no tiene URL directa funcional: NO incluir CENETEC para neumonía hasta tener una URL de documento verificada.
   - `https://goldcopd.org/2024-gold-report/` — es de EPOC, no de neumonía (pasa a la rama EPOC).
   - `https://ginasthma.org/reports/` — es listado de informes de asma (pasa a la rama ASMA, y ahí debe apuntar a un informe específico, no al listado).
   - `https://www.cochranelibrary.com` — portada genérica; toda fuente Cochrane debe apuntar al DOI de la revisión específica.
4. **Regla general (vale para TODAS las ramas del catálogo, no solo neumonía):** ninguna URL de portada, listado, catálogo o buscador. Cada fuente debe enlazar al documento específico del tema del caso. `isSearchUrl()` y el filtro del render deben seguir rechazando búsquedas.
5. Verificación triple habitual (lint/tsc 0 errores, build, audit de 6 roles) y commit local SIN push.

## Prueba obligatoria en tu navegador integrado (REGLA NUEVA)
Antes de reportar, prueba en tu navegador integrado (`/browser`):
- Genera un caso NUEVO de neumonía en https://asistente-medico-20.vercel.app/ (Simulador) y abre CADA «Consultar fuente»: todas deben llevar a un documento específico de neumonía.
- Repite la prueba de regresión con un caso nuevo de epilepsia (la corrección de ayer debe seguir intacta: 9 fuentes reales).
- Reporta la lista de fuentes con su URL y lo que muestra cada página al abrirla. Si alguna URL falla o no es del tema, corrígela antes de reportar.
- NO toques los casos viejos guardados en la Biblioteca (decisión de Gabriel: se dejan así).
- El push a GitHub lo hace Gabriel contigo después, con token temporal, como siempre. NO hagas push en este paso.
