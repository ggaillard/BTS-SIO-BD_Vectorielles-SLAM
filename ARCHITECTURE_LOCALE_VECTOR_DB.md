# 🏗️ ARCHITECTURE LOCALE - VECTOR DATABASE
## Setup complet avec Docker Compose + Scripts Python

---

## 📋 TABLE DES MATIÈRES

1. [Setup rapide (5 min)](#setup-rapide)
2. [Architecture Docker](#architecture-docker)
3. [Scripts Python complets](#scripts-python)
4. [Tests et validation](#tests-et-validation)
5. [Dépannage](#dépannage)

---

## SETUP RAPIDE

### Option 1: Chroma (Plus simple, recommandé pour TP)

**Installation (1 min):**
```bash
# Créer dossier
mkdir vector-db-local
cd vector-db-local

# Virtual environment
python3 -m venv venv
source venv/bin/activate

# Installer Chroma
pip install chromadb numpy pandas
```

**Script test immédiat:**
```python
# test_chroma.py
import chromadb
import numpy as np

# Initier (crée stockage local automatique)
client = chromadb.Client()
collection = client.create_collection(name="spotify")

# Ajouter chansons
collection.add(
    ids=["1", "2", "3"],
    documents=["Shape of You", "Thinking Out Loud", "Perfect"],
    embeddings=np.random.randn(3, 128).tolist()
)

# Requête
results = collection.query(query_embeddings=np.random.randn(1, 128).tolist(), n_results=2)
print("Résultats:", results["documents"])
```

**Exécuter:**
```bash
python test_chroma.py
```

**Avantages:**
- ✓ Installation ultra-simple
- ✓ Stockage local (fichiers)
- ✓ Pas Docker requis
- ✓ Parfait pour TP/learning

---

### Option 2: Qdrant + Docker (Recommandé pour production-like)

**Prérequis:**
- Docker installé (https://docker.com)

**Setup (3 min):**
```bash
# 1. Créer dossier projet
mkdir vector-db-docker
cd vector-db-docker

# 2. Lancer Qdrant
docker run -p 6333:6333 qdrant/qdrant

# 3. Vérifier en navigateur ou curl
curl http://localhost:6333/api/health
# → {"status":"ok"}
```

**Script Python test:**
```python
# test_qdrant.py
from qdrant_client import QdrantClient
from qdrant_client.models import PointStruct
import numpy as np

# Se connecter
client = QdrantClient("localhost", port=6333)

# Créer collection
client.recreate_collection(
    collection_name="spotify",
    vectors_config={"size": 128, "distance": "Cosine"}
)

# Ajouter points
points = [
    PointStruct(id=1, vector=np.random.randn(128).tolist(), 
                payload={"titre": "Shape of You"}),
    PointStruct(id=2, vector=np.random.randn(128).tolist(),
                payload={"titre": "Thinking Out Loud"}),
]
client.upsert(collection_name="spotify", points=points)

# Requête
results = client.search(
    collection_name="spotify",
    query_vector=np.random.randn(128).tolist(),
    limit=2
)

for hit in results:
    print(f"{hit.payload['titre']} (score: {hit.score:.3f})")
```

**Exécuter:**
```bash
# Install client
pip install qdrant-client

# Run
python test_qdrant.py
```

**Avantages:**
- ✓ Scalable
- ✓ Production-ready
- ✓ API Web intégrée
- ✓ Persistance data

---

## ARCHITECTURE DOCKER

### 1. Structure recommandée

```
vector-db-local/
├── docker-compose.yml          ← Configuration
├── api/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py                  ← Flask API
├── ui/
│   ├── Dockerfile
│   └── streamlit_app.py         ← Interface
└── data/
    ├── qdrant/                 ← Persistance Qdrant
    └── postgres/               ← Persistance PostgreSQL
```

### 2. docker-compose.yml complet

```yaml
version: '3.8'

services:
  # ===== VECTOR DB =====
  qdrant:
    image: qdrant/qdrant:v1.7.0
    ports:
      - "6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage
    environment:
      QDRANT_API_KEY: "test-key-for-dev"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:6333/api/health"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ===== METADATA DATABASE =====
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: vectordb_user
      POSTGRES_PASSWORD: vectordb_password
      POSTGRES_DB: vectordb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U vectordb_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ===== API BACKEND =====
  api:
    build: ./api
    ports:
      - "5000:5000"
    depends_on:
      qdrant:
        condition: service_healthy
      postgres:
        condition: service_healthy
    environment:
      QDRANT_URL: http://qdrant:6333
      POSTGRES_URL: postgresql://vectordb_user:vectordb_password@postgres:5432/vectordb
      FLASK_ENV: development
    volumes:
      - ./api:/app
    command: python app.py

  # ===== WEB UI =====
  ui:
    build: ./ui
    ports:
      - "8501:8501"
    depends_on:
      - api
    environment:
      API_URL: http://api:5000
    volumes:
      - ./ui:/app
    command: streamlit run streamlit_app.py --server.port=8501 --server.address=0.0.0.0

volumes:
  qdrant_data:
  postgres_data:
```

### 3. Démarrer architecture complète

```bash
# Lancer tous les services
docker-compose up -d

# Vérifier
docker-compose ps

# Logs
docker-compose logs -f

# Arrêter
docker-compose down
```

**Vérifier services:**
```bash
# Qdrant
curl http://localhost:6333/api/health
# → {"status":"ok"}

# PostgreSQL
psql -h localhost -U vectordb_user -d vectordb
# → psql (15.x)

# API Flask
curl http://localhost:5000/health
# → {"status":"ok"}

# UI Streamlit
open http://localhost:8501
```

---

## SCRIPTS PYTHON

### 1. API Flask - api/app.py

```python
from flask import Flask, request, jsonify
from qdrant_client import QdrantClient
from qdrant_client.models import PointStruct, Distance, VectorParams
import numpy as np
import psycopg2
from psycopg2.extras import RealDictCursor
import os
import json

app = Flask(__name__)

# ===== CONFIGURATION =====
QDRANT_URL = os.getenv("QDRANT_URL", "http://localhost:6333")
POSTGRES_URL = os.getenv("POSTGRES_URL", "postgresql://vectordb_user:vectordb_password@postgres:5432/vectordb")

# ===== CLIENTS =====
qdrant_client = QdrantClient(QDRANT_URL)
COLLECTION_NAME = "spotify"

def get_db_connection():
    """Connexion PostgreSQL"""
    return psycopg2.connect(POSTGRES_URL)

# ===== INIT DATABASE =====
def init_qdrant():
    """Créer collection si n'existe pas"""
    try:
        qdrant_client.get_collection(COLLECTION_NAME)
    except:
        print(f"Créant collection {COLLECTION_NAME}...")
        qdrant_client.create_collection(
            collection_name=COLLECTION_NAME,
            vectors_config=VectorParams(size=128, distance=Distance.COSINE),
        )

def init_postgres():
    """Créer tables PostgreSQL"""
    conn = get_db_connection()
    cur = conn.cursor()
    
    cur.execute("""
    CREATE TABLE IF NOT EXISTS chansons (
        id SERIAL PRIMARY KEY,
        qdrant_id INTEGER UNIQUE,
        titre VARCHAR(255),
        artiste VARCHAR(255),
        genre VARCHAR(100),
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
    """)
    
    conn.commit()
    cur.close()
    conn.close()
    print("✓ Tables PostgreSQL initialisées")

# ===== ENDPOINTS =====

@app.route('/health', methods=['GET'])
def health():
    """Vérifier état services"""
    return jsonify({
        "status": "ok",
        "qdrant": "connected",
        "postgres": "connected"
    })

@app.route('/upload', methods=['POST'])
def upload_chansons():
    """Upload chansons + embeddings"""
    data = request.json  # {chansons: [{titre, artiste, embedding},...]}
    
    points = []
    conn = get_db_connection()
    cur = conn.cursor()
    
    for i, chanson in enumerate(data['chansons']):
        # Insérer metadata dans PostgreSQL
        cur.execute("""
        INSERT INTO chansons (qdrant_id, titre, artiste, genre)
        VALUES (%s, %s, %s, %s)
        ON CONFLICT (qdrant_id) DO NOTHING
        """, (i, chanson['titre'], chanson['artiste'], chanson.get('genre', 'unknown')))
        
        # Préparer vecteur pour Qdrant
        points.append(PointStruct(
            id=i,
            vector=chanson['embedding'],
            payload={"titre": chanson['titre'], "artiste": chanson['artiste']}
        ))
    
    # Uploader dans Qdrant
    qdrant_client.upsert(
        collection_name=COLLECTION_NAME,
        points=points
    )
    
    conn.commit()
    cur.close()
    conn.close()
    
    return jsonify({
        "status": "success",
        "uploaded": len(points)
    })

@app.route('/search', methods=['POST'])
def search():
    """Chercher chansons similaires"""
    data = request.json  # {query_vector: [...], top_k: 10}
    
    results = qdrant_client.search(
        collection_name=COLLECTION_NAME,
        query_vector=data['query_vector'],
        limit=data.get('top_k', 10)
    )
    
    # Enrichir avec metadata PostgreSQL
    conn = get_db_connection()
    cur = conn.cursor(cursor_factory=RealDictCursor)
    
    response = []
    for hit in results:
        cur.execute("SELECT * FROM chansons WHERE qdrant_id = %s", (hit.id,))
        metadata = cur.fetchone()
        
        response.append({
            "titre": hit.payload['titre'],
            "artiste": hit.payload['artiste'],
            "similarity": hit.score,
            "metadata": dict(metadata) if metadata else {}
        })
    
    cur.close()
    conn.close()
    
    return jsonify(response)

@app.route('/stats', methods=['GET'])
def stats():
    """Statistiques collection"""
    collection_info = qdrant_client.get_collection(COLLECTION_NAME)
    
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute("SELECT COUNT(*) as total_chansons FROM chansons")
    total = cur.fetchone()[0]
    cur.close()
    conn.close()
    
    return jsonify({
        "collection": COLLECTION_NAME,
        "vector_count": collection_info.points_count,
        "chansons_metadata": total
    })

if __name__ == '__main__':
    init_qdrant()
    init_postgres()
    app.run(host='0.0.0.0', port=5000, debug=True)
```

### 2. UI Streamlit - ui/streamlit_app.py

```python
import streamlit as st
import requests
import numpy as np
import pandas as pd

API_URL = "http://api:5000"

st.set_page_config(page_title="Vector DB Search", layout="wide")

st.title("🎵 Vector Database Search")
st.markdown("Recherche musicale par similarité vectorielle")

# ===== SIDEBAR CONFIG =====
st.sidebar.header("Configuration")
top_k = st.sidebar.slider("Top K résultats", 1, 20, 10)
collection_name = st.sidebar.selectbox("Collection", ["spotify"])

# ===== MAIN TABS =====
tab1, tab2, tab3 = st.tabs(["🔍 Rechercher", "📤 Upload", "📊 Stats"])

# ===== TAB 1: RECHERCHER =====
with tab1:
    st.header("Rechercher chansons similaires")
    
    # Deux colonnes
    col1, col2 = st.columns([1, 2])
    
    with col1:
        query_type = st.radio("Type de requête", ["Random", "Exemple"])
        
        if query_type == "Random":
            query_vector = np.random.randn(128).tolist()
            st.info("Requête aléatoire générée")
        else:
            st.info("Requête pré-définie")
            query_vector = np.random.randn(128).tolist()
    
    with col2:
        if st.button("🔍 Rechercher", key="search_btn"):
            with st.spinner("Recherche en cours..."):
                try:
                    response = requests.post(
                        f"{API_URL}/search",
                        json={
                            "query_vector": query_vector,
                            "top_k": top_k
                        }
                    )
                    
                    if response.status_code == 200:
                        results = response.json()
                        
                        st.success(f"✓ {len(results)} résultats trouvés")
                        
                        # Tableau résultats
                        df = pd.DataFrame([
                            {
                                "Rang": i+1,
                                "Titre": r["titre"],
                                "Artiste": r["artiste"],
                                "Similarité": f"{r['similarity']:.3f}"
                            }
                            for i, r in enumerate(results)
                        ])
                        
                        st.dataframe(df, use_container_width=True)
                        
                        # Graphique similarité
                        scores = [r["similarity"] for r in results]
                        st.bar_chart(pd.DataFrame({
                            "Rang": range(1, len(scores)+1),
                            "Score": scores
                        }).set_index("Rang"))
                    else:
                        st.error(f"Erreur API: {response.status_code}")
                
                except Exception as e:
                    st.error(f"Erreur: {str(e)}")

# ===== TAB 2: UPLOAD =====
with tab2:
    st.header("Upload chansons")
    
    uploaded_file = st.file_uploader("Choisir fichier JSON", type=["json"])
    
    if uploaded_file is not None:
        import json
        data = json.load(uploaded_file)
        
        st.write(f"Fichier contient {len(data['chansons'])} chansons")
        
        if st.button("📤 Upload"):
            with st.spinner("Upload en cours..."):
                try:
                    response = requests.post(
                        f"{API_URL}/upload",
                        json=data
                    )
                    
                    if response.status_code == 200:
                        result = response.json()
                        st.success(f"✓ {result['uploaded']} chansons uploadées")
                    else:
                        st.error(f"Erreur: {response.status_code}")
                
                except Exception as e:
                    st.error(f"Erreur: {str(e)}")

# ===== TAB 3: STATS =====
with tab3:
    st.header("Statistiques")
    
    try:
        response = requests.get(f"{API_URL}/stats")
        
        if response.status_code == 200:
            stats = response.json()
            
            col1, col2 = st.columns(2)
            
            with col1:
                st.metric("Vecteurs en index", stats.get("vector_count", 0))
            
            with col2:
                st.metric("Chansons metadata", stats.get("chansons_metadata", 0))
            
            st.info(f"Collection: {stats.get('collection', 'N/A')}")
    
    except Exception as e:
        st.error(f"Erreur: {str(e)}")
```

### 3. Dockerfile pour API

```dockerfile
# api/Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

```
# api/requirements.txt
Flask==2.3.0
qdrant-client==2.4.0
psycopg2-binary==2.9.6
numpy==1.24.0
```

### 4. Dockerfile pour UI

```dockerfile
# ui/Dockerfile
FROM python:3.10-slim

WORKDIR /app

RUN pip install --no-cache-dir streamlit requests pandas numpy

COPY streamlit_app.py .

EXPOSE 8501

CMD ["streamlit", "run", "streamlit_app.py"]
```

---

## TESTS ET VALIDATION

### 1. Test API directe (curl)

```bash
# Health check
curl http://localhost:5000/health

# Stats
curl http://localhost:5000/stats

# Upload test
curl -X POST http://localhost:5000/upload \
  -H "Content-Type: application/json" \
  -d '{
    "chansons": [
      {"titre": "Shape of You", "artiste": "Ed Sheeran", "embedding": [0.1, 0.2, ...]},
      {"titre": "Thinking Out Loud", "artiste": "Ed Sheeran", "embedding": [0.11, 0.21, ...]}
    ]
  }'

# Search
curl -X POST http://localhost:5000/search \
  -H "Content-Type: application/json" \
  -d '{"query_vector": [0.15, 0.25, ...], "top_k": 5}'
```

### 2. Test Python script

```python
# test_full_stack.py
import requests
import numpy as np

API = "http://localhost:5000"

# 1. Health
print("1️⃣  Health check...")
resp = requests.get(f"{API}/health")
print(f"   Status: {resp.json()['status']}")

# 2. Upload
print("\n2️⃣  Upload chansons...")
chansons = [
    {"titre": "Shape of You", "artiste": "Ed Sheeran", "embedding": np.random.randn(128).tolist()},
    {"titre": "Thinking Out Loud", "artiste": "Ed Sheeran", "embedding": np.random.randn(128).tolist()},
]
resp = requests.post(f"{API}/upload", json={"chansons": chansons})
print(f"   Uploaded: {resp.json()['uploaded']}")

# 3. Search
print("\n3️⃣  Rechercher...")
query = np.random.randn(128).tolist()
resp = requests.post(f"{API}/search", json={"query_vector": query, "top_k": 2})
for result in resp.json():
    print(f"   {result['titre']} ({result['similarity']:.3f})")

# 4. Stats
print("\n4️⃣  Stats...")
resp = requests.get(f"{API}/stats")
print(f"   Vecteurs: {resp.json()['vector_count']}")
```

---

## DÉPANNAGE

### Problème: "Connection refused"

```bash
# Vérifier services lancés
docker-compose ps

# Vérifier logs
docker-compose logs qdrant
docker-compose logs api
```

### Problème: "Database error"

```bash
# Vérifier PostgreSQL
docker-compose exec postgres psql -U vectordb_user -d vectordb -c "\dt"

# Reinitialiser
docker-compose down -v
docker-compose up -d
```

### Problème: API not responding

```bash
# Redémarrer API
docker-compose restart api

# Vérifier endpoint
curl -v http://localhost:5000/health
```

---

## ✅ CHECKLIST DÉPLOIEMENT

- [ ] Docker installé
- [ ] Docker Compose lancé
- [ ] Qdrant healthy (curl http://localhost:6333/api/health)
- [ ] PostgreSQL healthy
- [ ] API répond (curl http://localhost:5000/health)
- [ ] UI accessible (http://localhost:8501)
- [ ] Données uploadées
- [ ] Recherche fonctionne

---

**Vous avez une architecture locale complète et prête ! 🚀**
