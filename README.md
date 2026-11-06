# ⚖️ Legal AI — RAG Avancé pour l'Expansion Sémantique des Documents Juridiques

> **Projet de Fin d'Études (PFE)**  
> Réalisé par **Assia Bouamir** — Master Intelligence Artificielle  
> Université Cadi Ayyad, Faculté des Sciences Semlalia, Marrakech  
> Encadré par **Dr. Jihad Zahir** (FSSM) & **Dr. Marie Bonnin** (IRD)  
> Entreprise d'accueil : **Institut de Recherche pour le Développement (IRD)**  
> Soutenance : **30/06/2026**

---

## 📖 Description

Ce projet implémente l'architecture de **RAG Avancé Multi-Graph & Multi-Agent** pour résoudre le problème de **Document-Level Retrieval Mismatch (DMR)** dans les documents juridiques africains (droit maritime, législation environnementale).

Il combine :
- **3 graphes de connaissances** : graphe lexical (hiérarchie des lois), graphe de citations (renvois entre articles), graphe des définitions (glossaire juridique).
- **Un moteur de recherche hybride** : BM25 (0.50) + Dense BGE-M3 (0.35) + Graphe (0.15), avec traversée récursive et reranking.
- **Un pipeline multi-agents LangGraph** orchestrant 5 agents spécialisés.
- **Un pipeline d'ingestion autonome** : veille législative, détection de conflits, déduplication temporelle.

> 📄 Ce travail a été présenté à **LaTeLL 2026** (*Language Technologies for Low-resource Languages*, Fès) :  
> **« Beyond RAG: A Multi-Graph, Multi-Agent, Recursive Retrieval Architecture for Traceable and Source-Grounded Legal Answers »**  
> Assia Bouamir, Marie Bonnin, Youssef Al Mouatamid, Jihad Zahir — LISI, Université Cadi Ayyad · IRD / LEMAR, Univ Brest · UMMISCO-IRD

---

## 📊 Résultats

Évaluation sur un benchmark de **263 paires question-réponse validées par des experts** (accord inter-annotateurs κ = 0.79), comparée à un RAG classique.

| Métrique | RAG classique | Multi-Agent (ce projet) |
|---|---|---|
| **Exactitude des citations** (loi + n° d'article corrects) | 25.9 % (68/263) | **93.9 % (247/263)** |
| **BERTScore F1** (vs. réponses expertes) | 0.378 | **0.845** |

L'exactitude des citations est confirmée par une évaluation humaine indépendante (κ = 0.81, 60 questions, 2 annotateurs experts).

**Métriques RAGAS par thème**

| Métrique | Baseline (Baleine) | Multi-Agent (Baleine) | Baseline (Hydrocarbures) | Multi-Agent (Hydrocarbures) |
|---|---|---|---|---|
| Faithfulness | 0.41 | **0.83** | 0.39 | **0.88** |
| Answer Relevancy | 0.51 | **0.84** | 0.48 | **0.87** |
| Context Recall | 0.22 | **0.86** | 0.19 | **0.89** |
| Answer Correctness | 0.14 | **0.78** | 0.12 | **0.90** |

**Étude d'ablation (Chasse à la baleine)** — les graphes et les agents ont un effet super-additif :
sans graphes → BERTScore 0.559 · sans agents → 0.579 · système complet → **0.882**.

**Erreurs résiduelles identifiées** : précédence temporelle (loi ancienne citée au lieu d'une loi révisée, non résolue), contamination juridictionnelle (atténuée par filtrage des métadonnées), exceptions implicites (atténuées par un prompt à contrainte négative).

---

## 🏗️ Structure du Projet

```
PFE_IRD/
│
├── 📄 server.py              ← Serveur Flask (API backend + authentification)
├── 📄 index.html             ← Interface Web (Dashboard chatbot + monitoring)
├── 📄 main.py                ← Point d'entrée du pipeline d'ingestion autonome
├── 📄 process_pdf.py         ← Traitement des PDFs (LlamaParse + structuration)
├── 📄 requirements.txt       ← Dépendances Python du projet racine
├── 📄 .env.example           ← Modèle de configuration (variables d'environnement)
│
├── 📁 legal_rag/             ← ⭐ Module principal RAG Multi-Graphes
│   ├── agents.py             ← Définition des 5 agents LangGraph
│   ├── graph_builder.py      ← Construction des graphes (lexical, citations, défis)
│   ├── retrieval_system.py   ← Moteur de recherche hybride + reranker
│   ├── main_data_integration.py ← Intégration et routage global multi-documents
│   ├── data_loader.py        ← Chargement et indexation des documents traités
│   ├── models.py             ← Modèles de données (Pydantic)
│   ├── config.py             ← Configuration centralisée du module
│   └── requirements.txt      ← Dépendances spécifiques du module RAG
│
├── 📁 src/                   ← Module de veille et d'ingestion automatique
│   ├── parser.py             ← Extraction PDF → JSON structuré (via LlamaParse+Groq)
│   ├── monitor.py            ← Veille législative Internet (scraping + détection)
│   ├── watcher.py            ← Surveillance du dossier d'entrée (watchdog)
│   ├── scheduler.py          ← Planificateur des tâches automatiques
│   ├── notifier.py           ← Envoi d'alertes email (SMTP)
│   └── config.py             ← Configuration du module d'ingestion
│
├── 📁 scripts/               ← Scripts utilitaires et d'expérimentation
│   ├── expert_distiller_v4.py ← Distillation des Q&A experts pour l'évaluation
│   ├── legal_multi_agent.py  ← Script de test du pipeline multi-agents
│   ├── retrieval_fusion.py   ← Tests de fusion de retrieval
│   └── export_to_neo4j.py    ← Export des graphes vers Neo4j (optionnel)
│
├── 📁 scratch/               ← Scripts de test et de débogage
│   └── test_flask_endpoint.py ← Test de l'API REST /api/query
│
├── 📁 data/                  ← [Non versionné] Données brutes
│   ├── input/                ← PDFs à ingérer
│   ├── parsed/               ← Markdowns extraits par LlamaParse
│   ├── structured/           ← JSONs structurés (clauses + définitions)
│   └── archive/              ← PDFs traités et archivés
│
├── 📁 data_processed/        ← [Non versionné] Nodes RAG indexés (JSON)
├── 📁 chroma_db/             ← [Non versionné] Index vectoriel local (ChromaDB)
├── 📁 model_cache_v2/        ← [Non versionné] Cache des modèles d'embedding
└── 📁 lib/                   ← Bibliothèques JS (Vis.js, Tom-Select)
```

---

## 🛠️ Installation et Configuration

### 1. Prérequis
| Outil | Version | Usage |
|---|---|---|
| **Python** | 3.12+ | Langage principal |
| **pip** | 23+ | Gestionnaire de paquets |
| **Git** | 2.40+ | Versionnement |
| **Compte Groq** | — | LLM (llama-3.1-8b-instant) |
| **Compte LlamaParse** | — | Optionnel, extraction PDF avancée |

### 2. Cloner le projet et créer l'environnement virtuel

```powershell
# Cloner le dépôt
git clone https://github.com/ASSIAbouamir/PFE_IRD.git
cd PFE_IRD

# Créer et activer l'environnement virtuel
python -m venv .venv

# Sur Windows (PowerShell) :
.venv\Scripts\Activate.ps1

# Sur macOS / Linux :
source .venv/bin/activate

# Installer toutes les dépendances
pip install -r requirements.txt
```

> [!NOTE]
> L'installation de `torch`, `sentence-transformers` et `faiss-cpu` peut prendre plusieurs minutes et nécessite ~3 Go d'espace disque.

### 3. Configurer les variables d'environnement

Copiez le fichier modèle et remplissez vos valeurs :
```powershell
cp .env.example .env
```

| Variable | Description | Exemple |
|---|---|---|
| `GROQ_API_KEY` | **Obligatoire.** Clé API Groq | `gsk_...` |
| `GROQ_MODEL` | Modèle LLM à utiliser | `llama-3.1-8b-instant` |
| `LLAMA_CLOUD_API_KEY` | Optionnel. Clé LlamaParse pour PDFs complexes | `llx-...` |
| `OPENAI_API_KEY` | Optionnel. Pour les embeddings OpenAI | `sk-...` |
| `TOP_K_VECTOR` | Nb de résultats vectoriels à récupérer | `10` |
| `TOP_K_FINAL` | Nb de passages finaux après reranking | `5` |
| `ALERT_EMAIL_TO` | Destinataires des alertes de veille | `user@example.com` |
| `SMTP_HOST` | Serveur d'envoi email | `smtp.gmail.com` |
| `SMTP_PASSWORD` | Mot de passe app SMTP | `xxxx xxxx xxxx xxxx` |

---

## 🚀 Utilisation

### Étape 1 — Lancer le Serveur Backend

```powershell
python server.py
```

Le serveur démarre sur **`http://127.0.0.1:5000`** et connecte automatiquement le moteur RAG au corpus indexé.

> [!NOTE]
> **Problème résolu :** Nous avons ajouté `use_reloader=False` pour empêcher le serveur Flask de redémarrer au milieu d'une requête RAG lorsque PyTorch charge ses fichiers de cache dynamiques. Sans ce correctif, le serveur coupe la connexion HTTP avec l'erreur `ConnectionResetError(10054)`.

---

### Étape 2 — Accéder à l'Interface Web

Ouvrez votre navigateur sur **`http://127.0.0.1:5000`**

**Connexion :** deux rôles sont disponibles (🧑‍⚖️ Avocat / Expert et ⚙️ Administrateur). Les comptes de démonstration sont créés au premier lancement dans `server.py`.

> [!WARNING]
> Ces comptes de démonstration servent uniquement en local. Modifiez leurs mots de passe avant tout déploiement.

---

### Étape 3 — Utiliser le Chatbot RAG

Une fois connecté, naviguez vers l'onglet **💬 Chatbot RAG** :

1. **Sélectionner un pays** dans le menu déroulant (ex: `Bénin`, `Cameroun`, `Togo`...) ou laisser `Tous` pour une recherche globale.
2. **Sélectionner un agent** selon votre besoin :
   - `Recursive Retrieval Agent` — requête générale multi-documents
   - `Recherche` — Requête rapide avec routing automatique
3. **Saisir votre question juridique** dans la zone de texte.
4. **Observer le pipeline multi-agents** s'exécuter visuellement (recherche initiale puis 5 agents).

**Exemple de questions :**
```
"Est-ce que le rejet d'hydrocarbures au Bénin entraîne une peine de prison ?"
"Quelles sont les sanctions en cas de capture de baleines au Cameroun ?"
"Y a-t-il une interdiction de l'antifouling TBT au Togo ?"
```

---

### Étape 4 — Ingestion de Nouveaux Documents (Optionnel)

Pour ajouter un nouveau document juridique au système :

**a) Déposer le PDF** dans le dossier `data/input/` en nommant le fichier selon la convention :
```
[Theme]_[Pays].pdf
# Exemples :
Baleine_Maroc.pdf
Rejet_hydrocarbure_Senegal.pdf
```

**b) Lancer l'ingestion ponctuelle :**
```powershell
python main.py --once
```

Le pipeline extrait automatiquement le texte (LlamaParse), le structure en clauses JSON (Groq), et indexe le document dans le moteur RAG.

**c) Activer la veille autonome continue :**
```powershell
python main.py --monitor
```

Le système surveille les nouvelles publications législatives sur Internet toutes les `MONITOR_INTERVAL_HOURS` heures et envoie des alertes email en cas de changement détecté.

---

## 🧪 Tests et Vérification

### Tester l'API REST directement

```powershell
python scratch/test_flask_endpoint.py
```

Attendu en sortie :
```
STATUS CODE: 200

ANSWER:
Oui, selon l'article 638 de la loi n°2010-11...

SOURCES:
- ID: QA_Injection_... | Title: Loi n° 98-030 | Country: Bénin

RESOLVED DOC:
Theme: Rejet hydrocarbure | Country: Bénin
```

### Endpoints API disponibles

| Méthode | URL | Description |
|---|---|---|
| `POST` | `/api/query` | ⭐ Soumettre une question au RAG |
| `GET` | `/api/stats` | Statistiques globales du système |
| `GET` | `/api/reports` | Liste des rapports de veille générés |
| `GET` | `/api/monitoring/status` | Statut du module de veille |
| `POST` | `/api/login` | Authentification utilisateur |
| `POST` | `/api/upload_document` | Uploader un PDF via l'interface |
| `GET` | `/api/conflicts` | Liste des conflits juridiques détectés |

---

## 📦 Déploiement et Sauvegarde sur GitHub

### Configuration initiale du dépôt distant

Si le dépôt distant n'est pas encore configuré :
```powershell
git init
git remote add origin https://github.com/ASSIAbouamir/PFE_IRD.git
git branch -M main
```

### Pousser des modifications

```powershell
# 1. Vérifier l'état des fichiers modifiés
git status

# 2. Ajouter les fichiers (le .gitignore exclut automatiquement les données locales)
git add .

# 3. Créer un commit descriptif
git commit -m "feat: description de votre modification"

# 4. Pousser vers GitHub
git push origin main
```

> [!IMPORTANT]
> Les fichiers suivants sont **exclus automatiquement** du dépôt GitHub par `.gitignore` :
> - `chroma_db/` — Index vectoriel local (volumieux, régénérable)
> - `users.db` — Base de données SQLite des utilisateurs (données personnelles)
> - `model_cache_v2/` — Cache des modèles HuggingFace (volumieux, re-téléchargeable)
> - `data/` et `data_processed/` — Documents juridiques (confidentiels)
> - `*.log` — Logs d'exécution
> - `lexical_graph.json` et `definitions_graph.json` — Graphes générés (>10 Mo)
> - `.env` — Vos clés API (ne jamais versionner !)

> [!CAUTION]
> Ne jamais committer votre fichier `.env`. Seul `.env.example` (sans valeurs réelles) doit être versionné.

---

## 🤖 Description des 5 Agents LangGraph

Après une recherche hybride initiale, 5 agents spécialisés enrichissent le contexte jusqu'à ce qu'il soit complet (3 itérations au maximum).

| Agent | Rôle |
|---|---|
| ⚖️ **Supervisor Agent** | Évalue l'état du dossier, choisit l'agent à activer, vérifie la complétude du contexte et décide d'une nouvelle itération (3 au maximum) |
| 💡 **Definition Agent** | Injecte les définitions légales officielles des termes juridiques non résolus, depuis le graphe des définitions |
| 🎯 **Reference Detector Agent** | Détecte les renvois explicites dans les clauses récupérées (`voir article X`, `conformément à la loi Y`) |
| 🔗 **Cross-Reference Resolution Agent** | Résout les renvois via le graphe de citations et récupère les clauses citées ; relance un cycle si de nouvelles références apparaissent |
| 📝 **Answering Agent** | Rédige la réponse finale avec une section « Sources consultées » obligatoire |

---

## 🗂️ Thèmes et Pays Couverts

Corpus de droit africain francophone : **50 PDF uniques**, **17 juridictions**, **7 943 DocumentNodes** (clauses qualifiées). Ce corpus multi-juridictionnel de droit civil n'est pas couvert par les benchmarks existants (LegalBench, CLERC).

| Thème | Documents | Clauses | Paires Q/R du benchmark |
|---|---|---|---|
| 🐋 **Chasse à la baleine** | 23 | 2 080 | 131 |
| 🦅 **Oiseaux marins** | 14 | 957 | — (corpus seulement) |
| 🛢️ **Rejets d'hydrocarbures** | 21 | 4 906 | 132 |

L'interface prend également en charge les thèmes **TBT (antifouling)** et **Général** (textes généraux du droit maritime et environnemental).

---

## 👩‍💻 Auteur et Encadrants

| Rôle | Nom |
|---|---|
| **Étudiante** | Assia Bouamir |
| **Encadrante FSSM** | Dr. Jihad Zahir |
| **Encadrante IRD** | Dr. Marie Bonnin |
| **Examinatrice** | Dr. Latifa Errajy |
| **Examinatrice** | Dr. Hasna El Alaoui |

---

*Année universitaire 2025-2026 — Université Cadi Ayyad, Faculté des Sciences Semlalia, Marrakech*
