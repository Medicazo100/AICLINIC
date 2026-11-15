# CORRECCIÓN — Fuentes del Simulador no llevan a ningún documento

## Contexto
Bug diagnosticado el 15 nov 2026 por Gabriel en AICLINIC (asistente-medico-20.vercel.app), caso de epilepsia «Un apagón repentino en medio de la reunión», sección «Fuentes y Evidencia Científica» al final del caso.

## El problema (verificado en el navegador de Gabriel)
Las 6 fuentes NO enlazan a ningún documento específico. Cada botón «Consultar fuente» construye una URL de BÚSQUEDA usando el diagnóstico completo como consulta:

- **[1] CENETEC** → `google.com/search?q=site:gob.mx/cenetec` + todo el diagnóstico + «guia practica clinica» → **0 resultados** («No se han encontrado resultados para tu búsqueda»).
- **[2] AHA / ACC** → `ahajournals.org/action/doSearch?AllField=` + todo el diagnóstico → **0 resultados** («Your search did not return any results»).

Las 6 fuentes son siempre el mismo patrón: un nombre genérico + el diagnóstico pegado:
1. Guía de Práctica Clínica CENETEC (Sector Salud México)
2. AHA / ACC Cardiovascular Guidelines & Statements (¡cardiología, especialidad equivocada para un caso de epilepsia!)
3. PubMed / National Library of Medicine
4. SciELO / IMSS
5. UpToDate
6. Cochrane Library

Causa: la app genera las fuentes como texto (prompt en `services/geminiService.ts`, sección «Integrar fuentes») y el render (`components/MedicalSimulator.tsx`, que parsea «### Fuentes» del texto del diagnóstico) las convierte en búsquedas. Nunca hay una URL real verificada de un documento.

## La corrección que se pide
1. **Localiza dónde se construye la URL de cada fuente** (prompt y render) y corrígelo ahí. No toques el contenido del caso, el diagnóstico ni ningún otro módulo.
2. **Cada fuente debe enlazar a la URL REAL y VERIFICADA del documento específico** que respalda el caso (guía o artículo concreto). Para epilepsia, por ejemplo: la GPC de CENETEC sobre epilepsia (su PDF real), guías ILAE, NICE NG137, guías AES. El modelo debe devolver la URL real de cada documento y la app debe enlazar DIRECTO a ella.
3. **Prohibido construir URLs de búsqueda**: nada de `google.com/search`, nada de `doSearch`, nada de búsquedas internas de un sitio. Si una fuente no tiene URL real verificada, se ELIMINA esa fuente; nunca se inventa una URL.
4. **Las fuentes deben ser de la especialidad del caso.** AHA/ACC (cardiología) NUNCA debe aparecer en un caso de neurología/epilepsia. El prompt debe restringir las fuentes a la especialidad del diagnóstico (neurología/epileptología para epilepsia, cardiología solo para casos cardiológicos, etc.).
5. **Respaldo (solo si no se puede obtener la URL real):** construir la búsqueda con palabras CLAVE CORTAS (2–5 palabras, p. ej. «epilepsia focal crisis tónico-clónica guía»), sin restricción `site:` y sin pegar todo el diagnóstico. Esto es solo respaldo, no el objetivo.
6. Aplica la misma corrección a las citas numéricas `[n]` del texto si enlazan a algún lado.

## Verificación obligatoria (antes de darlo por terminado)
- Corre el caso de epilepsia «Un apagón repentino en medio de la reunión» y haz clic en cada «Consultar fuente»: **cada enlace debe abrir una página (HTTP 200) cuyo contenido sea el tema específico del caso**, no una búsqueda ni una página genérica. Reporta la URL final de cada una de las 6 fuentes.
- `lint` / `tsc` 0 errores, build OK.
- Commit LOCAL con autor Medicazo100 <medicazo100@gmail.com>. **SIN push** (el push lo hace Frida después, con token temporal).
- Reporta: archivos cambiados, las 6 URLs finales verificadas y el resultado de las pruebas de clic.
