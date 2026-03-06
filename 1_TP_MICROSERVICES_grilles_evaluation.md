# 📊 GRILLES D'ÉVALUATION - TP MICROSERVICES

## GRILLE 1 : ÉVALUATION ÉTUDIANT (Notation /25)

### Compétence B2.1 - Conception solution applicative

| Indicateur | Niveau 1 (1-2 pts) | Niveau 2 (2-3 pts) | Niveau 3 (3-4 pts) | Niveau 4 (4-5 pts) | Score |
|------------|-------------------|-------------------|-------------------|-------------------|-------|
| **Décomposition en services** | Services mal séparés, couplage visible | Services définis mais dépendances peu claires | Services bien séparés, dépendances claires | Services optimaux, Conway's Law appliquée | ___ / 5 |
| **Architecture distribuée** | Ignoré ou mal compris | Partiellement compris | Architecture cohérente | Architecture réfléchie avec alternatives | ___ / 5 |

### Compétence B2.2 - Cybersécurité & RGPD

| Indicateur | Niveau 1 (1 pt) | Niveau 2 (2 pts) | Niveau 3 (3 pts) | Score |
|------------|-----------------|-----------------|-----------------|-------|
| **Isolation données & authentification** | Non adressée | Mentionnée superficiellement | Plan clair avec mTLS/API keys | ___ / 3 |
| **Droit à l'oubli distribué** | Ignoré | Reconnu comme problème | Solution proposée | ___ / 2 |

### Compétence B5.1 - Projet & Management

| Indicateur | Score |
|------------|-------|
| **Timeline migration réaliste** | ___ / 4 |
| **Gestion des dépendances inter-équipes** | ___ / 3 |
| **Identification des risques réels** | ___ / 3 |

### Transversal - Communication

| Indicateur | Score |
|------------|-------|
| **Note DSI professionnelle & claire** | ___ / 4 |
| **Pitch convainquant (90 sec)** | ___ / 2 |

---

### **TOTAL ÉTUDIANT: _____ / 25**

---

## GRILLE 2 : AUTO-ÉVALUATION RED TEAM

**Utilisée après la phase 2 (critique IA)**

Pour chaque équipe (Technique, Org, Sécu):

### Équipe TECHNIQUE

```
Question: "Avez-vous identifié au moins 1 vrai problème technique?"
❌ Non → Relire la proposition + points clés
✓ Oui → Continuez à Phase 3

Questions auto-contrôle:
- [ ] Décomposition en services logique ?
- [ ] Patterns de communication justifiés (REST vs async) ?
- [ ] Gestion consistance distribuée (Saga) envisagée ?
- [ ] Scalabilité verticale/horizontale ?
```

### Équipe ORGANISATIONNELLE

```
Question: "Avez-vous identifié un enjeu RH réel?"
❌ Non → Pensez: budget, compétences, coordination
✓ Oui → Continuez

Questions auto-contrôle:
- [ ] Nombre équipes nécessaire × salaire estimé ?
- [ ] Formation requise (DevOps, nouvelles stacks) ?
- [ ] Dépendances inter-équipes gérables ?
- [ ] Qui pilote la migration ?
```

### Équipe SÉCURITÉ

```
Question: "Avez-vous identifié 1 risque RGPD ?"
❌ Non → Pensez: droit oubli, iso données, audit
✓ Oui → Continuez

Questions auto-contrôle:
- [ ] Droit à l'oubli gérable en distribué ?
- [ ] Chiffrement données en transit ?
- [ ] Authentification inter-services (mTLS) ?
- [ ] Audit trail centralisé possible ?
```

---

## GRILLE 3 : CRITÈRES POUR ÉVALUER LA PROPOSITION IA

**À complètement en Phase 2 (30 min)**

### Dimension TECHNIQUE (20 pts max)

```
1. Décomposition métier
   □ Service par domaine logique (DDE, Formations, etc.)
   □ Peu de dépendances inter-services
   □ Données clairement séparées
   Score: ___ / 5

2. Communication services
   □ REST pour requêtes urgentes (temps réel)
   □ Async (Kafka) pour notifications
   □ Timeouts & fallbacks en place
   Score: ___ / 5

3. Gestion données distribuées
   □ Saga pattern pour transactions
   □ Consistance eventual acceptable
   □ Duplication données maîtrisée
   Score: ___ / 5

4. Opérabilité
   □ Monitoring distribué (logging centralisé)
   □ Versioning APIs clair
   □ Déploiement indépendant des services
   Score: ___ / 5

TOTAL TECHNIQUE: ___ / 20
```

### Dimension ORGANISATIONNELLE (20 pts max)

```
1. Structure équipes
   □ Équipes autonomes par service (5-8 devs max)
   □ Pas de dépendance artificielle
   □ Responsabilité claire (qui possède quoi)
   Score: ___ / 5

2. Compétences
   □ Stacks technologiques appropriées
   □ Formation DevOps incluse dans planning
   □ Pas de spécialiste unique (bus factor)
   Score: ___ / 5

3. Timeline réaliste
   □ M1-M6 services faciles (peu couplés)
   □ M7-M12 services moyens
   □ M13+ services critiques (Allocations)
   Score: ___ / 5

4. Gouvernance
   □ DSI pilote le projet
   □ Points d'avancement réguliers
   □ Autorisation/refus clairs par phase
   Score: ___ / 5

TOTAL ORGANISATIONNEL: ___ / 20
```

### Dimension SÉCURITÉ/RGPD (20 pts max)

```
1. Isolation données
   □ Chaque service sa propre DB
   □ Pas d'accès direct entre DBs
   □ Données perso chiffrées
   Score: ___ / 5

2. Authentification
   □ mTLS inter-services
   □ API keys + JWT tokens
   □ Révocation possible
   Score: ___ / 5

3. RGPD Compliance
   □ Droit à l'oubli géré (comment ?)
   □ Audit trail par service
   □ Minimisation données
   Score: ___ / 5

4. Continuité activité
   □ Service A down ≠ Tout crash
   □ Fallbacks en place
   □ Backup distribué
   Score: ___ / 5

TOTAL SÉCURITÉ: ___ / 20
```

### Dimension FINANCIÈRE (10 pts max)

```
1. Coût infra actuel vs proposé
   □ Estimation réaliste
   □ Licences logicielles
   □ Personnel DevOps additionnel
   Score: ___ / 5

2. ROI / Justification
   □ Réduction time-to-market quantifiée
   □ Gain stabilité mesurable
   □ Breakeven calculé
   Score: ___ / 5

TOTAL FINANCIER: ___ / 10
```

---

### **TOTAL CRITIQUE: ___ / 70**

**Interprétation:**
- 50-70: Excellente analyse, critique constructive
- 35-49: Bonne analyse, quelques lacunes
- 20-34: Analyse partielle
- <20: Analyse insuffisante

---

## GRILLE 4 : ÉVALUATION COLLECTIVE (FIN TP)

**Remplit par le prof après débat Phase 4**

| Observation | Oui | Partiellement | Non |
|------------|-----|---|---|
| Étudiants comprennent différence monolithe/micro | ☐ | ☐ | ☐ |
| Risques organisationnels identifiés (pas juste tech) | ☐ | ☐ | ☐ |
| Conway's Law compris (structure org = structure code) | ☐ | ☐ | ☐ |
| Dépendances distribuées reconnues comme défi | ☐ | ☐ | ☐ |
| Communication profession (pitch, note DSI) | ☐ | ☐ | ☐ |
| Critique constructive (pas "c'est nul") | ☐ | ☐ | ☐ |

**Observations générales:**
```
Ce qui a bien marché:
_________________________________________________

À améliorer pour prochain TP:
_________________________________________________

Obstacles didactiques observés:
_________________________________________________
```

---

## 📌 NOTES POUR L'ÉVALUATION

- **B2.1:** Évaluez sur décomposition architecture (technique) + cohérence
- **B2.2:** Évaluez sur identification risques sécu + solutions
- **B5.1:** Évaluez sur timeline (réalisme) + gestion dépendances
- **E6:** Évaluez sur conception application (microservices = design pattern)

**Pointage total étudiant:** Moyenne des grilles 1 + observations grille 4

---

**Bon évaluation ! 🎓**
