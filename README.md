# MetroDecide

Herramienta de aseguramiento y control metrológico para la verificación de
contenido de principio activo en tabletas por espectroscopía NIR, con dos
espectrofotómetros (NIR-1 y NIR-2).

Requisito normativo del proyecto: **declaración de conformidad y reglas de
decisión** — ILAC-G8 e INTE-ISO/IEC 17025:2017, 7.8.6. El modelo se valida
contra ASTM E1655-17 (Reaprobada 2024).

## Perfiles de usuario

El acceso es **simulado**: no hay servidor ni datos personales. Los dos perfiles
están escritos en el propio archivo y existen para demostrar los dos niveles de
permiso.

| Usuario | Clave | Perfil | Puede |
|---|---|---|---|
| `tecnico` | `1234` | Técnico de laboratorio | Medir, elegir equipo y modelo, pegar espectros, guardar en el registro |
| `gerente` | `1234` | Responsable de calidad | Todo lo anterior, más el registro y tablero, la ficha del método, los riesgos de la regla y la definición de la regla de decisión |

La regla de decisión solo la cambia el responsable de calidad, porque
ISO/IEC 17025 7.1.3 pide que se acuerde y documente **antes** de medir. El
técnico mide siempre con la regla vigente y la ve declarada en pantalla.

## Reglas de decisión disponibles

| Regla | Banda de guarda | Límites de aceptación |
|---|---|---|
| Aceptación simple | w = 0 | AL = TL, AU = TU |
| Banda de guarda r = 0,5 | w = 0,5 U | AL = TL + w, AU = TU − w |
| Banda de guarda r = 1 (adoptada) | w = U | AL = TL + w, AU = TU − w |
| Banda de guarda r = 1,5 | w = 1,5 U | AL = TL + w, AU = TU − w |

## Qué hace con cada tableta

La aplicación no recalcula el modelo: todo sale de `datos.json`. Con cada
tableta ejecuta cinco pasos sobre el espectro crudo de 650 valores.

1. **SNV** — resta la media del espectro y divide entre su desviación
   poblacional (N = 650).
2. **Savitzky–Golay y recorte** — ventana 21, polinomio de orden 2, segunda
   derivada; se conservan las salidas hasta 1700 nm (551 valores).
3. **Predicción** — `x = intercepto + SNV(espectro) · pesos − b`. Los pesos ya
   llevan el filtro y el recorte adentro.
4. **Compuerta de dominio** — leverage h (E1655 16.2, Ec. 69) y residuo
   espectral RMSSR (16.4.4, Ec. 75) contra sus límites. Si alguno se excede, el
   resultado es NO EVALUABLE y la tableta va al laboratorio de referencia.
5. **Veredicto** — CONFORME si el resultado cae entre AL y AU; si no, NO
   CONFORME.

Los pasos 1 a 3 corren sin mostrarse: la interfaz presenta el resultado, no el
procedimiento.

## Pantallas

- **Medición** — resultado en mg con su incertidumbre expandida, veredicto, la
  regla de conformidad con la tolerancia y la banda de guarda, y el estado de la
  compuerta de dominio. Permite guardar la medición en el registro, con lote.
- **Registro y tablero** — resumen, veredictos por equipo, veredicto por lote,
  tabla filtrable y exportación a CSV.
- **Ficha del método** — cuestionario de la Sección 25, tabla de trazabilidad
  normativa, SEC, SEV, U y acuerdo entre equipos.
- **Riesgos de la regla** — barrido del ancho de banda de guarda y resultados
  por equipo.

## Registro de mediciones

Las mediciones se guardan en el almacenamiento local del navegador, así que
sobreviven al cierre de la página. Son propias de cada navegador y equipo: no se
comparten entre computadoras.

El botón **Cargar caso práctico** mide las tabletas de demostración en los dos
equipos y las guarda, lo que deja más de ochenta registros listos para revisar.
**Vaciar registro** los borra.

**Exportar CSV** descarga el registro filtrado, separado por punto y coma, listo
para abrir en Excel. Si el visor bloquea la descarga, el mismo botón muestra el
contenido para copiarlo.

## Veredicto por lote

Cada medición se etiqueta con un lote. En el tablero, un lote sale:

- **NO CONFORME** si al menos una tableta es no conforme;
- **PENDIENTE** si quedan tabletas no evaluables por resolver en el laboratorio
  de referencia;
- **CONFORME** en el resto de los casos.

## Archivos

| Archivo | Qué es |
|---|---|
| `index.html` | La interfaz completa: estructura, estilos, núcleo de cálculo y lógica de pantalla. Sin dependencias |
| `datos.json` | Modelos PLS y ELC, matriz del filtro Savitzky–Golay y espectros de demostración |

## Cargar otro modelo

El botón «Cargar otro JSON del modelo» acepta `modelo_PLS_por_instrumento.json`,
`modelo_ELC.json` o `modelos_metrodecide_ambos.json` tal como los escribe el
script de R. No hay que tocar el código: la interfaz lee el JSON y se actualiza
sola.

## Instalación

No requiere instalación ni servidor. Basta con abrir `index.html`, o publicar los
dos archivos en cualquier hospedaje estático. La versión publicada está en
GitHub Pages.

## Alcance

El modelo exportado sirve solo para NIR-1 y NIR-2 con este producto, dentro del
intervalo de calibración de 151,6 a 239,1 mg. Un equipo distinto o un producto
distinto exigen repetir el estudio de calibración.
