# Práctica 4 — Visión por Computador

Autores
- Alberto José Rodríguez Ruano
- Miguel Ángel Rodríguez Ruano

Descripción breve

Esta carpeta contiene los notebooks y materiales de la Práctica 4, cuyo objetivo principal es construir un prototipo para la detección y seguimiento de personas y vehículos en vídeo, con localización de matrículas.

Estructura relevante

- `P4/` — Notebook principal: `VC_P4.ipynb` y recursos ligados al pipeline de detección, tracking y anonimización (blur). Contiene también ejemplos de uso y código para ejecutar el procesamiento de vídeo.
- `P4b/` — Notebook complementario: `VC_P4b.ipynb` (ejercicio comparativo de OCR). Aquí se comparan EasyOCR, Tesseract y PaddleOCR sobre un dataset de matrículas, y se generan resultados/CSV con métricas.

Leer antes de ejecutar

1. Abrir `P4/README.md` para ver la explicación completa del pipeline principal, requisitos y cómo ejecutar el notebook `VC_P4.ipynb`.
2. Abrir `P4b/README.md` para ver el ejercicio de OCR (cómo generar/extraer el dataset, ejecutar la comparación y dónde encontrar los resultados).

Enlaces rápidos

- Notebook principal (pipeline vídeo / blur / tracking): `P4/README.md`
- Ejercicio OCR comparativo: `P4b/README.md`

Notas

- Asegúrate de tener instalados los pesos de los modelos (`yolo11n.pt`, `best.pt`) y las dependencias necesarias (ver cada README para instrucciones de instalación y recomendaciones de entorno).
- Los notebooks asumen que se ejecutan desde su carpeta correspondiente; si cambias el working directory adapta las rutas de los archivos en las primeras celdas.


