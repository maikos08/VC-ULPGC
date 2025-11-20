# Práctica 5 — Índice introductorio

## 👥 Autores
- **Alberto José Rodríguez Ruano**  
- **Miguel Ángel Rodríguez Ruano**

---

Este README es una página índice: aquí encontrarás enlaces directos a los notebooks y READMEs de cada subcarpeta y una breve introducción sobre qué contiene cada una y qué se espera entregar.

## Carpetas y notebooks principales

- [P5-1/ — Demos con detectores preentrenados](P5-1/README.md)
	- Notebook principal: [P5-1/VC_P5-filtro_basico.ipynb](P5-1/VC_P5-filtro_basico.ipynb)
	- Introducción: ejemplos y utilidades para detección de landmarks (MediaPipe), filtros y prototipado rápido.
	- Qué se espera: usar estos ejemplos como punto de partida para diseñar prototipos; en particular, P5-1 contiene demos que usan modelos preentrenados — si uno de vuestros entregables debe usar un modelo propio, este deberá desarrollarse y añadirse separadamente.

- [P5-2/ — Prototipos con modelo propio](P5-2/README.md)
	- Notebooks principales:
		- [P5-2/VC_P5_emotion_detector.ipynb](P5-2/VC_P5_emotion_detector.ipynb) — prototipo 1 (fondos emocionales + efectos).
		- [P5-2/VC_P5_emotion_detector_monkeys.ipynb](P5-2/VC_P5_emotion_detector_monkeys.ipynb) — prototipo 2 (monos lado a lado, comparativa).
	- Pesos del modelo: [P5-2/best_model_emotions.pth](P5-2/best_model_emotions.pth)
	- Introducción: implementa un clasificador de emociones entrenado por el equipo (ej.: ResNet50) y demos que usan esa salida para generar efectos.
	- Qué se espera: incluir documentación del entrenamiento (dataset, arquitectura, hiperparámetros) y una demo corta que muestre la inferencia del modelo.

## Qué encontrarás en cada README

- [P5-1/README.md](P5-1/README.md): explicación del ejercicio, instrucciones de ejecución, requerimientos mínimos y descripción de las demos incluidas (koala, monos). Ideal para empezar a probar las demos en local.
- [P5-2/README.md](P5-2/README.md): descripción del modelo propio, instrucciones de instalación (PyTorch, DeepFace opcional), cómo ejecutar los notebooks y notas sobre el entreno y dataset.

---

## Entregables esperados (resumen)

- Código y notebooks de los dos prototipos (uno de ellos debe usar un modelo entrenado por vosotros).
- Un GIF o vídeo corto (≤ 30s) mostrando fragmentos representativos de cada prototipo.
- Una memoria breve explicando la idea, arquitectura y cómo ejecutar las demos.
