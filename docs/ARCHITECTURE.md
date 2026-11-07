# Arquitectura de FotoSearch Cloud

[Presentación](../README.md) · [Procedencia](../PROVENANCE.md). Fuente: `main` en `9d44eee`, derivada de `FotoSearch/master` en `339221d`.

## Componentes y flujo

1. Flutter pide permiso de galería y consulta qué activos faltan en el índice.
2. Genera miniaturas JPEG y las transmite a FastAPI (`/index/image` o `/index/images`).
3. El servidor procesa los bytes con CLIP, conserva embeddings/identificadores y descarta las miniaturas del procesamiento ordinario. No es una garantía sobre logs, proxies o backups de una infraestructura futura.
4. `/search` codifica el texto multilingüe, compara vectores con NumPy y devuelve IDs; Flutter recupera los originales desde su galería.
5. Feedback, conceptos e identidad opcional modifican la clasificación según los servicios implementados.

`Backend/app/main.py` expone la API; `services.py` coordina modelos; `vector.py` compara vectores; `query.py` prepara consultas; `database.py` mantiene SQLite. `config.py` concentra variables de entorno. Flutter se implementa principalmente en `frontend/lib/main.dart`.

SQLite incluye `app_metadata`, `photos`, `photo_faces`, `face_identities`, `personal_concepts`, `concept_examples` y `search_feedback`. Los bytes originales no son el índice: los vectores y los datos de personalización sí persisten y requieren protección.

## Modelos y decisiones

CLIP ViT-B/32 genera vectores visuales y el encoder textual multilingüe los alinea con consultas en español y otros idiomas. No hay LLM, fine-tuning ni una API de IA paga obligatoria. Se seleccionan hasta 100 candidatos por score CLIP antes de aplicar personalización y feedback acotado.

FaceNet es opcional; si no carga, la búsqueda general continúa y `/health` lo refleja. Su instalación usa `requirements-face.txt`; requiere revisar compatibilidad del intérprete y de NumPy. El README previo señala Python 3.12 o anterior para esa combinación. No se verificó una instalación nueva aquí.

La traducción remota es opcional y está desactivada por defecto. Requiere `requirements-translation.txt` y `FOTOSEARCH_TRANSLATION_ENABLED=true`; una falla conserva las variantes locales previstas por el código.

## API, acceso y privacidad

La API incluye `/health`, `/index/stats`, `/index/pending`, `/search`, `/feedback`, `/feedback/correction`, `/identities/face/enroll`, `/identities/face` y `/concepts`. No se observa un sistema de cuentas y autorización multiusuario en esas rutas. No debe exponerse una base compartida como servicio público sin diseñar aislamiento y controles.

La configuración de CORS predeterminada es `*`; no es autenticación. El HTTP admitido en Android facilita desarrollo local, pero un alojamiento externo requiere HTTPS y revisar la excepción de red. Las miniaturas abandonan el teléfono; el índice, las identidades faciales y los conceptos quedan en el backend. No debe equipararse esta variante con la privacidad de la inferencia visual local de FotoSearch/production.

## Operación y alojamiento requerido

| Configuración | Uso |
| --- | --- |
| `FOTOSEARCH_HOST`, `FOTOSEARCH_PORT` | Escucha del servidor (`0.0.0.0`, `8000`) |
| `FOTOSEARCH_DB` | SQLite; ruta predeterminada anclada a `Backend/data/fotosearch.db` |
| `FOTOSEARCH_CORS_ORIGINS` | Orígenes permitidos |
| `FOTOSEARCH_CLIP_MODEL`, `FOTOSEARCH_TEXT_MODEL` | Modelos visual y textual |
| `FOTOSEARCH_EMBEDDING_VERSION`, `FOTOSEARCH_QUERY_PIPELINE_VERSION` | Compatibilidad de índice y feedback |
| `FOTOSEARCH_ENABLE_FACE_RECOGNITION`, `FOTOSEARCH_FACE_DEVICE` | Identidad opcional y dispositivo |
| `FOTOSEARCH_*_WEIGHT`, `FOTOSEARCH_*_THRESHOLD` | Ranking; valores en `.env.example` |

`python -m app.main` usa el host/puerto configurados; `FOTOSEARCH_RELOAD=true` habilita recarga en desarrollo. Alternativa: `python -m uvicorn app.main:app --host 0.0.0.0 --port 8000`. La primera ejecución descarga modelos; se necesita acceso a sus fuentes. PyTorch puede utilizar CUDA cuando está disponible.

El alojamiento debe ofrecer memoria suficiente para modelos, procesos Python, persistencia de SQLite, espacio para pesos y una API HTTPS alcanzable por Flutter. Un proceso/contenedor con volumen persistente es una posibilidad que debe dimensionarse; no se afirma que ya exista un servicio hospedado.

Al cambiar versiones/modelos, el backend puede invalidar fotos y feedback para solicitar reindexación; los conceptos incompatibles requieren reconstrucción. Las identidades faciales tienen versión propia. Antes de operar una base existente deben revisarse esas reglas y sus backups; no se ejecutaron cambios de datos aquí.

## Historial, límites y mejoras

`339221d` es la base original; `9d44eee`, de Juan David Reyes Cure, registra copia y procedencia. El resto de la autoría se conserva conforme a Git. La documentación consolida la explicación y operación; las pruebas y `PROVENANCE.md` permanecen.

La calidad de CLIP no garantiza distinguir atributos finos; la detección facial puede fallar. Grandes galerías requieren evaluar indexación por lotes y un índice vectorial especializado. Antes de publicar faltan revisión de licencias de modelos, privacidad, autenticación/aislamiento y comprobación de infraestructura. No se atribuyen mejoras propuestas a funcionalidades terminadas.
