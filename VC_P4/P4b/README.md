# Práctica 4b — Comparativa OCR sobre matrículas

Este README resume el notebook `VC_P4b.ipynb`, que contiene un ejercicio práctico para comparar el rendimiento de distintos motores OCR (EasyOCR y Tesseract, con hooks para PaddleOCR) sobre un dataset de recortes de matrículas. El notebook realiza:

- extracción de recortes (a partir de etiquetas YOLO),
- preprocesado y lectura con EasyOCR y Tesseract,
- normalización y métricas (Levenshtein, accuracy carácter),
- guardado de resultados en CSV y ejemplos visuales.

Repositorio / Estructura

- `VC_P4/P4b/VC_P4b.ipynb` — notebook principal con bloques numerados (config, carga de modelos, funciones OCR, prueba por imagen, procesamiento completo, estadísticas, y pipeline de vídeo con OCR).
- `VC_P4/P4b/ejemplos/` — imágenes de ejemplo generadas por el notebook (recortes y comparativas). Algunas muestras: `0116GPD.png`, `0290KWT.png`, `0303BML.png`.
- `VC_P4/P4b/ocr_comparison_results/` — carpeta de salida generada por el notebook al procesar el dataset: `comparativa_ocr.csv`, `ejemplos/`, `comparativa_grafica.png`..
- `VC_P4/P4b/outputs/` — carpeta de salida generada por el notebook al procesar el video: `detecciones_ocr.csv` y `salida_con_ocr.mp4`.

## Requisitos mínimos

El notebook utiliza las siguientes librerías (instalación orientativa):

```bash
conda create -n VC_P4b python=3.9 -y
conda activate VC_P4b
pip install opencv-python numpy matplotlib pandas easyocr pytesseract torch
```

Además es necesario tener instalado Tesseract OCR y configurar la ruta `TESSERACT_PATH` en el notebook (por defecto `C:\Program Files\Tesseract-OCR\tesseract.exe` en Windows).

## Cómo usar el notebook (pasos rápidos)

1. Abrir `VC_P4b.ipynb` en Jupyter/VSCode.
2. Editar la celda de configuración (BLOCK 1) para apuntar a `DATASET_DIR` y `TESSERACT_PATH` si es necesario.
3. Ejecutar las celdas en orden:
   - BLOQUE 1: configuración y utilidades
   - BLOQUE 2: carga de modelos (EasyOCR)
   - BLOQUE 3: funciones OCR (normalización y lectura)
   - BLOQUE 4: prueba con una imagen
   - BLOQUE 5: procesamiento completo del dataset
   - BLOQUE 6: estadísticas y gráficas

Al final se generará `ocr_comparison_results/comparativa_ocr.csv` y ejemplos en `ocr_comparison_results/ejemplos/`.

4. Ejecutar la última celda para obtener las matrículas en el vídeo

Al final se generará `VC_P4/P4b/outputs/`

## Fragmentos útiles

1) Configuración (extraída del BLOQUE 1):

```python
DATASET_DIR = "../Matriculas"
LABELS_DIR = os.path.join(DATASET_DIR, "labels")
IMAGES_DIR = DATASET_DIR
TESSERACT_PATH = r"C:\Program Files\Tesseract-OCR\tesseract.exe"
OUTPUT_DIR = "ocr_comparison_results"
RESULTS_CSV = os.path.join(OUTPUT_DIR, "comparativa_ocr.csv")
SAVE_EXAMPLES = True
```

2) Lectura y normalización (funciones centrales):

```python
def clean_text(text):
    return re.sub(r"[^A-Z0-9]", "", text.upper())

def fix_common_errors(text):
    text = text.replace('O', '0').replace('I', '1').replace('Q', '0')
    return text

def normalize_plate(text, ground_truth=""):
    # Aplica limpieza y reglas heurísticas para intentar formar
    # una matrícula española estándar (4 números + 3 letras) u otras variantes.
    text = clean_text(text)
    text = fix_common_errors(text)
    # ... lógica adicional en el notebook para elegir el mejor patrón
    return text
```

3) Procesamiento por imagen (recorte y ejecución de OCR):

```python
def process_image_ocr(img_path, label_path, ground_truth, save_example=False):
    img = cv2.imread(img_path)
    yolo_coords = read_yolo_label(label_path)
    x1, y1, x2, y2 = yolo_to_bbox(yolo_coords, img.shape)

    # Ajuste heurístico para placas modernas
    x1_new = x1 + int((x2 - x1) * 0.10)
    crop = img[y1:y2, x1_new:x2].copy()
    crop_resized = cv2.resize(crop, None, fx=3, fy=3, interpolation=cv2.INTER_CUBIC)

    # EasyOCR
    results = reader.readtext(crop_resized, paragraph=True, detail=0)
    easy_raw = " ".join(results).upper()

    # Tesseract
    gray = cv2.cvtColor(crop_resized, cv2.COLOR_BGR2GRAY)
    _, enhanced = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
    tess_raw = pytesseract.image_to_string(enhanced, config="--oem 1 --psm 7 -l spa").strip().upper()

    # Normalizar y devolver métricas
    easy_norm = normalize_plate(easy_raw, ground_truth)
    tess_norm = normalize_plate(tess_raw, ground_truth)
    return { 'easy_normalized': easy_norm, 'tess_normalized': tess_norm }
```

## Ejemplos visuales de la clasificación

![Ejemplo 0116GPD](images/0116GPD.png)
![Ejemplo 0116GPD](images/0476MNN.png)
![Ejemplo 0116GPD](images/0962LLT.png)
![Ejemplo 0116GPD](images/0416MLX.png)

## Salida esperada y CSV

El CSV `ocr_comparison_results/comparativa_ocr.csv` incluye columnas como:

- `imagen`, `ground_truth`, `easy_raw`, `easy_normalized`, `easy_accuracy`, `easy_levenshtein`, `easy_time`,
- `tess_raw`, `tess_normalized`, `tess_accuracy`, `tess_levenshtein`, `tess_time`

## Notas y recomendaciones

- Ajusta `TESSERACT_PATH` según tu sistema.
- Si obtienes lecturas con caracteres repetidos o ruido, prueba a aumentar el zoom del recorte (fx/fy) o aplicar técnicas de realce (CLAHE, top-hat) antes de OCR.
- Considera ejecutar una heurística que elija entre EasyOCR y Tesseract según Levenshtein contra una lista de formatos válidos.
