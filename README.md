# MetroDecide

Interfaz de verificación de contenido de principio activo en tabletas por
espectroscopía NIR, para dos espectrofotómetros (NIR-1 y NIR-2).

Aplica ASTM E1655-17 (Reaprobada 2024) para la validación del modelo y la regla
de decisión binaria con banda de guarda de ILAC-G8, JCGM 106 e ISO/IEC 17025.

## Qué hace

La aplicación no recalcula el modelo: todo sale del archivo `datos.json`. Con
cada tableta ejecuta los cinco pasos sobre el espectro crudo de 650 valores.

1. **SNV** — resta la media del espectro y divide entre su desviación
   poblacional (N = 650).
2. **Savitzky–Golay y recorte** — ventana 21, polinomio de orden 2, segunda
   derivada; se conservan las salidas hasta 1700 nm (551 valores). Solo hace
   falta para el paso 4.
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
  compuerta de dominio.
- **Ficha del método** — cuestionario de la Sección 25, tabla de trazabilidad
  normativa, SEC, SEV, U y acuerdo entre equipos.
- **Riesgos de la regla** — barrido del ancho de banda de guarda y resultados
  por equipo.

## Archivos

| Archivo | Qué es |
|---|---|
| `index.html` | La interfaz completa: estructura, estilos, núcleo de cálculo y lógica de pantalla. Sin dependencias |
| `datos.json` | Modelos PLS y ELC, matriz del filtro Savitzky–Golay y espectros de demostración |

## Cargar otro modelo

El botón «Cargar otro JSON del modelo» de la barra lateral acepta
`modelo_PLS_por_instrumento.json`, `modelo_ELC.json` o
`modelos_metrodecide_ambos.json` tal como los escribe el script de R. No hay que
tocar el código: la interfaz lee el JSON y se actualiza sola.

## Alcance

El modelo exportado sirve solo para NIR-1 y NIR-2 con este producto, dentro del
intervalo de calibración de 151,6 a 239,1 mg. Un equipo distinto o un producto
distinto exigen repetir el estudio de calibración.
