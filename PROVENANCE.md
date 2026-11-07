# FotoSearch Cloud

Variante recuperada de `IngenieroCui/FotoSearch`, rama `master`, commit
`339221dd53c74c8863ee717bb909386d114ff01d`. Conserva el historial original.

Incluye Flutter Android y backend FastAPI/Python con CLIP y SQLite. El backend
procesa miniaturas temporales. No se ha cambiado su arquitectura ni ejecutado
pruebas durante esta migración. Requiere un alojamiento compatible con los
modelos y la persistencia; no es una aplicación Android ejecutable en Vercel.

La rama `production` del repositorio original se conserva sin cambios. Esa rama
mantiene inferencia visual local, pero aún solicita embeddings de texto a una API;
no se presenta como implementación totalmente offline.
