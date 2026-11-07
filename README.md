# FotoSearch Cloud

**Búsqueda semántica de una galería móvil con procesamiento CLIP en un backend propio.**

## Problema y solución

Para buscar fotos por su contenido, Flutter envía miniaturas temporales a FastAPI. El servidor genera embeddings y guarda un índice SQLite; devuelve referencias de galería para que el teléfono muestre sus originales.

## Funcionalidades

- Indexación de miniaturas, consulta textual multilingüe y ranking semántico.
- Feedback de resultados y conceptos personales sin entrenar una red nueva.
- Identidad facial opcional con FaceNet, sujeta a dependencias compatibles.
- Estado del índice y detección de cambios de versión para reindexación.

## Tecnologías y arquitectura

`Flutter/Dart → FastAPI/Python → CLIP + NumPy → SQLite`.

El encoder visual es `clip-ViT-B-32` y el textual multilingüe es `clip-ViT-B-32-multilingual-v1`. Los originales permanecen en el teléfono; **las miniaturas sí se transmiten** y sus bytes se procesan temporalmente. Embeddings, IDs y personalización persisten en el servidor.

## Ejecución local

```powershell
cd Backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
python -m app.main
```

El backend escucha en el puerto 8000 por defecto. La primera ejecución necesita descargar modelos; después puede inferir localmente en el servidor.

En `frontend`, ejecuta `flutter pub get`, crea el `.env` que espera la app y configura `API_BASE_URL` con una dirección alcanzable desde el teléfono. Ejecuta `flutter run`. El archivo se declara como asset: nunca incluyas secretos en él.

## Estado y límites

La rama `main` conserva la variante de `FotoSearch/master`, con [procedencia y créditos](PROVENANCE.md). **Cloud describe la variante; no acredita alojamiento activo.** Requiere un servidor con memoria para PyTorch/CLIP, almacenamiento persistente y conectividad; no es una app Android ejecutable en Vercel.

El HTTP de desarrollo en red local debe revisarse para exposición pública. No hay aislamiento multiusuario acreditado. Los embeddings faciales y conceptos personales requieren tratamiento como datos sensibles. No se ejecutaron pruebas, builds ni descargas de modelos en este sprint.

## Contribuciones

Se conserva el historial de Andrecodev. Juan David Reyes Cure registra en `9d44eee` la identificación de esta copia y su procedencia; ese commit no se presenta como autoría de CLIP o del desarrollo anterior.

[Arquitectura, operación y límites](docs/ARCHITECTURE.md).
