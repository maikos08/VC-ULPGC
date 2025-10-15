# Práctica 3 - Visión por Computador

Este documento recoge el desarrollo de la tercera práctica de la asignatura **Visión por Computador**.  
La práctica se centra en la **detección y reconocimiento de formas**, con dos tareas principales: identificación de monedas y clasificación de microplásticos utilizando técnicas de procesamiento de imágenes.

---

## Autores
- *Alberto José Rodríguez Ruano*  
- *Miguel Ángel Rodríguez Ruano* 

---

## Tarea 1 - Identificación y Conteo de Monedas con Diferenciación por Color

### Objetivo
Desarrollar un sistema que permita determinar la cantidad de dinero presente en una imagen mediante:
1. **Detección automática** de todas las monedas presentes
2. **Selección interactiva** de una moneda de referencia (1€) mediante clic
3. **Clasificación por valor** utilizando análisis de tamaño y color
4. **Cálculo del valor total** en euros y céntimos
5. **Visualización diferenciada** por color según el valor de las monedas

### Metodología Implementada

#### 1. Detección de Círculos con Transformada de Hough
Se utiliza la transformada de Hough para detectar círculos (monedas) en la imagen:

```python
# Preprocesamiento de la imagen
img_gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
img_blurred = cv2.medianBlur(img_gray, 5)  # Reduce ruido

# Detección de círculos usando Hough
circles = cv2.HoughCircles(
    img_blurred, cv2.HOUGH_GRADIENT, 1, 100, 
    param1=100, param2=50, minRadius=10, maxRadius=150
)
circles = np.uint16(np.around(circles)) if circles is not None else None
```

#### 2. Interfaz de Selección por Clic
Sistema interactivo para selección de moneda de referencia mediante clic del ratón:

```python
# Función callback para detectar clic en moneda
def select_reference_coin(event, x, y, flags, params):
    global reference_coin
    if event == cv2.EVENT_LBUTTONDOWN and circles is not None:
        for idx, (cx, cy, r) in enumerate(circles[0, :]):
            if np.hypot(x - cx, y - cy) <= r:
                reference_coin = (cx, cy, r)
                # Marcar moneda seleccionada
                cv2.circle(display_img, (int(cx), int(cy)), int(r), (0, 0, 255), 3)
                cv2.putText(display_img, "1 Euro", (int(cx - 40), int(cy - r - 10)),
                            cv2.FONT_HERSHEY_TRIPLEX, 0.8, (0, 0, 255), 2)
                return

# Configurar ventana interactiva
cv2.namedWindow("clica en la moneda de 1€", cv2.WINDOW_NORMAL)
cv2.setMouseCallback("clica en la moneda de 1€", select_reference_coin)
```

#### 3. Establecimiento de Escala
Cálculo de correspondencia píxeles-milímetros usando moneda de 1€:

```python
# Radios conocidos de las monedas en milímetros (radios, no diámetros)
COIN_RADII_MM = {
    1: 16.25,    # 1 céntimo
    2: 18.75,    # 2 céntimos  
    5: 21.25,    # 5 céntimos
    10: 19.75,   # 10 céntimos
    20: 22.25,   # 20 céntimos
    50: 24.25,   # 50 céntimos
    100: 23.25,  # 1 euro
    200: 25.75   # 2 euros
}

# Cálculo de tamaño real basado en referencia
radius_mm = r / reference_coin[2] * COIN_RADII_MM[100]
```

#### 4. Clasificación Multimodal (Tamaño + Color)
Análisis combinado para identificación precisa usando valores HSV:

```python
# Función de análisis de color en espacio HSV
def analyze_coin_color(x, y, patch_size=5):
    # Extrae parche de la imagen HSV y calcula promedio
    patch = img_hsv[y1:y2, x1:x2]
    mean_hsv = np.mean(patch, axis=(0, 1))
    h, s, v = mean_hsv  # OpenCV HSV: H 0-180, S/V 0-255
    
    # Normalización: h a 0-360°, s y v a 0-1
    h = h * 2
    s = s / 255.0
    v = v / 255.0
    
    # Clasificación por rangos HSV
    if s < 0.25:
        return "SILVER"  # Monedas plateadas (1€, 2€)
    elif 0 < h < 70 and s > 0.2 and v > 0.2:
        if h < 35:
            return "COPPER"  # Monedas cobrizas (1c, 2c, 5c)
        else:
            return "GOLD"    # Monedas doradas (10c, 20c, 50c)
    else:
        return "UNKNOWN"

# Grupos de monedas por color
COPPER_COINS = [1, 2, 5]      # Monedas cobrizas
GOLD_COINS = [10, 20, 50]     # Monedas doradas  
SILVER_COINS = [100, 200]     # Monedas plateadas/bimetálicas
```

#### 5. Visualización con Códigos de Color
Sistema de diferenciación visual según valor de las monedas:

```python
# Asignación de colores según valor de las monedas
for idx, value, color in final_classifications:
    cx, cy, r = circles[0, idx - 1]
    
    # Colores de bordes según grupo de valor
    if value in [1, 2, 5]:        # Monedas de bajo valor
        border_color = (0, 255, 255)  # Amarillo
    elif value in [10, 20, 50]:   # Monedas de valor medio
        border_color = (255, 0, 0)    # Azul  
    elif value in [100, 200]:     # Monedas de alto valor
        border_color = (0, 0, 255)    # Rojo

    # Dibujar círculo y etiquetas
    cv2.circle(img_final, (int(cx), int(cy)), int(r), border_color, 3)
    label = f"{value//100}eu" if value >= 100 else f"{value}ct"
    text = f"{label}  |  {color[:3]}  |  {radius_mm:.1f}mm"
```

### Características Técnicas

**Parámetros de Detección:**
- Método: Transformada de Hough para círculos
- Parámetros HoughCircles: dp=1, minDist=100, param1=100, param2=50
- Rango de radios: 10-150 píxeles
- Resolución de acumulador: 1 píxel

**Análisis de Color:**
- Espacio HSV para robustez ante iluminación
- Tamaño de parche inicial: 5x5 píxeles
- Tamaño de parche ampliado: 10x10 píxeles
- Puntos de muestreo: centro + 4 puntos radiales a 90% del radio

**Interfaz de Usuario:**
- Selección por clic del ratón en la moneda de 1€
- Ventana OpenCV redimensionable
- Feedback visual inmediato con círculo rojo
- Destrucción automática de ventanas

### Resultados Obtenidos

A continuación se presentan los resultados obtenidos tras aplicar el sistema de detección y clasificación de monedas sobre tres imágenes diferentes, con condiciones de iluminación y calidad variables.

#### Caso 1: Imagen en condiciones controladas

La primera imagen se tomó en un entorno controlado, con fondo blanco y buena iluminación.
El sistema detecta correctamente todas las monedas y clasifica de forma precisa tanto su valor como su color.

![Resultados con fondo blanco](Resultado%20monedas.png)

---

#### Caso 2: Imagen tomada con el móvil (sombras y reflejos pronunciados)

En esta captura, realizada con la cámara del móvil, aparecen **sombras y reflejos** que dificultan la segmentación y el análisis de color.
El sistema muestra varios errores de clasificación, confundiendo algunas monedas de diferente color y valor.

![Resultados con sombras y reflejos - fallos](Resultado%20monedas%202.png)

---

#### Caso 3: Imagen móvil con sombras moderadas

En esta imagen también existen sombras y reflejos.
El sistema logra **clasificar correctamente la mayoría de las monedas**, fallando únicamente en una, lo que demuestra una buena **robustez ante variaciones lumínicas**.

![Resultados con sombras moderadas - aciertos](Resultado%20monedas%203.png)

