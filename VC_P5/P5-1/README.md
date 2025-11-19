# P5-1 — Filtros y prototipos basados en detección facial

## Objetivo de este primer ejercicio

La primera tarea que hemos hecho consiste en proponer **dos escenarios de aplicación** y desarrollar **dos prototipos** (temática libre) que provoquen reacciones o efectos a partir de la información extraída del rostro.

El notebook incluido en esta carpeta (`VC_P5-filtro_basico.ipynb`) trabaja con modelos **preentrenados** (MediaPipe y utilidades de OpenCV). En esta práctica implementamos **dos filtros** basados en detectores preentrenados — ambos son ejemplos con modelos ya entrenados, no hay aquí un modelo entrenado por el equipo. Tened esto en cuenta: el enunciado del ejercicio pide que uno de los prototipos que entreguéis use un modelo entrenado por vosotros; ese modelo no está incluido en este repositorio y debe desarrollarse/añadirse por separado.

---

## Qué hay en esta carpeta

- `VC_P5-filtro_basico.ipynb`: Notebook con demos y funciones útiles:
  - detección de landmarks faciales con MediaPipe
  - superposición de imágenes (filtros tipo "koala", gafas, etc.)
  - funciones de rotación y ajuste por landmark
  - demos en tiempo real usando la webcam

---

## Requisitos (rápido)

Se recomienda usar un entorno virtual (conda). Dependencias mínimas (instaladlas por separado):

```bash
pip install opencv-python
pip install mediapipe
pip install matplotlib
```

---

## Pasos para la ejecución de ambas demos

Primero se encuentra la demo con el apartado de los monos y la demo con el filtro del koala. Para poner las demos a funcionar se deben ejecutar todos los bloques en orden. Sin ninguna excepción.

---

## Demo de monos

Descripción
- Propósito: demostrar detección de landmarks faciales y de manos con MediaPipe Holistic, y usar la información (expresión y gestos) para seleccionar y mostrar imágenes de "monos" que representen el estado detectado.
- Comportamiento: la demo muestra en tiempo real la cámara (lado izquierdo) y, a la derecha, la imagen del mono correspondiente al estado (neutral, feliz, sorprendido, pensando, enfadado). Hay un suavizado temporal para estabilizar el estado mostrado.

Fragmento de código (ejecución en tiempo real — versión resumida):

```python
# Iniciar webcam
cap = cv2.VideoCapture(0)
historial = deque(maxlen=3)

while True:
  ret, frame = cap.read()
  if not ret:
    break
  frame = cv2.flip(frame, 1)
  image_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
  results = holistic.process(image_rgb)

  # Dibujar landmarks en frame
  annotated_frame = frame.copy()
  if results.face_landmarks:
    mp_drawing.draw_landmarks(
      annotated_frame, results.face_landmarks,
      mp_holistic.FACEMESH_CONTOURS,
      connection_drawing_spec=mp_drawing_styles.get_default_face_mesh_contours_style()
    )

  # Detectar expresión y suavizar
  estado = detectar_expresion(results.face_landmarks,
                results.left_hand_landmarks,
                results.right_hand_landmarks)
  historial.append(estado)
  estado_actual = Counter(historial).most_common(1)[0][0]

  # Seleccionar imagen del mono y combinar lado a lado
  mono_img = monos.get(estado_actual, monos['neutral'])
  mono_resized = redimensionar_imagen(mono_img, annotated_frame.shape[0])
  combined = np.hstack((annotated_frame, mono_resized)) if mono_resized is not None else annotated_frame

  cv2.imshow('Espejo con el mono', combined)
  if cv2.waitKey(1) & 0xFF == ord('q'):
    break

cap.release()
cv2.destroyAllWindows()
```

Dónde colocar recursos
- GIF de la demo (máx. 30s): `P5-1/demos/monos_demo.gif`
- Imágenes de ejemplo (miniaturas): `P5-1/images/monos_1.png`, `P5-1/images/monos_2.png`

Placeholder (coloca tus ficheros en las rutas anteriores):

```
![Demo monos (gif)](demos/monos_demo.gif)
![Monos - ejemplo 1](images/monos_1.png)
![Monos - ejemplo 2](images/monos_2.png)
```

---
A continuación se amplían las explicaciones y se describen las dos demos relacionadas con los "monos" que hay en el notebook: la parte de procesamiento por lotes de imágenes estáticas y las dos demos en tiempo real (lado a lado y superposición).

### Demo A — Procesamiento de imágenes estáticas (carga y anotación)

Descripción
- Objetivo: probar cómo MediaPipe Holistic detecta rostros y manos en imágenes estáticas (no webcam). Permite validar la robustez del detector frente a distintas expresiones y poses (útil para crear dataset de ejemplo o para depuración).

Qué hace exactamente
- Carga una lista de rutas de imágenes (p. ej. `../images/mono_*.jpg`) y ejecuta `holistic.process()` en cada imagen.
- Dibuja los landmarks faciales y de manos sobre la imagen usando `mp_drawing.draw_landmarks()` con estilos predefinidos.
- Guarda las imágenes anotadas con un prefijo como `detectado_<original>` para revisarlas posteriormente.

Fragmento de código (procesamiento por lotes):

```python
mp_holistic = mp.solutions.holistic
holistic = mp_holistic.Holistic(static_image_mode=True, model_complexity=2,
                min_detection_confidence=0.3)

imagenes = ['../images/mono_sorprendido.jpg', '../images/mono_feliz.png']
for img_path in imagenes:
  image = cv2.imread(img_path)
  image_rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
  results = holistic.process(image_rgb)
  annotated = image.copy()
  if results.face_landmarks:
    mp_drawing.draw_landmarks(annotated, results.face_landmarks,
                  mp_holistic.FACEMESH_CONTOURS)
  cv2.imwrite(f'detectado_{os.path.basename(img_path)}', annotated)

holistic.close()
```

Consejos y parámetros importantes
- `model_complexity`: 0-2 (más alto = más preciso, más lento). Para imágenes estáticas usar 2 si dispones de tiempo.
- `min_detection_confidence`: ajustar si detecta falsos positivos en fondo complejo.
- Guarda las anotaciones para inspección manual y para crear conjuntos de entrenamiento o validación.

Placeholder (imagenes generadas al procesar estáticas):

```
![Monos - anotado 1](images/monos_annotated_1.png)
![Monos - anotado 2](images/monos_annotated_2.png)
```

### Demo B — Webcam lado a lado (comparativa detección vs resultado)

Descripción
- Propósito: visualizar en vivo cómo los landmarks se proyectan sobre el frame y simultáneamente mostrar una representación visual del estado (p. ej. la imagen del mono) para comparar la detección frente al resultado aplicado.

Detalles técnicos
- Usa `holistic` en modo de vídeo (static_image_mode=False) con `model_complexity=1` para equilibrar latencia y calidad.
- Suavizado temporal: se guarda un buffer corto (`deque(maxlen=3)`) y se toma la moda del buffer para evitar cambios bruscos entre frames.
- Visualmente se combina el frame anotado y la imagen representativa mediante `np.hstack()`.

Puntos de mejora que podéis intentar
- Ajustar el tamaño del buffer para suavizado (2-5). Mayor suavizado = menos reactividad.
- Añadir thresholds para considerar gesto solo si hay confianza alta en landmarks.

Placeholder (toma de pantalla / gif corto):

```
![Monos - lado a lado](demos/monos_side_by_side.gif)
```

### Demo C — Superposición (filtro tipo máscara)

Descripción
- Propósito: demostrar la capacidad de superponer una gráfica (imagen RGBA) sobre la cara detectada usando transformaciones geométricas (rotación, escalado y posicionamiento basado en landmarks).
- Uso típico: filtros tipo máscara, elementos estéticos o elementos informativos (p. ej. indicador de emoción sobre la frente).

Funciones clave en el notebook
- `calcular_angulo_inclinacion(face_landmarks, w, h)`: devuelve ángulo de la cabeza calculado a partir de la posición de los ojos.
- `rotar_imagen_con_alpha(img, angulo)`: rota imágenes que contienen canal alpha manteniendo transparencia.
- `superponer_con_transparencia(frame, overlay, x, y, ancho, alto)`: superpone `overlay` (RGBA) sobre `frame` con manejo de alpha.

Recomendaciones
- Para capas complejas (gafas con sombras) preprocesar la imagen de overlay con `eliminar_fondo_gafas()` o similares para limpiar el canal alpha.
- Calibrar `ancho_filtro` y `alto_filtro` (multiplicadores usados en el notebook) si la máscara no encaja correctamente en caras con distinta proporción.

Placeholder (ejemplo de superposición / gif):

```
![Monos - superposicion](demos/monos_overlay.gif)
```

---
## Demo de filtro de koala

Descripción
- Propósito: superponer una máscara (orejas y nariz de koala) sobre la cara detectada y permitir activar/desactivar unas gafas de sol. Usa MediaPipe FaceMesh para landmarks finos y varias funciones de transformación geométrica para rotación/escala y eliminación de fondo en los gráficos superpuestos.
- Controles en tiempo real: `1` para activar/desactivar las gafas, `D` para alternar modo debug (muestra rectángulos y puntos), `Q` para salir.

Fragmento de código (versión resumida del bucle principal):

```python
cap = cv2.VideoCapture(0)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)

debug_mode = False
gafas_activas = False

while True:
  ret, frame = cap.read()
  if not ret:
    break
  frame = cv2.flip(frame, 1)
  image_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
  results = face_mesh.process(image_rgb)

  if results.multi_face_landmarks:
    face_landmarks = results.multi_face_landmarks[0]
    angulo, _, _ = calcular_angulo_inclinacion(face_landmarks, *frame.shape[:2])
    frame_con_filtro = aplicar_filtro_koala_con_rotacion(frame, face_landmarks, filtro_koala, debug_mode)
    if gafas_activas and gafas_sol is not None:
      frame_con_filtro = aplicar_gafas_sol(frame_con_filtro, face_landmarks, gafas_sol, angulo, debug_mode)
  else:
    frame_con_filtro = frame

  cv2.putText(frame_con_filtro, '1:gafas D:debug Q:salir', (10, frame.shape[0] - 20),
        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255,255,255), 2)
  cv2.imshow('Filtro koala + gafas', frame_con_filtro)

  key = cv2.waitKey(1) & 0xFF
  if key == ord('q'):
    break
  elif key == ord('1'):
    gafas_activas = not gafas_activas
  elif key == ord('d'):
    debug_mode = not debug_mode

cap.release()
cv2.destroyAllWindows()
```

Dónde colocar recursos
- GIF de la demo (máx. 30s): `P5-1/demos/koala_demo.gif`
- Imágenes de ejemplo (miniaturas): `P5-1/images/koala_1.png`, `P5-1/images/koala_2.png`

Placeholder (coloca tus ficheros en las rutas anteriores):

```
![Demo koala (gif)](demos/koala_demo.gif)
![Koala - ejemplo 1](images/koala_1.png)
![Koala - ejemplo 2](images/koala_2.png)
```

---

Si quieres que inserte las miniaturas y el reproductor directamente en el README cuando subas los ficheros, lo hago por ti (indica las rutas exactas). También puedo crear `P5-1/demos/run_demo.cmd` para arrancar la demo en Windows si lo deseas.




