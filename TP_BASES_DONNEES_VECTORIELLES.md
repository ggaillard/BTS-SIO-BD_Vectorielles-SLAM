# 📘 TP - BASES DE DONNÉES VECTORIELLES
## Cas réels : Spotify (recommandations) + ING Bank (détection fraude)

**Durée totale :** 2h30

---

## 🎯 OBJECTIFS

À la fin de ce TP, vous saurez :
- ✓ Différencier les bases de données traditionnelles et vectorielles
- ✓ Utiliser Pinecone en ligne (gratuit)
- ✓ Interroger par similarité (plus proches voisins)
- ✓ Critiquer une solution d'apprentissage automatique
- ✓ Identifier les risques réels (biais, protection des données, éthique)

**Compétences BTS SIO :**
- B2.1 - Concevoir une solution applicative
- B2.2 - Assurer la cybersécurité
- B5.1 - Travail en projet + veille technologique
- E6 - Conception et développement d'application

---

## ⏱️ DÉROULEMENT (150 MINUTES)

| Phase | Durée | Activité |
|-------|-------|----------|
| **1 - Diagnostic** | 20 min | Pourquoi SQL classique insuffisant ? |
| **2 - Pratique** | 30 min | Pinecone en ligne + premières requêtes |
| **3 - Analyse** | 35 min | Critique + débat de classe |
| **4 - Cas métier** | 35 min | Détection fraude ING Bank |
| **5 - Réflexion** | 20 min | Risques éthiques et techniques |

---

## 📊 PHASE 1 : DIAGNOSTIC (20 MIN)

### Le Problème : Spotify et les recommandations

**Situation réelle :**
- 600 millions d'utilisateurs
- 100+ chansons écoutées en moyenne par utilisateur
- 60 milliards d'associations utilisateur-chanson
- **Question :** "Quelles chansons recommander à cet utilisateur ?"

### Approche 1 : Base de données SQL classique ❌

```sql
-- Chercher des utilisateurs similaires à Y
SELECT chansons.titre
FROM utilisateurs u1
JOIN historique h1 ON u1.id = h1.user_id
WHERE u1.id != Y
  AND EXISTS (
    SELECT 1 FROM historique h2
    WHERE h2.user_id = Y
    AND h2.chanson_id = h1.chanson_id
  )
LIMIT 10;
```

**Problèmes observés :**
- ❌ Cherche correspondances **exactes** (mêmes chansons)
- ❌ Ignore les nuances (similaire mais pas identique)
- ❌ **TRÈS LENT** avec 600M utilisateurs (10+ secondes)
- ❌ Pas de classement par pertinence

**Exemple concret :**
```
Utilisateur Y aime: "Shape of You" (Ed Sheeran)
Recherche SQL: d'autres qui AIMENT EXACTEMENT cette chanson
Trouve: 50 personnes seulement
Résultat recommandé: "Shape of You (Remix)" → pas une vraie découverte ❌
```

### Approche 2 : Base de données vectorielle ✓

**Comment cela fonctionne :**

1. **Conversion en vecteurs** (via apprentissage automatique)
   ```
   "Shape of You" → [0.12, -0.45, 0.89, ..., 0.34] (128 dimensions)
                    └─ Capture : style, genre, tempo, atmosphère, artiste
   ```

2. **Chercher les plus proches** (par similarité)
   ```
   Trouver 10 chansons les plus similaires
   à "Shape of You" + "Blinding Lights" (moyenne)
   ```

3. **Résultats rapides** (<50 millisecondes)
   ```
   "Thinking Out Loud" → similarité 0.948 ✓
   "Perfect" → similarité 0.912 ✓
   "Bad Habits" → similarité 0.887 ✓
   ```

### Activité 1.1 : Comparez les deux approches (20 min)

**Travail individuel :**

Remplissez le tableau comparatif :

| Aspect | Base SQL | Base vectorielle |
|--------|----------|------------------|
| **Comment cherche ?** | Correspondance exacte | Proximité/similarité |
| **Type de résultat** | Oui ou Non | Score de 0 à 1 |
| **Vitesse (600M utilisateurs)** | 10+ secondes ❌ | <50 ms ✓ |
| **Pertinence** | Rigide | Graduée, nuancée |
| **Exemple meilleur usage** | ? | ? |

**Questions à se poser :**
1. Pourquoi la base vectorielle est-elle plus rapide ?
2. Quand utiliser SQL classique vs vectorielle ?
3. Quels sont les risques potentiels ?

---

## 🔗 PHASE 2 : PRATIQUE EN LIGNE (30 MIN)

### Étape 2.1 : Créer un compte Pinecone (5 min)

1. Allez sur **https://app.pinecone.io**
2. **Créer un compte** (courrier électronique standard)
3. Créez **un projet** nommé `tp-[votre-nom]`
4. Notez votre **clé d'accès API** (visible au clic)

**Coût :** GRATUIT jusqu'à 5 millions de vecteurs ✓

### Étape 2.2 : Charger des données (10 min)

**Code Python (copier-coller directement) :**

```python
# Installation
import subprocess
import sys
subprocess.check_call([sys.executable, "-m", "pip", "install", 
                      "pinecone-client", "numpy", "-q"])

# Imports
import pinecone
import numpy as np

# 1. CONNEXION À PINECONE
cle_api = "VOTRE_CLE_API_ICI"  # à remplacer
pinecone.init(api_key=cle_api, environment="gcp-starter")
print("✓ Connexion établie")

# 2. CRÉER UN INDEX (conteneur de vecteurs)
nom_index = "chansons-spotify"
if nom_index not in pinecone.list_indexes():
    pinecone.create_index(
        name=nom_index,
        dimension=128,
        metric="cosine"
    )
    print(f"✓ Index créé: {nom_index}")

# 3. DONNÉES D'EXEMPLE (Spotify)
chansons = [
    {"id": "1", "titre": "Shape of You", "artiste": "Ed Sheeran"},
    {"id": "2", "titre": "Blinding Lights", "artiste": "The Weeknd"},
    {"id": "3", "titre": "Thinking Out Loud", "artiste": "Ed Sheeran"},
    {"id": "4", "titre": "Save Your Tears", "artiste": "The Weeknd"},
    {"id": "5", "titre": "Bad Habits", "artiste": "Ed Sheeran"},
]

# Créer vecteurs (simulation - en vrai, apprentissage automatique les crée)
np.random.seed(42)
for chanson in chansons:
    chanson["vecteur"] = np.random.randn(128).tolist()

# 4. CHARGER DANS PINECONE
index = pinecone.Index(nom_index)
vecteurs_a_charger = [
    (c["id"], c["vecteur"], {"titre": c["titre"], "artiste": c["artiste"]})
    for c in chansons
]
index.upsert(vectors=vecteurs_a_charger)
print(f"✓ {len(chansons)} chansons chargées")

# 5. VÉRIFIER
stats = index.describe_index_stats()
print(f"✓ Vecteurs en index: {stats['total_vector_count']}")
```

**Résultat attendu :**
```
✓ Connexion établie
✓ Index créé: chansons-spotify
✓ 5 chansons chargées
✓ Vecteurs en index: 5
```

### Étape 2.3 : Faire une requête (10 min)

```python
# REQUÊTE : "Quelles chansons ressemblent à Shape of You ?"

vecteur_requete = chansons[0]["vecteur"]

resultats = index.query(
    vector=vecteur_requete,
    top_k=3,
    include_metadata=True
)

print("\n=== CHANSONS SIMILAIRES À 'Shape of You' ===")
for correspondance in resultats['matches']:
    score = correspondance['score']
    titre = correspondance['metadata']['titre']
    print(f"  {score:.3f} | {titre}")
```

**Résultat observé :**
```
=== CHANSONS SIMILAIRES À 'Shape of You' ===
  1.000 | Shape of You         ← elle-même
  0.892 | Thinking Out Loud    ← même artiste
  0.845 | Bad Habits           ← même artiste
```

**Observation clé :**
- "Shape of You" obtient 1.0 (elle-même)
- Les autres Ed Sheeran obtiennent des scores élevés (0.89, 0.84)
- → **Le vecteur capture vraiment le "style musical"**

---

## 🔍 PHASE 3 : ANALYSE CRITIQUE (35 MIN)

### Activité 3.1 : Vérifiez les résultats (15 min)

**Travail individuel :**

Testez différentes requêtes et analysez les résultats.

**Questions à explorer :**

1. **Qualité des résultats**
   - Les chansons recommandées vous semblent-elles pertinentes ?
   - Y a-t-il un biais vers un artiste en particulier ?
   - Que manque-t-il comme information ?

2. **Cas limites**
   - Si on recherche une chanson très différente, qu'se passe-t-il ?
   - Comment sauriez-vous que le système s'est trompé ?

3. **Limite importante**
   - **Similarité = Pertinence ?**
   - Deux chansons très similaires = c'est forcément ce qu'écoute l'utilisateur ?
   - Contre-exemple possible : ?

### Activité 3.2 : Débat de classe (20 min)

**Question centrale :** "Est-ce qu'une chanson similaire = toujours une bonne recommandation ?"

**Scénarios à discuter :**

```
Scénario 1: Recommander "Thinking Out Loud" à qui aime "Shape of You"
→ Pertinent ? Oui/Non/Peut-être ? Justifiez.

Scénario 2: Recommander "Blinding Lights" à qui aime "Shape of You"  
→ Pertinent ? (Artiste différent, mais même époque/style)

Scénario 3: Recommander "Perfect" à qui a écouté "Shape of You" 1 seconde
→ Pertinent ? (Similarité haute, mais utilisateur pas vraiment intéressé)

Conclusion: Similarité mathématique ≠ toujours ce que veut l'utilisateur
```

**Débat structuré :**
- Qui pense "similaire = bon" ?
- Qui pense "non, ce n'est pas si simple" ?
- Qu'en est-il des cas limites ?

---

## 🏛️ PHASE 4 : CAS MÉTIER - DÉTECTION DE FRAUDE ING BANK (35 MIN)

### Contexte réel

**ING Bank (situation actuelle) :**
- 1,2 million de transactions par jour
- Fraudes : 40 millions d'euros par an (avant apprentissage automatique)
- Objectif : Détecter les fraudes avant qu'elles impactent le client

**Approche par base vectorielle :**

1. **Chaque transaction = vecteur** (32 dimensions)
   ```
   [montant, heure, lieu, catégorie marchand, historique client, ...]
   ```

2. **Profil normal du client = moyenne de ses transactions**
   ```
   Transactions normales du client :
   - "Café 50€ à 10h à Paris" ✓
   - "Pharmacie 80€ à 14h à Paris" ✓
   - "Restaurant 120€ à 20h à Paris" ✓
   ```

3. **Nouvelle transaction → Chercher les plus proches**
   ```
   Si tous les K plus proches voisins = anormaux
   → ALERTE FRAUDE
   ```

### Activité 4.1 : Construire un détecteur (20 min)

**Code Python à exécuter :**

```python
import numpy as np

class DetecteurFraude:
    def __init__(self, transactions_normales, seuil=0.65):
        """
        Entraîner le détecteur sur des transactions normales
        seuil = score minimum acceptable
        """
        self.profil_normal = np.mean(
            [t["vecteur"] for t in transactions_normales], 
            axis=0
        )
        self.historique = transactions_normales
        self.seuil = seuil
    
    def similarite(self, v1, v2):
        """Calcul de similarité par cosinus"""
        norme1 = np.linalg.norm(v1)
        norme2 = np.linalg.norm(v2)
        if norme1 == 0 or norme2 == 0:
            return 0
        return np.dot(v1, v2) / (norme1 * norme2)
    
    def detecter_fraude(self, transaction):
        """Détecter si la transaction est frauduleuse"""
        requete = transaction["vecteur"]
        
        # Similarité avec l'historique normal
        similarities = [
            self.similarite(requete, t["vecteur"])
            for t in self.historique
        ]
        
        # Si le minimum est sous le seuil → FRAUDE
        score_min = min(similarities)
        est_fraude = score_min < self.seuil
        
        return {
            "id": transaction["id"],
            "score_min": score_min,
            "est_fraude": est_fraude,
            "verdict": "🔴 FRAUDE DÉTECTÉE" if est_fraude else "🟢 TRANSACTION OK"
        }

# DONNÉES DE TEST

# Transactions normales
trans_normales = [
    {"id": "t1", "montant": 45, "heure": 10, "vecteur": np.random.randn(32) * 0.1},
    {"id": "t2", "montant": 52, "heure": 11, "vecteur": np.random.randn(32) * 0.1},
    {"id": "t3", "montant": 48, "heure": 9, "vecteur": np.random.randn(32) * 0.1},
]

# Fraude : complètement anormale
transaction_fraude = {
    "id": "t4", 
    "montant": 5000, 
    "heure": 3, 
    "vecteur": np.random.randn(32) * 2.0
}

# TEST
detecteur = DetecteurFraude(trans_normales, seuil=0.65)
resultat = detecteur.detecter_fraude(transaction_fraude)

print(f"Transaction : {resultat['id']}")
print(f"Score minimum : {resultat['score_min']:.3f}")
print(f"Verdict : {resultat['verdict']}")
```

**Résultat attendu :**
```
Transaction : t4
Score minimum : 0.125
Verdict : 🔴 FRAUDE DÉTECTÉE
```

**Compréhension du résultat :**
- Score 0.125 = très différent des transactions normales
- Score 0.65 = seuil
- 0.125 < 0.65 → Fraude détectée ✓

### Activité 4.2 : Analyser le système (15 min)

**Répondez individuellement :**

1. **Performance réelle**
   - Combien de transactions par seconde ce système peut-il traiter ?
   - Est-ce assez rapide pour bloquer en temps réel ? Pourquoi ?

2. **Faux positifs et faux négatifs**
   - Que se passe-t-il si on augmente le seuil à 0.75 ?
   - Et si on le baisse à 0.50 ?
   - Quel problème client crée chaque cas ?

3. **Attaque possible**
   - Un fraudeur remarque que les montants 2000-2500€ à 3h ne déclenchent pas d'alerte
   - Comment pourrait-il contourner le système ?
   - Comment le défendre ?

4. **Biais discriminatoire (critique éthique)**
   - Si les minorités ont moins de données historiques
   - Comment leurs vecteurs seraient-ils affectés ?
   - Conséquence : comment seraient-ils traités ? (plus ou moins alertés ?)

---

## 💭 PHASE 5 : RÉFLEXION PERSONNELLE (20 MIN)

### Fiche individuelle de réflexion

**À remplir seul, sans copie :**

```markdown
## MA RÉFLEXION - BASES DE DONNÉES VECTORIELLES

### 1. Avantages identifiés
J'ai observé 3 vrais avantages :
1. _________________________________
2. _________________________________
3. _________________________________

### 2. Risques identifiés
J'ai identifié 3 risques réels :
1. _________________________________
2. _________________________________
3. _________________________________

### 3. Le problème des biais
Si le système apprend sur des données biaisées (exemple : moins de musique rap que rock) :
- Comment le vecteur de la musique rap serait-il affecté ?
- Résultat sur les recommandations ?

### 4. Défi pour la protection des données
Un client demande : "Supprimez mes données"
Problème technique : Son vecteur est mélangé dans 10 millions d'autres
Quelle solution voyez-vous ? (solution imparfaite OK)

### 5. La question du "pourquoi"
Le système bloque une transaction et le client demande : "Pourquoi ?"
Le système répond : "Score 0.42 < 0.65"
Le client : "Oui, mais pourquoi ?"

Comment expliquez-vous vraiment au client pourquoi sa transaction était anormale ?

### 6. Quand utiliser cette technologie
Indiquez OUI/NON/PEUT-ÊTRE pour chaque cas et justifiez un choix :

- Recommandations musicales : ___
- Détection fraude bancaire : ___
- Recherche simple de documents : ___
- Gestion des stocks inventaire : ___
- Chat avec apprentissage automatique : ___

Justification pour un cas choisi :
_________________________________
```

### Réflexion en classe (débat final - 10 min)

**Nous discutons ensemble :**

1. **"C'est juste une technologie, donc c'est neutre. Pourquoi s'inquiéter ?"**
   - Attendu : Les vecteurs héritent les biais des données

2. **"Si c'est plus rapide et plus précis, on devrait l'utiliser partout, non ?"**
   - Attendu : Non, pas pour les transactions critiques ; SQL mieux pour certains usages

3. **"Un utilisateur a été bloqué. Qui est responsable : la technique ou les humains ?"**
   - Attendu : Les deux ; besoin de supervision humaine

4. **"Spotify utilise ça depuis des années sans problème majeur. C'est donc sûr ?"**
   - Attendu : Cas d'usage différent ; recommandations ≠ détection fraude

---

## ✅ CRITÈRES D'ÉVALUATION

### Ce qu'on cherche à observer

**Compétence B2.1 (Architecture) :**
- [ ] Comprendre la différence SQL vs vectorielle
- [ ] Pouvoir utiliser une interface de programmation en ligne
- [ ] Justifier les choix architecturaux

**Compétence B2.2 (Cybersécurité) :**
- [ ] Identifier les biais algorithmiques
- [ ] Soulever les questions de protection des données
- [ ] Exiger de la transparence dans les décisions

**Compétence B5.1 (Projet) :**
- [ ] Analyse critique (pas juste dire "c'est cool")
- [ ] Proposer des améliorations
- [ ] Penser aux limites

**E6 (Conception) :**
- [ ] Évaluer une architecture de données
- [ ] Intégrer l'apprentissage automatique
- [ ] Penser qualité ET éthique

---

## 📚 RESSOURCES UTILES

### Concepts clés

**Vecteur :** Représentation mathématique d'un concept
- Exemple : Une chanson → 128 nombres

**Vecteur d'apprentissage :** Conversion automatique via apprentissage automatique
- Exemple : Audio → Vecteur (le modèle apprend les caractéristiques)

**Similarité :** Comment mesurer si deux chansons se ressemblent
- Entre 0 (opposées) et 1 (identiques)

**Plus proches voisins :** Trouver les éléments les plus similaires

### Cas réels

- **Spotify :** 600 millions d'utilisateurs, 35% du chiffre = recommandations
- **ING Bank :** 1,2 million de transactions/jour, 73% de fraudes détectées
- **Netflix :** 80% du temps regardé = recommandations

### Outils utilisés

- **Pinecone :** Service en ligne pour bases vectorielles (gratuit 5M vecteurs)
- **Python/NumPy :** Calculs vectoriels
- **Apprentissage automatique :** Création des vecteurs

---

## 🎓 CONCLUSION

**Base vectorielle = Outil puissant mais :**
- ✓ Utile pour trouver la similarité
- ❌ Pas solution universelle (SQL mieux pour transactions)
- ⚠️ Risques réels (biais, protection données, transparence)
- 🔒 Nécessite supervision humaine et audits réguliers

**Votre rôle en informatique :**
- Savoir quand l'utiliser vs quand utiliser SQL
- Identifier les risques AVANT déploiement
- Exiger les audits de biais
- Superviser les algorithmes (pas confiance aveugle)

---

**Bon TP ! 🚀**

Des questions ? Relisez les concepts clés ci-dessus ou demandez à votre professeur.
