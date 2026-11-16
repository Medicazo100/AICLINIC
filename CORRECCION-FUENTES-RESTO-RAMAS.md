# CORRECCIÓN DE FUENTES — RESTO DE LAS RAMAS + CITAS EN EL TEXTO (AICLINIC)

## PARTE A — CANCELADA por Gabriel (16 nov ~09:00)
Los pendientes de neumonía (Cochrane pub5 404, fuente IMSS genérica, NG138 reemplazada) se DEJAN COMO ESTÁN: 5 de 7 fuentes abren documento real y Gabriel dijo que así está bien («no pasa nada»). NO tocar la rama de neumonía.

## PARTE C — Reintegrar las citas dentro del texto del diagnóstico final (NUEVO, pedido por Gabriel 16 nov ~09:00)
Antes, el texto del diagnóstico final (fisiopatología, correlación de hallazgos, plan de manejo) traía las citas entre corchetes [1], [2]... dentro del propio texto, y aparte venía la lista completa de fuentes hasta abajo. Ahora solo aparece la lista de abajo. Hay que reintegrar las citas en el texto.
**Dato verificado por Frida en el código actual:** el mecanismo YA EXISTE en ambos lados y hay que hacerlo cumplir, no construirlo de cero:
- El prompt de `getFinalDiagnosis` en `services/geminiService.ts` (líneas ~1088, ~1098, ~1101) YA ordena insertar llamadas numéricas entre corchetes [1]–[6] en el texto para respaldar fisiopatología, criterios, fármacos con dosis y evidencia.
- El render en `components/MedicalSimulator.tsx` (`renderDiagnosisHtml`, línea ~981) YA convierte `[1]`, `[1, 2, 4]` en pastillas clicables que saltan a `#fuente-N` de la lista de abajo.
**Tarea:** investiga por qué los casos nuevos en producción NO muestran esas citas en el texto (¿el modelo no pone los [n]? ¿los números no corresponden con el orden final de la lista tras integrar catálogo + fuentes de la IA?). Corrige la causa raíz: el prompt, el orden de integración de fuentes, o un posprocesado que garantice que el texto del diagnóstico traiga citas [n] que correspondan EXACTAMENTE con las fuentes numeradas de la lista de abajo. Regla: cada afirmación de fisiopatología, criterio diagnóstico y esquema farmacológico debe llevar su [n]; el número debe saltar a la fuente correcta. No inventar citas: solo números que existan en la lista. Verificación en navegador integrado: caso nuevo de neumonía y caso nuevo de epilepsia; el texto del diagnóstico final debe mostrar pastillas [n] clicables que lleven a la fuente correcta de abajo, y la lista completa debe seguir abajo como ahora.

## PARTE B — Resto de las ramas del catálogo

- **Rama 1. Epilepsia** — verificar que sigue intacta (regresión, sin cambios si está bien).
- **Rama 2. EVC / Ictus** — HOY usa la URL genérica de DGMOSS para la GPC CENETEC de EVC.
- **Rama 3. Cardiología (infarto, insuficiencia cardíaca, arritmias)** — HOY usa DGMOSS genérico para la GPC CENETEC de IAM. **Incluir HIPERTENSIÓN ARTERIAL: verificar si la rama la cubre; si no, crear rama propia de hipertensión con sus fuentes (p. ej. guías AHA/ACC, ESC/ESH, NICE, GPC).**
- **Rama 4. Diabetes / cetoacidosis / estado hiperosmolar** — HOY usa DGMOSS genérico para la GPC CENETEC de cetoacidosis.
- **Rama 8. Otras afecciones respiratorias** — HOY usa portadas genéricas (atsjournals.org, chestnet.org). Reemplazar por documentos específicos del tema que corresponda (derrame pleural, insuficiencia respiratoria, etc.).
- **Rama 9. Sepsis / choque séptico / meningitis** — HOY usa DGMOSS genérico para la GPC CENETEC de choque séptico.
- **Rama 10. Abdomen agudo / apendicitis / colecistitis / pancreatitis** — HOY usa DGMOSS genérico para la GPC CENETEC de apendicitis.
- **Rama 11. Nefrología / lesión renal aguda / hiperpotasemia / ERC** — HOY usa DGMOSS genérico para la GPC CENETEC de lesión renal aguda.
- **Rama 12. Respaldo general** — HOY usa el catálogo maestro CENETEC (`cenetec.salud.gob.mx/contenidos/gpc/catalogoMaestroGPC.html`). **Ese dominio YA NO RESUELVE DNS: eliminarlo.** El respaldo general debe ser solo documentos reales y verificados.

## Hechos verificados que debes tomar en cuenta (no los re-verifiques por curl, ya están confirmados)
- `https://www.gob.mx/salud/dgmoss` es una página genérica de la DGMOSS, NO un documento de guía. No sirve como fuente de ningún tema.
- El dominio `cenetec.salud.gob.mx` NO resuelve DNS. Ninguna URL de CENETEC con ese dominio funciona.
- La copia de una GPC en SlideShare u otros sitios no oficiales NO es fuente oficial: no la uses.

## Reglas (las mismas de la corrección de neumonía)
1. Cada fuente debe enlazar DIRECTO a un documento específico del tema de su rama: guía clínica, artículo, capítulo o revisión con título real. **Prohibido:** portadas, listados, catálogos, páginas de inicio de sociedades, URLs de búsqueda y URLs genéricas de portales.
2. Cada URL nueva debe verificarse EN EL NAVEGADOR INTEGRADO (no solo por curl): debe abrir el documento real, con su título visible. Anota el código HTTP y el título observado de cada una, como se hizo con neumonía.
3. Si una URL devuelve 403 anti-bot / Cloudflare en el chequeo automático, indícalo explícitamente en el reporte y verifica si un usuario real la abre en el navegador integrado.
4. Mantener 5–6 fuentes por rama, todas de la especialidad correcta. No cruzar temas entre ramas.
5. No tocar los casos viejos guardados en la Biblioteca. La corrección actúa al GENERAR casos nuevos.
6. No tocar la rama de neumonía, EPOC ni asma (ya están bien), salvo regresión.

## Verificación obligatoria (regla de prueba en navegador)
1. **Triple verificación:** `npm run lint` (0 errores), `tsc --noEmit` (0 errores), `npm run build` exitoso, y auditoría de gobernanza de 6 roles aprobada.
2. **Prueba en el navegador integrado (localhost):** genera UN caso nuevo por cada rama corregida y verifica que sus fuentes sean todas específicas del tema. **Mínimo obligatorio:** un caso de sepsis, uno de hipertensión y uno de EVC, con captura de sus listas de fuentes.
3. **Regresión:** un caso nuevo de neumonía (sus 6 fuentes deben seguir intactas) y uno de epilepsia (sus 7 fuentes intactas).
4. **Commit LOCAL sin push** con mensaje `fix(simulator): corregir fuentes de las demas ramas del catalogo` y reporte completo: qué cambió por rama, tabla de URLs con código HTTP y título, resultados de las pruebas en navegador y de la regresión.
5. NO hacer push. El push se hace después, con token temporal, como siempre.
