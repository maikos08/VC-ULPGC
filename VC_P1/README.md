# Práctica 1 - Visión por Computador

Este documento recoge el desarrollo de la primera práctica de la asignatura **Visión por Computador**.  
La práctica consiste en 5 tareas orientadas al tratamiento de imágenes y vídeo en Python, utilizando librerías especializadas en visión artificial.  
Cada tarea incluye su propósito, descripción del procedimiento y fragmentos de código relevantes.

---

## Autores
- *Alberto José Rodríguez Ruano*  
- *Miguel Ángel Rodríguez Ruano* 

---

## Tarea 1 - Generar un tablero de ajedrez
Se crea un tablero de ajedrez en escala de grises de 8x8 casillas, rellenando las posiciones blancas mediante bucles anidados.

```python
gris_img = np.zeros((800,800,1), dtype=np.uint8)
height, width = 100, 100

for i in range(8):
    for j in range(8):
        if (i % 2 == 0 and j % 2 == 0) or (i % 2 != 0 and j % 2 != 0):
            x_start, x_end = i * width, (i + 1) * width
            y_start, y_end = j * height, (j + 1) * height
            gris_img[x_start:x_end, y_start:y_end, 0] = 255

plt.imshow(gris_img, cmap='gray')
plt.show()
```

---

## Tarea 2 - Crear una imagen estilo Mondrian
Se genera una composición inspirada en Mondrian utilizando rectángulos de colores primarios y blanco.

```python
color_img = np.zeros((325,400,3), dtype=np.uint8)

cv2.rectangle(color_img,(0,0),(15,100),(255,255,255),-1)
cv2.rectangle(color_img,(25,0),(80,100),(255,255,0),-1)
cv2.rectangle(color_img,(90,0),(200,100),(0,0,255),-1)
cv2.rectangle(color_img,(210,0),(330,100),(255,0,0),-1)

# Para ver todos los rectángulos del ejercicio entrar en el entregable (archivo VC_P1.ipynb)
...

plt.imshow(color_img)
plt.show()
```

---

## Tarea 3 - Modificar valores de un plano de la imagen
Se captura vídeo en tiempo real desde la cámara y se sustituye el canal azul por el canal rojo.

```python
vid = cv2.VideoCapture(0)

while True:
    ret, frame = vid.read()
    if ret:
        frame[:,:,0] = frame[:,:,2]  # Canal azul sustituido por rojo
        cv2.imshow('WebCam', frame)
    if cv2.waitKey(20) == 27:
        break

vid.release()
cv2.destroyAllWindows()
```

---

## Tarea 4 - Detectar píxeles más claros y oscuros
Se convierte el vídeo en escala de grises y se localizan los píxeles con menor y mayor intensidad.  
Se marcan con círculos de colores en la imagen original.

```python
vid = cv2.VideoCapture(0)

while True:
    ret, frameIN = vid.read()
    if not ret:
        break

    gray = cv2.cvtColor(frameIN, cv2.COLOR_BGR2GRAY)
    minVal, maxVal, minLoc, maxLoc = cv2.minMaxLoc(gray)

    cv2.circle(frameIN, minLoc, 5, (255,0,0), 2)  # Azul en el píxel más oscuro
    cv2.circle(frameIN, maxLoc, 5, (0,0,255), 2)  # Rojo en el píxel más claro

    cv2.imshow('Cam', frameIN)
    if cv2.waitKey(20) == 27:
        break

vid.release()
cv2.destroyAllWindows()
```

---

## Tarea 5 - Propuesta propia de Pop Art
Se implementa un efecto **Pop Art** en tiempo real dividiendo la pantalla en un collage de 18 cuadros (3 filas x 6 columnas).  
En la parte izquierda se aplican transformaciones de color sobre los canales RGB y en la parte derecha diferentes variaciones en escala de grises.

```python
vid = cv2.VideoCapture(0)
w = int(vid.get(cv2.CAP_PROP_FRAME_WIDTH) / 6)
h = int(vid.get(cv2.CAP_PROP_FRAME_HEIGHT) / 3)

collage = np.zeros((h*3, w*6, 3), dtype=np.uint8)

while True:
    ret, frame = vid.read()
    if not ret: break
    frame = cv2.resize(frame, (w, h))
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    gray = cv2.cvtColor(gray, cv2.COLOR_GRAY2BGR)

    # Ejemplo de transformaciones
    # Para ver todas las propuestas en el ejercicio entrar en el entregable (archivo VC_P1.ipynb)
    collage[0:h, 0:w] = [frame[:,:,0], frame[:,:,1], 255 - frame[:,:,2]]
    collage[0:h, 3*w:4*w] = gray
    collage[h:2*h, 4*w:5*w] = cv2.convertScaleAbs(gray, alpha=0.5, beta=100)

    cv2.imshow('Cam', collage)
    if cv2.waitKey(20) == 27: break

vid.release()
cv2.destroyAllWindows()
```

---

## Instalación y requisitos
Para ejecutar esta práctica es necesario tener instalado **Python y Jupyter** junto con las siguientes librerías:

```bash
pip install opencv-python numpy matplotlib
```

Se requiere disponer de una cámara web para las tareas que implican captura de vídeo.

---

## Fuentes
- [Documentación de OpenCV](https://docs.opencv.org/)  
- [Documentación de NumPy](https://numpy.org/doc/)  
- [Documentación de Matplotlib](https://matplotlib.org/stable/contents.html)  
- Ejemplos dispuestos en la base de esta práctica 1 de VC proporcionados por el profesorado 
- Copilot
- ChatGPT  

