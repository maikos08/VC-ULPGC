# Práctica 2 - Visión por Computador

Este documento recoge el desarrollo de la segunda práctica de la asignatura **Visión por Computador**.  
La práctica consiste en tareas orientadas al procesamiento de imágenes utilizando funciones básicas de OpenCV, incluyendo detección de bordes, análisis de píxeles, y desarrollo de demostradores interactivos.  
Cada tarea incluye su propósito, descripción del procedimiento y fragmentos de código relevantes.

---

## Autores
- *Alberto José Rodríguez Ruano*  
- *Miguel Ángel Rodríguez Ruano* 

---

## Tarea 1 - Análisis de píxeles blancos por filas con Canny
Se realiza el conteo de píxeles blancos por filas en la imagen resultante del operador de Canny, determinando las filas con valores superiores al 90% del máximo.

```python
# Obtiene contornos con el operador de Canny
gris = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
canny = cv2.Canny(gris, 100, 200)

# Cuenta píxeles no nulos por fila
row_counts = cv2.reduce(canny, 1, cv2.REDUCE_SUM, dtype=cv2.CV_32SC1) / 255

# Determina el máximo y el umbral
maxFil = row_counts.max()
limit = 0.9 * maxFil
rows = np.where(row_counts[:,0] >= limit)[0]

print(f"Valor máximo de píxeles blancos por fila: {maxFil}")
print(f"Número de filas con >= 0.90*maxFil: {len(rows)}")
```

---

## Tarea 2 - Comparación entre Sobel y Canny
Se aplica el operador Sobel, se umbraliza el resultado y se compara con los resultados obtenidos mediante Canny.

```python
# Aplicar Sobel en X y Y
sobelx = cv2.Sobel(img, cv2.CV_64F, 1, 0, ksize=3)
sobely = cv2.Sobel(img, cv2.CV_64F, 0, 1, ksize=3)
sobel = np.sqrt(sobelx**2 + sobely**2)

# Convertir a 8 bits y umbralizar
sobel_8u = cv2.convertScaleAbs(sobel)
_, sobel_thresh = cv2.threshold(sobel_8u, 50, 255, cv2.THRESH_BINARY)

# Conteo de píxeles no nulos por fila y columna
row_counts = cv2.reduce(sobel_thresh, 1, cv2.REDUCE_SUM, dtype=cv2.CV_32SC1) / 255
col_counts = cv2.reduce(sobel_thresh, 0, cv2.REDUCE_SUM, dtype=cv2.CV_32SC1) / 255
```

**Comparación**: Sobel detecta más detalles internos y texturas, mientras que Canny es más selectivo y detecta principalmente los contornos más prominentes.

---

## Tarea 3 - Demostrador interactivo con múltiples modos
Se desarrolla una aplicación que captura imágenes de la cámara web y permite cambiar entre diferentes modos de procesamiento en tiempo real mediante las teclas numéricas.

```python
vid = cv2.VideoCapture(0)
eliminadorFondo = cv2.createBackgroundSubtractorMOG2(history=100, varThreshold=50, detectShadows=True)

modo = 0  

while True:
    ret, frame = vid.read()
    if not ret: break

    frame = cv2.flip(frame, 1)

    if   modo == 0: salida = frame.copy()
    elif modo == 1:
        objetos = eliminadorFondo.apply(frame)
        salida = cv2.cvtColor(objetos, cv2.COLOR_GRAY2BGR)
    elif modo == 2:
        gris = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        bordes = cv2.Canny(gris, 100, 200)
        salida = cv2.cvtColor(bordes, cv2.COLOR_GRAY2BGR)
    elif modo == 3: salida = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    elif modo == 4: salida = cv2.GaussianBlur(frame, (15,15), 0)
    elif modo == 5: salida = cv2.medianBlur(frame, 15)
    elif modo == 6: salida = cv2.Laplacian(cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY), cv2.CV_64F)
    elif modo == 7: salida = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    elif modo == 8: salida = cv2.cvtColor(frame, cv2.COLOR_BGR2LAB)
    elif modo == 9: salida = cv2.applyColorMap(frame, cv2.COLORMAP_JET)
    else: salida = frame.copy()

    cv2.imshow("Demostrador", salida)
    
    key = cv2.waitKey(20) & 0xFF
    if key == 27: break
    elif key in [ord(str(i)) for i in range(10)]:
        modo = int(chr(key))
```

**Modos disponibles** (cambiar con teclas numéricas 0-9):
- **Modo 0**: Imagen original
- **Modo 1**: Sustracción de fondo (MOG2)
- **Modo 2**: Detección de bordes con Canny
- **Modo 3**: Escala de grises
- **Modo 4**: Desenfoque Gaussiano
- **Modo 5**: Filtro mediano
- **Modo 6**: Operador Laplaciano
- **Modo 7**: Espacio de color HSV
- **Modo 8**: Espacio de color LAB
- **Modo 9**: Mapa de colores JET

---

## Tarea 4 - Reinterpretación inspirada en "My little piece of privacy"
Se implementa un sistema de protección de privacidad que detecta rostros en tiempo real y aplica desenfoque sobre ellos.

```python
face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')
cap = cv2.VideoCapture(0)

while True:
    ret, frame = cap.read()
    if not ret: break
    
    frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    faces = face_cascade.detectMultiScale(frame_rgb, scaleFactor=1.1, minNeighbors=5, minSize=(30, 30))
    
    display_frame = frame.copy()
    
    if len(faces) > 0:
        for (x, y, w, h) in faces:
            roi = frame[y:y+h, x:x+w]
            blurred_roi = cv2.GaussianBlur(roi, (99, 99), 0)
            display_frame[y:y+h, x:x+w] = blurred_roi
            
            cv2.rectangle(display_frame, (x, y), (x+w, y+h), (0, 0, 255), 2)
            cv2.putText(display_frame, "Privacidad activada", (x, y-10), 
                       cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 255), 1)
    
    cv2.putText(display_frame, f"Rostros detectados: {len(faces)}", (10, 30), 
               cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 0), 2)
    
    cv2.imshow('Protección de Privacidad', display_frame)
    
    key = cv2.waitKey(1) & 0xFF
    if key == 27: break
    elif key == ord(' '): pausado = not pausado
```

---

## Instalación y requisitos
Para ejecutar esta práctica es necesario tener instalado **Python y Jupyter** junto con las siguientes librerías:

```bash
pip install opencv-python numpy matplotlib pillow
```

Se requiere disponer de una cámara web para las tareas que implican captura de vídeo en tiempo real.

---

## Fuentes
- [Documentación de OpenCV](https://docs.opencv.org/)  
- [Documentación de NumPy](https://numpy.org/doc/)  
- [Documentación de Matplotlib](https://matplotlib.org/stable/contents.html)  
- [My little piece of privacy](https://www.niklasroy.com/project/88/my-little-piece-of-privacy), por Niklas Roy
- [Messa di voce](https://youtu.be/GfoqiyB1ndE?feature=shared), por Golan Levin y Zachary Lieberman
- [Virtual air guitar](https://youtu.be/FIAmyoEpV5c?feature=shared)
- Ejemplos dispuestos en la base de esta práctica 2 de VC proporcionados por el profesorado 
- Copilot
- ChatGPT
