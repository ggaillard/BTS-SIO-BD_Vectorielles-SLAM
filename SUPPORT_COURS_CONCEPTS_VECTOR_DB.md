# 📖 SUPPORT DE COURS - VECTOR DATABASES
## Concepts clés, illustrations, formules et architecture

---

## 🎯 TABLE DES MATIÈRES

1. [Concepts fondamentaux](#concepts-fondamentaux)
2. [Architecture Vector DB](#architecture-vector-db)
3. [Formules et mathématiques](#formules-et-mathématiques)
4. [Cas pratiques](#cas-pratiques)
5. [Architecture locale](#architecture-locale)
6. [Points clés à retenir](#points-clés-à-retenir)

---

## CONCEPTS FONDAMENTAUX

### 1. VECTEUR

**Définition simple:**
Un vecteur = liste de nombres représentant un concept dans un espace mathématique

**Exemple - Chanson "Shape of You":**
```
Dimension 1 : Énergie (0 = très calme, 1 = très énergique)
Dimension 2 : Mélancolie (0 = joyeux, 1 = triste)
Dimension 3 : Tempo (0 = lent, 1 = rapide)
...
Dimension 128 : [autres caractéristiques]

Résultat: [0.23, 0.45, 0.67, ..., 0.34]
          (128 dimensions)
```

**Visualisation 3D (simplifié):**
```
        Tempo (axe Z)
           ↑
           |     Shape of You
           |    /
           |   /
           |  •
           | /
    ------•----→ Énergie (axe X)
          /|
         / |
        /  | Mélancolie (axe Y)
       /   |
```

**En réalité:** 128 dimensions (impossible à visualiser, mais mathématiquement pareil)

---

### 2. EMBEDDING

**Définition:**
Processus de conversion de données (musique, texte, image) en vecteur via un modèle ML

**Processus:**
```
INPUT: Chanson "Shape of You" (audio MP3, 44.1kHz)
       ↓ [Spectrogramme extraction]
       → Fréquences, rythme, instruments détectés
       ↓ [Deep Learning Model]
       → Apprentissage du "style musical"
       ↓ [Réduction dimensionnelle]
       → Compression 1000s features → 128 dimensions
OUTPUT: Vecteur [0.23, 0.45, 0.67, ..., 0.34]
```

**Propriété clé:**
Chansons **similaires** → **vecteurs proches** dans l'espace

```
"Shape of You" [0.23, 0.45, 0.67, ...]
"Thinking Out Loud" [0.24, 0.46, 0.68, ...] ← très proche
"Bad Habits" [0.25, 0.44, 0.66, ...] ← proche
"Rocket" [0.78, 0.12, 0.21, ...] ← très éloigné
```

---

### 3. SIMILARITÉ (DISTANCE)

**Cosine Similarity (plus utilisée):**

```
Formule: sim(A, B) = (A · B) / (||A|| × ||B||)

Résultat: Entre 0 et 1
  - 1.0 = identiques
  - 0.9 = très similaires
  - 0.5 = moyennement similaires
  - 0.0 = complètement opposés
```

**Exemple concret:**
```
A = "Shape of You" [0.1, 0.2, 0.3]
B = "Thinking Out Loud" [0.11, 0.21, 0.29]

A · B = 0.1×0.11 + 0.2×0.21 + 0.3×0.29 = 0.1319
||A|| = √(0.1² + 0.2² + 0.3²) = 0.374
||B|| = √(0.11² + 0.21² + 0.29²) = 0.376

Similarité = 0.1319 / (0.374 × 0.376) = 0.939

→ Les chansons sont TRÈS similaires (0.939 ≈ 1.0)
```

**Visualisation:**
```
Deux vecteurs proches       Deux vecteurs éloignés
    ↑ B                         ↑ B
    |\ angle petit              | angle grand
    | \                         |      \
    |  \ A                      |       \ A
    |___→                       |________→

Angle petit → similarité haute    Angle grand → similarité basse
```

---

### 4. KNN (K-Nearest Neighbors)

**Définition:**
Chercher les K vecteurs **les plus proches** d'une requête

**Processus:**
```
1. Query Vector (ce qu'on cherche):
   "Chansons similaires à Shape of You"
   → Utiliser embedding de Shape of You

2. Index (où chercher):
   ✓ "Thinking Out Loud" (distance 0.06)
   ✓ "Perfect" (distance 0.08)
   ✓ "Bad Habits" (distance 0.12)
   ✗ "Rocket" (distance 0.87) ← trop loin

3. Retour Top-K (ex: K=3):
   1. Thinking Out Loud (0.939 similarité)
   2. Perfect (0.922)
   3. Bad Habits (0.887)
```

**Algorithme simplifié:**
```
1. Pour CHAQUE vecteur en base:
   - Calculer distance avec query
2. Trier par distance croissante
3. Retourner Top-K

Complexité: O(n) = lent pour n > 1M
Solution: Index spatial (voir Architecture)
```

---

### 5. INDEX SPATIAL

**Problème:** 
Avec 600M utilisateurs × 100 chansons = 60M vecteurs
- Chercher tous = 60M comparaisons = très lent (10+ sec)

**Solution:**
Structurer les vecteurs dans un **index spécialisé** pour KNN rapide

**Types d'index:**

```
┌─ Brute Force
│  - Cherche tous les vecteurs
│  - Exact mais TRÈS lent O(n)
│  - Utilisé si < 10k vecteurs
│
├─ LSH (Locality Sensitive Hashing)
│  - Groupe vecteurs par hash
│  - Cherche seulement groupe pertinent
│  - Rapide mais approximate
│  - 99% exactitude acceptable
│
├─ HNSW (Hierarchical Navigable Small World) ← Pinecone utilise
│  - Graphe hiérarchique
│  - Navigation type "dico"
│  - Very rapide (O(log n))
│  - 100% exact
│
└─ Annoy (utilisé par Spotify)
   - Forest d'arbres aléatoires
   - Rapide et scalable
   - Version open-source
```

**Comparaison:**
```
                Vitesse     Exactitude   Mémoire
Brute Force      ❌ Lent      ✓ Exact    ✓ Peu
LSH             ✓ Rapide     ⚠ 99%      ✓ Peu
HNSW (Pinecone) ✓ Rapide     ✓ Exact    ⚠ Plus
Annoy           ✓ Rapide     ⚠ 98%      ✓ Peu
```

---

## ARCHITECTURE VECTOR DB

### 1. ARCHITECTURE GÉNÉRALE

```
┌─────────────────────────────────────────────────────┐
│                   APPLICATION                        │
│              (Spotify, ING Bank, etc.)               │
└────────────────────┬────────────────────────────────┘
                     │ API REST
                     ↓
┌─────────────────────────────────────────────────────┐
│            VECTOR DB (Pinecone, Weaviate)           │
├─────────────────────────────────────────────────────┤
│                                                       │
│  ┌─ INPUT LAYER                                      │
│  │  ├─ Embeddings (vecteurs)                        │
│  │  └─ Metadata (titre, artiste, etc.)              │
│  │                                                   │
│  ├─ INDEX LAYER (HNSW, LSH, Annoy)                 │
│  │  ├─ Structure spatiale optimisée                 │
│  │  └─ Permet KNN rapide < 100ms                    │
│  │                                                   │
│  ├─ QUERY LAYER                                      │
│  │  ├─ Reçoit query vecteur                         │
│  │  ├─ Navigue index                                │
│  │  └─ Retourne Top-K résultats                     │
│  │                                                   │
│  └─ STORAGE LAYER                                    │
│     ├─ Vecteurs en mémoire/SSD                      │
│     └─ Metadata en base relationelle                │
│                                                       │
└─────────────────────────────────────────────────────┘
```

### 2. FLUX DONNÉES

**Ingestion (Setup):**
```
1. Données brutes (chansons, transactions)
   ↓
2. Feature Engineering (extraire caractéristiques)
   ↓
3. Embedding Model (ML → vecteurs)
   ├─ Pré-entraîné (OpenAI, Hugging Face)
   └─ Custom fine-tuned
   ↓
4. Upload Vector DB
   ├─ Chaque vecteur + metadata
   ├─ Créer index spatial
   └─ Prêt pour requêtes
```

**Requête (Runtime):**
```
1. Utilisateur: "Recommande musique pour moi"
   ↓
2. Récupérer embeddings historique utilisateur
   ↓
3. Calculer moyenne (profil utilisateur)
   ↓
4. Query Vector DB: "Top 10 similaires"
   ↓
5. Retourner résultats < 50ms
   ├─ Vecteurs + scores
   └─ Enrichir avec metadata (titre, artiste)
   ↓
6. Afficher à utilisateur
```

---

## FORMULES ET MATHÉMATIQUES

### 1. COSINE SIMILARITY

```
Formule:        sim(A, B) = (A · B) / (||A|| × ||B||)

Composants:
- A · B = produit scalaire = Σ aᵢ × bᵢ
- ||A|| = norme L2 = √(Σ aᵢ²)
- ||B|| = norme L2 = √(Σ bᵢ²)

Exemple avec 3D (pour visualiser):
A = [1, 2, 3]
B = [1, 2, 2]

A · B = 1×1 + 2×2 + 3×2 = 1 + 4 + 6 = 11
||A|| = √(1² + 2² + 3²) = √14 = 3.74
||B|| = √(1² + 2² + 2²) = √9 = 3

sim(A,B) = 11 / (3.74 × 3) = 11 / 11.22 = 0.98

→ Très similaires !
```

### 2. EUCLIDEAN DISTANCE

```
Formule:        d(A, B) = √(Σ(aᵢ - bᵢ)²)

Plus petit = plus similaire

Exemple 3D:
A = [1, 2, 3]
B = [1, 2, 2]

d(A,B) = √((1-1)² + (2-2)² + (3-2)²)
       = √(0 + 0 + 1)
       = 1

Plus court : A et B SIMILAIRES
Plus long : A et B DIFFÉRENTS
```

### 3. KNN COMPLEXITY

```
Brute Force:
- N vecteurs en base
- 1 requête = N calculs distance
- Complexité: O(N)

Avec Index (HNSW):
- N vecteurs en base
- Index = structure hiérarchique
- 1 requête = naviguer hiérarchie
- Complexité: O(log N)

Exemple:
- 1M vecteurs
  ├─ Brute Force: 1M comparaisons = 1 seconde
  └─ HNSW: log₂(1M) ≈ 20 hops = 0.02 secondes
  
→ 50x plus rapide !
```

---

## CAS PRATIQUES

### 1. SPOTIFY RECOMMANDATIONS

**Données:**
```
600M utilisateurs
100 chansons écoutées en moyenne/utilisateur
= 60M associations utilisateur-chanson
```

**Processus:**
```
1. Embedding chansons (une fois)
   ├─ 50M chansons
   ├─ Model: VAE ou Transformer
   └─ Output: 128D vecteur/chanson

2. Embedding utilisateurs (dynamique)
   ├─ Moyenne embeddings historique
   └─ Profil utilisateur = [moyenne des 100 vecteurs]

3. Query: "Top 10 similaires"
   ├─ Query = profil utilisateur
   ├─ Vector DB: KNN top 10
   └─ Latence: <50ms ✓

4. Résultats
   "Thinking Out Loud" (0.948)
   "Perfect" (0.912)
   ...

5. Feedback loop
   ├─ Utilisateur écoute ou skip
   ├─ Data collectée
   ├─ Model retrain (continu)
   └─ Embeddings s'améliorent
```

**Valeur créée:**
```
Sans IA:
- Utilisateurs écoutent ce qu'ils connaissent déjà
- Engagement moyen

Avec Vector DB:
- Découvertes musicales
- Engagement +80%
- Rétention utilisateurs +25%
- 35% du CA = recommandations ✓
```

### 2. ING BANK DÉTECTION FRAUDE

**Données:**
```
1.2M transactions/jour
= 432M transactions/an
= 10 ans = 4.3B transactions historiques
```

**Processus:**
```
1. Feature Engineering
   Chaque transaction → 32 features:
   ├─ Montant (normalisé)
   ├─ Heure (0-24)
   ├─ Location (lat/lon)
   ├─ Catégorie marchand
   ├─ Historique client (freq, avg, etc.)
   └─ ... (autres)

2. Vecteur transaction
   ├─ Combinaison 32 features
   ├─ Dimension: 32D
   └─ Norme: 1.0 (pour similarité stable)

3. Profil client NORMAL
   ├─ Moyenne 100 transactions précédentes
   ├─ Représente "patron normal" du client
   └─ Mise à jour quotidienne

4. Nouvelle transaction
   ├─ Query Vector DB
   ├─ KNN = 3 (chercher 3 plus proches)
   ├─ Si min(similarité) < 0.65 → FRAUDE
   └─ Latence: <100ms ✓

5. Action
   ├─ OK: Autoriser transaction
   ├─ ALERTE: Demander confirmation
   └─ FRAUDE: Bloquer
```

**Résultats:**
```
Avant (classique):
- 40M€ fraude/an
- 45% détection
- Beaucoup faux positifs

Après (Vector DB):
- 10M€ fraude/an ✓
- 73% détection ✓
- Moins faux positifs ✓
- Gain: 30M€/an + confiance
```

---

## ARCHITECTURE LOCALE

### 1. ARCHITECTURE DE DÉVELOPPEMENT LOCAL

```
┌─────────────────────────────────────────────────────┐
│              VOTRE ORDINATEUR                        │
├─────────────────────────────────────────────────────┤
│                                                       │
│  ┌─────────────────────────────────────────────┐   │
│  │  Python Script / Jupyter Notebook           │   │
│  │  ├─ Charger données (chansons, transactions)│   │
│  │  ├─ Feature engineering                     │   │
│  │  ├─ Appeler embedding API (OpenAI)          │   │
│  │  └─ Uploader dans Chroma/Qdrant            │   │
│  └──────────────┬─────────────────────────────┘   │
│                 │                                   │
│  ┌──────────────▼─────────────────────────────┐   │
│  │  LOCAL VECTOR DB (Chroma ou Qdrant)       │   │
│  │  ├─ Port: localhost:8000                   │   │
│  │  ├─ Stockage: fichiers locaux              │   │
│  │  └─ Index: HNSW en mémoire/disk            │   │
│  └──────────────┬─────────────────────────────┘   │
│                 │ requête KNN                      │
│  ┌──────────────▼─────────────────────────────┐   │
│  │  Query Script                              │   │
│  │  ├─ Créer query vecteur                    │   │
│  │  ├─ Chercher Top-K                         │   │
│  │  └─ Afficher résultats                     │   │
│  └─────────────────────────────────────────────┘   │
│                                                       │
└─────────────────────────────────────────────────────┘
```

### 2. SETUP COMPLET LOCAL (Docker optionnel)

#### Option A: Chroma (plus simple) ⭐ Recommandé

**Installation:**
```bash
# 1. Créer dossier projet
mkdir vector-db-tp
cd vector-db-tp

# 2. Virtual environment
python3 -m venv venv
source venv/bin/activate  # macOS/Linux
# OU
venv\Scripts\activate  # Windows

# 3. Installer dépendances
pip install chromadb numpy pandas openai

# 4. Créer script test
cat > test_chroma.py << 'EOF'
import chromadb
import numpy as np

# Initialiser Chroma (stockage local)
client = chromadb.Client()
collection = client.create_collection(name="chansons")

# Données exemple
chansons = [
    {"id": "1", "titre": "Shape of You", "artiste": "Ed Sheeran"},
    {"id": "2", "titre": "Thinking Out Loud", "artiste": "Ed Sheeran"},
    {"id": "3", "titre": "Perfect", "artiste": "Ed Sheeran"},
]

# Embeddings simulés
np.random.seed(42)
embeddings = np.random.randn(3, 128)

# Ajouter à Chroma
collection.add(
    ids=[c["id"] for c in chansons],
    documents=[c["titre"] for c in chansons],
    metadatas=[{"artiste": c["artiste"]} for c in chansons],
    embeddings=embeddings.tolist()
)

# Requête: similaires à Shape of You
results = collection.query(
    query_embeddings=embeddings[0:1],
    n_results=2
)

print("Similaires à 'Shape of You':")
for i, doc in enumerate(results['documents'][0]):
    print(f"  {doc}")
EOF

# 5. Exécuter
python test_chroma.py
```

**Résultat attendu:**
```
Similaires à 'Shape of You':
  Shape of You
  Thinking Out Loud
```

---

#### Option B: Qdrant (plus puissant)

**Installation Docker:**
```bash
# 1. Installer Docker (https://docker.com)

# 2. Lancer Qdrant
docker run -p 6333:6333 qdrant/qdrant

# 3. Vérifier (navigateur)
http://localhost:6333/api/health
→ {"status":"ok"}

# 4. Script Python
cat > test_qdrant.py << 'EOF'
from qdrant_client import QdrantClient
import numpy as np

# Connexion Qdrant
client = QdrantClient("localhost", port=6333)

# Créer collection
client.recreate_collection(
    collection_name="chansons",
    vectors_config={
        "size": 128,
        "distance": "Cosine",
    }
)

# Données + embeddings
data = [
    {"id": 1, "titre": "Shape of You", "embedding": np.random.randn(128)},
    {"id": 2, "titre": "Thinking Out Loud", "embedding": np.random.randn(128)},
]

# Uploader
client.upsert(
    collection_name="chansons",
    points=[
        {"id": d["id"], "vector": d["embedding"].tolist(), 
         "payload": {"titre": d["titre"]}}
        for d in data
    ]
)

# Requête
results = client.search(
    collection_name="chansons",
    query_vector=data[0]["embedding"].tolist(),
    limit=2
)

print("Top 2 similaires:")
for hit in results:
    print(f"  {hit.payload['titre']} (score: {hit.score:.3f})")
EOF

python test_qdrant.py
```

---

#### Option C: Weaviate (complet)

**Installation Docker Compose:**
```bash
# 1. Créer docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.4'
services:
  weaviate:
    image: semitechnologies/weaviate:latest
    ports:
      - "8080:8080"
    environment:
      QUERY_DEFAULTS_LIMIT: 25
      AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED: 'true'
      PERSISTENCE_DATA_PATH: /var/lib/weaviate
      DEFAULT_VECTORIZER_MODULE: none
EOF

# 2. Lancer
docker-compose up -d

# 3. Vérifier (navigateur)
http://localhost:8080/v1/.well-known/ready
→ {"status":"ok"}

# 4. Script Python
pip install weaviate-client
```

**Script exemple:**
```python
import weaviate
import numpy as np

client = weaviate.Client("http://localhost:8080")

# Créer classe
client.schema.create_class({
    "class": "Chanson",
    "properties": [
        {"name": "titre", "dataType": ["text"]},
        {"name": "artiste", "dataType": ["text"]},
    ]
})

# Ajouter objet
client.data_object.create({
    "class": "Chanson",
    "properties": {
        "titre": "Shape of You",
        "artiste": "Ed Sheeran",
    },
    "vector": np.random.randn(128).tolist()
})

# Requête KNN
results = client.query.get("Chanson", ["titre", "artiste"]).with_near_vector({
    "vector": np.random.randn(128).tolist()
}).with_limit(5).do()

print(results)
```

---

### 3. ARCHITECTURE PRODUCTION-LIKE LOCAL

```
┌─────────────────────────────────────────────────────┐
│             Docker Compose Local                    │
├─────────────────────────────────────────────────────┤
│                                                       │
│  ┌─────────────────────────────────────────────┐   │
│  │  Container 1: Vector DB (Qdrant)            │   │
│  │  - Port: 6333                                │   │
│  │  - Volume: /data (persistance)              │   │
│  └─────────────────────────────────────────────┘   │
│                                                       │
│  ┌─────────────────────────────────────────────┐   │
│  │  Container 2: Python API (Flask)            │   │
│  │  - Port: 5000                                │   │
│  │  - Parle à Qdrant                           │   │
│  └─────────────────────────────────────────────┘   │
│                                                       │
│  ┌─────────────────────────────────────────────┐   │
│  │  Container 3: UI (Streamlit)               │   │
│  │  - Port: 8501                                │   │
│  │  - Parle à Flask API                        │   │
│  └─────────────────────────────────────────────┘   │
│                                                       │
│  ┌─────────────────────────────────────────────┐   │
│  │  Container 4: PostgreSQL (metadata)         │   │
│  │  - Port: 5432                                │   │
│  │  - Stocke: titre, artiste, description     │   │
│  └─────────────────────────────────────────────┘   │
│                                                       │
└─────────────────────────────────────────────────────┘
```

**docker-compose.yml complet:**
```yaml
version: '3.8'

services:
  # Vector Database
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage
    environment:
      - QDRANT_API_KEY=your-secret-key

  # Python API
  api:
    build: ./api
    ports:
      - "5000:5000"
    depends_on:
      - qdrant
      - postgres
    environment:
      - QDRANT_URL=http://qdrant:6333
      - DATABASE_URL=postgresql://user:pass@postgres:5432/vectordb

  # Web UI
  ui:
    build: ./ui
    ports:
      - "8501:8501"
    depends_on:
      - api

  # Database (metadata)
  postgres:
    image: postgres:15
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=vectordb
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  qdrant_data:
  postgres_data:
```

---

## POINTS CLÉS À RETENIR

### 1. CONCEPTS ESSENTIELS

| Concept | Définition | Analogie |
|---------|-----------|----------|
| **Vecteur** | Liste nombres (128D) | Coordonnées dans espace |
| **Embedding** | Conversion données→vecteur | Traduction langage naturel |
| **Similarité** | Proximité 2 vecteurs (0-1) | Distance entre points |
| **KNN** | Chercher K plus proches | "Qui habite près de moi ?" |
| **Index** | Structure pour KNN rapide | Table des matières d'un livre |

### 2. CAS D'USAGE APPROPRIÉS

```
✓ UTILISER Vector DB si:
├─ Besoin similarité nuancée
├─ Recherche sémantique
├─ Recommandations personnalisées
├─ Détection anomalies/fraude
└─ Clustering basé pattern

✗ N'UTILISER PAS si:
├─ Besoin exact-match (SQL)
├─ Transactions ACID critiques
├─ Données petit volume (<1M)
├─ Pas de modèle ML entraîné
└─ RGPD strict (droit oubli)
```

### 3. ARCHITECTURE CHOIX

```
CLOUD (Pinecone, Weaviate):
✓ Pas d'installation
✓ Auto-scalable
✓ Gestion infrastructure fournie
❌ Coût par requête
❌ Données chez tiers

LOCAL (Chroma, Qdrant):
✓ Contrôle total
✓ Données locales
✓ Gratuit
❌ Vous gérez infrastructure
❌ Scaling manuel

HYBRID:
✓ Prod: Cloud (Pinecone)
✓ Dev/Test: Local (Qdrant)
→ Meilleur des 2 mondes
```

### 4. RISQUES À BIEN COMPRENDRE

```
🔴 BIAIS
  - Vecteurs héritent biais données
  - Minorities recommandées moins souvent
  - Mitigation: audit fairness

🔴 RGPD
  - Droit oubli impossible isoler
  - Retraining complet requis (coûteux)
  - Mitigation: anonymization, data minimization

🔴 OPACITÉ
  - Pourquoi cette recommandation ?
  - Black-box: KNN pas explicable
  - Mitigation: LIME/SHAP, logs détaillés

🔴 FEEDBACK LOOPS
  - Recommandation affecte utilisateur
  - Utilisateur génère données
  - Données réentraînent modèle
  - Peut amplifier biais
  - Mitigation: diversification intentionnelle
```

---

**Vous avez maintenant tous les concepts pour utiliser Vector DB ! 🚀**

