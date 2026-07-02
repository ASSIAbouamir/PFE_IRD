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
- **Un moteur de recherche hybride** : BM25 + Dense (BGE-M3) + Graph, avec fusion dynamique des scores.
- **Un pipeline multi-agents LangGraph** orchestrant 6 agents spécialisés.
- **Un pipeline d'ingestion autonome** : veille législative, détection de conflits, déduplication temporelle.

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
│   ├── agents.py             ← Définition des 6 agents LangGraph
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

Le serveur démarre sur **`http://127.0.0.1:5000`** et connecte automatiquement le moteur RAG (48 documents indexés).

> [!NOTE]
> **Problème résolu :** Nous avons ajouté `use_reloader=False` pour empêcher le serveur Flask de redémarrer au milieu d'une requête RAG lorsque PyTorch charge ses fichiers de cache dynamiques. Sans ce correctif, le serveur coupe la connexion HTTP avec l'erreur `ConnectionResetError(10054)`.

---

### Étape 2 — Accéder à l'Interface Web

Ouvrez votre navigateur sur **`http://127.0.0.1:5000`**

**Identifiants de connexion :**
| Rôle | Login | Mot de passe |
|---|---|---|
| 🧑‍⚖️ Avocat / Expert | `avocat` | `avocat` |
| ⚙️ Administrateur | `admin` | `admin` |

---

### Étape 3 — Utiliser le Chatbot RAG

Une fois connecté, naviguez vers l'onglet **💬 Chatbot RAG** :

1. **Sélectionner un pays** dans le menu déroulant (ex: `Bénin`, `Cameroun`, `Togo`...) ou laisser `Tous` pour une recherche globale.
2. **Sélectionner un agent** selon votre besoin :
   - `Recursive Retrieval Agent` — requête générale multi-documents
   - `Recherche` — Requête rapide avec routing automatique
3. **Saisir votre question juridique** dans la zone de texte.
4. **Observer le pipeline multi-agents** s'exécuter visuellement (6 étapes animées).

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

## 🤖 Description des 6 Agents LangGraph

| Agent | Rôle |
|---|---|
| ⚖️ **Supervisor Agent (Évaluation)** | Évalue l'état initial du dossier, décide de l'agent à activer en premier |
| 💡 **Definition Agent** | Détecte les termes juridiques ambigus et les enrichit depuis le graphe des définitions |
| 🎯 **Reference Detector Agent** | Identifie les renvois explicites (`voir article X`, `conformément à la loi Y`) |
| 🔗 **Cross-Reference Resolution Agent** | Résout les citations, récupère les clauses référencées dans d'autres documents |
| 🔄 **Supervisor Agent (Complétude)** | Réévalue la complétude du contexte, décide si une itération supplémentaire est nécessaire |
| 📝 **Answering Agent** | Génère la réponse finale formatée avec traçabilité complète des sources |

---

## 🗂️ Thèmes et Pays Couverts

Le système RAG couvre **48 documents** répartis sur **5 thèmes juridiques** :

| Thème | Description |
|---|---|
| 🐋 **Baleine** | Réglementation sur la protection des cétacés et mammifères marins |
| 🛢️ **Rejet hydrocarbure** | Législation sur les rejets pétroliers dans les eaux maritimes |
| 🦅 **Oiseaux marins** | Protection des espèces aviaires marines |
| ⚓ **TBT (Antifouling)** | Réglementation des peintures antifouling aux tributylétain |
| 📋 **General** | Textes généraux du droit maritime et environnemental |

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
