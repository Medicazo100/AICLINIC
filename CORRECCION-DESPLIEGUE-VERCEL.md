# CORRECCIÓN — El despliegue de Vercel falla en `npm install` (EBADPLATFORM)

## Contexto
El commit 521aa73 («fix(simulator): vincular fuentes clínicas a documentos
reales verificados y eliminar URLs de búsqueda») ya está en GitHub main
(Medicazo100/AICLINIC), pero su despliegue en Vercel (proyecto `aiclinic`,
dominio asistente-medico-20.vercel.app) FALLÓ el 15 nov 2026 a las 17:53:34
(GMT-6), estado Error, duración 3 segundos. La producción sigue sirviendo el
despliegue anterior (11 nov). El código de la corrección de Fuentes nunca
llegó a producción.

## El error exacto (registro de construcción de Vercel)
```
17:53:33.848  npm error code EBADPLATFORM
17:53:33.848  npm error notsup Unsupported platform for @rollup/rollup-darwin-arm64@4.68.3: wanted {"os":"darwin","cpu":"arm64"} (current: {"os":"linux","cpu":"x64"})
17:53:33.848  npm error notsup Valid os:   darwin
17:53:33.848  npm error notsup Actual os:   linux
17:53:33.848  npm error notsup Valid cpu:   arm64
17:53:33.848  npm error notsup Actual cpu:   x64
17:53:33.876  Error: [BUILD_UTILS_SPAWN_1] Command "npm install" exited with 1
```

## Diagnóstico
`npm install` en la máquina de construcción de Vercel (Linux x64) intenta
instalar el binario opcional de Rollup para macOS ARM64
(`@rollup/rollup-darwin-arm64@4.68.3`) y falla porque esa plataforma no
corresponde. El binario quedó amarrado en las dependencias / en el archivo de
bloqueo (`package-lock.json`) como si fuera obligatorio, en lugar de quedar
como opcional y omitirse en Linux. El despliegue muere antes de compilar; el
código de la app no es el problema.

## La corrección que se pide
1. Revisa `package.json` y `package-lock.json` y localiza por qué el paquete
   `@rollup/rollup-darwin-arm64` (y cualquier otro binario opcional de
   Rollup para otra plataforma) queda como obligatorio en la instalación.
2. Corrige el bloqueo de dependencias para que la instalación funcione en
   Linux x64 (la máquina de Vercel): el binario de macOS debe quedar como
   opcional y omitirse en Linux, o debe instalarse el binario de Linux
   correspondiente. Usa la solución estándar para este error; NO cambies el
   comando de construcción ni la configuración de Vercel, y NO toques el
   código de la app ni la corrección de Fuentes (commit 521aa73).
3. No cambies versiones de dependencias de la app ni agregues dependencias
   nuevas salvo que sea indispensable para la corrección.

## Verificación obligatoria (antes de darlo por terminado)
- `npm run lint` con 0 errores, `tsc --noEmit` con 0 errores, build de
  producción (`npm run build`) OK en esta máquina.
- Comprueba en el `package-lock.json` resultante que la entrada del binario
  de Rollup para macOS ya no obliga su instalación en Linux (muestra la
  entrada corregida en tu reporte).
- Commit LOCAL con autor Medicazo100 <medicazo100@gmail.com>. **SIN push**
  (el push lo coordina Frida después, con token temporal).
- Reporta: archivos cambiados, qué estaba mal, cómo se corrigió y el
  resultado de la verificación. Si el despliegue se puede relanzar desde
  Vercel, NO lo relances: Frida lo verifica y lo coordina después.
