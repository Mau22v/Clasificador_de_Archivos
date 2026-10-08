# Clasificador Distribuido de Archivos Cientificos

Sistema distribuido para clasificacion de PDFs cientificos con FastAPI, consenso por mayoria entre workers y eleccion de lider con algoritmo Bully.

**Nota sobre este repositorio:** Este proyecto fue desarrollado en colaboración. En este *fork* destaco la arquitectura general y mis contribuciones específicas al diseño del sistema.

## Mi Rol y Contribuciones

Durante el desarrollo de este sistema, fui responsable de:
* **Arquitectura de Microservicios:** Mejoras en la implementación de la comunicación entre el nodo *Master* (API de negocio y ruteo) y los nodos *Workers* (procesamiento y clasificación).
* **Desarrollo Frontend:** Creación e integración de la interfaz de usuario para interactuar con el sistema distribuido, facilitando la carga de archivos y la visualización de resultados.
* **Clasificación de Metadatos (Python):** Desarrollo de la lógica de extracción y categorización de archivos científicos basándose en su contenido y metadatos.

## 🛠️ Stack Tecnológico
* **Backend:** Python, FastAPI, Uvicorn
* **Frontend:** Interfaz web de usuario
* **Base de Datos & Auth:** Supabase
* **Sistemas Distribuidos:** Algoritmo Bully (Elección de líder), Consenso por mayoría.
* 
## Arquitectura

```text
clasificador-final/
├── master/
│   ├── __init__.py
│   ├── main.py
│   ├── routes.py
│   ├── auth.py
│   ├── gateway.py
│   ├── consensus.py
│   ├── adapter.py
│   ├── database.py
│   ├── deletion_coordinator.py
│   └── apa.py
├── worker/
│   ├── __init__.py
│   ├── main.py
│   ├── extractor.py
│   ├── classifier.py
│   └── sync.py
├── shared/
│   ├── __init__.py
│   ├── election.py
│   └── leader_db.py
├── metadata/
├── storage/
├── frontend/
├── requirements.txt
└── CRONOGRAMA.md
```

## Componentes clave

- master: API de negocio, autenticacion, gestion de areas, documentos y administracion.
- worker: procesamiento de PDF (extraccion + clasificacion).
- shared/election.py: deteccion de caida de lider y eleccion automatica.
- shared/leader_db.py: registro del lider activo y heartbeat en Supabase.
- master/consensus.py: votacion entre workers para clasificacion final.

## Instalacion

```bash
pip install -r requirements.txt
```

## Ejecucion local

Abrir 4 terminales desde la raiz. Cada proceso debe usar un `NODO_ID` distinto; el nodo con mayor ID disponible es el que puede quedar como lider si el actual cae:

```bash
uvicorn worker.main:app --port 5001
uvicorn worker.main:app --port 5002
uvicorn worker.main:app --port 5003
uvicorn master.main:app --port 8000
```

Documentacion OpenAPI:

```text
http://localhost:8000/docs
```

## Variables de entorno

- NODO_ID: id del nodo actual para eleccion de lider.
- SUPABASE_URL: URL del proyecto Supabase.
- SUPABASE_KEY: clave de acceso para operaciones de lider.
- ALMACENAMIENTO_NODO: ruta local del nodo worker.

## Endpoints principales

- Auth: /register, /login, /logout
- Areas: /categories, /areas, /areas/{area}/sub
- Documentos: /upload, /files, /download, /document
- Admin: /admin/users, /admin/areas/*
- Eleccion de lider: /heartbeat, /leader, /election/start, /election/coordinator

## Flujo funcional

1. Usuario autentica y obtiene token.
2. Master valida archivo y metadatos.
3. Master envia el PDF a workers y calcula mayoria.
4. PDF se replica en storage/node1-node3.
5. Metadatos y sesion se guardan en Supabase (tablas usuarios, tokens_sesion, tematicas, subtematicas, documentos, nodos_almacenamiento y consenso_votos).
6. Si el lider cae, cualquier worker o el master puede asumir el rol, siempre que tenga el `NODO_ID` correcto y gane la eleccion Bully.

## Documentacion complementaria

- CRONOGRAMA.md
- RESUMEN_EJECUTIVO.md
- INDICE_DOCUMENTACION.md
- DOCUMENTACION_TECNICA.md
