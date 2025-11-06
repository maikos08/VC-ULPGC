#  Práctica 4 — Visión por Computador

## 👥 Autores
- **Alberto José Rodríguez Ruano**  
- **Miguel Ángel Rodríguez Ruano**

---

## 📄 Descripción breve

Esta práctica tiene como objetivo **desarrollar un prototipo para la detección y seguimiento de personas y vehículos en vídeo**, incluyendo la **localización y lectura de matrículas** mediante OCR.

En esta carpeta encontrarás los **notebooks principales y materiales complementarios** necesarios para el desarrollo de la práctica.

---

## 📂 Estructura del proyecto

### [`P4/`](./P4/)
Notebook principal: [`VC_P4.ipynb`](./P4/VC_P4.ipynb)

Contiene el **pipeline completo de detección, tracking y anonimización (blur)**.  
Incluye ejemplos de uso y código para ejecutar el procesamiento de vídeo paso a paso.

### [`P4b/`](./P4b/)
Notebook complementario: [`VC_P4b.ipynb`](./P4b/VC_P4b.ipynb)

Ejercicio comparativo de **OCR sobre matrículas**, evaluando los modelos:
- **EasyOCR**
- **Tesseract**

El notebook genera métricas y resultados en formato CSV.

---

## ⚙️ Instrucciones previas a la ejecución

1. 📘 Lee primero [`P4/README.md`](./P4/README.md) para conocer:
   - El funcionamiento del pipeline principal.  
   - Requisitos e instalación de dependencias.  
   - Cómo ejecutar el notebook [`VC_P4.ipynb`](./P4/VC_P4.ipynb).

2. 🔍 Revisa después [`P4b/README.md`](./P4b/README.md) para el ejercicio de OCR:
   - Cómo generar y procesar el dataset de matrículas.  
   - Cómo comparar los resultados entre distintos motores OCR.
   - Vídeo de como EasyOCR lee matrículas mientras están siendo trackeadas.
   - Dónde encontrar las métricas y CSV generados.

---

## 🔗 Enlaces rápidos

| Descripción | Enlace |
|--------------|--------|
| 📹 Pipeline principal (vídeo / blur / tracking) | [`P4/README.md`](./P4/README.md) |
| 🔤 Ejercicio comparativo OCR | [`P4b/README.md`](./P4b/README.md) |

---

## 🧩 Notas importantes

- Asegúrate de tener instalados los **pesos de los modelos** necesarios:  
  - `yolo11n.pt`  
  - `best.pt`

- Verifica que las **dependencias** estén correctamente instaladas (consulta los README correspondientes).

- Los notebooks **asumen ejecución desde su propia carpeta**.  
  Si modificas el *working directory*, **ajusta las rutas** en las primeras celdas del código.

---

> 💡 *Consejo:* se recomienda usar un entorno virtual con las versiones de librerías especificadas en cada README para evitar conflictos de dependencias.



