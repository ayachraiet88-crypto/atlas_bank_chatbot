# Atlas Bank AI Assistant

Chatbot documentaire bancaire basé sur **LangChain**, **RAG (Retrieval-Augmented Generation)**, **ChromaDB** et **Gemini**, développé dans le cadre du Projet 2 — Document Chatbot with RAG.

Le chatbot répond aux questions des clients d'une banque fictive (**Atlas Bank**) en s'appuyant exclusivement sur des documents internes fournis (règlements, FAQ, procédures, catalogue produits, tarifs), et peut également effectuer des calculs de simulation de crédit et répondre à des questions générales via Wikipédia.

---

## 1. Objectif du projet

Démontrer qu'un système RAG peut :
- Charger et indexer des documents hétérogènes (PDF, DOCX, CSV, XLSX, TXT)
- Retrouver les passages pertinents par recherche sémantique
- Générer des réponses fiables, ancrées dans les documents, via un LLM
- Décider intelligemment quel outil utiliser (recherche documentaire, calcul, ou connaissance générale) via un agent LangChain

---

## 2. Stack technique

| Composant | Choix |
|---|---|
| Framework agent | LangChain (`create_agent`) |
| LLM | Google Gemini (`gemini-3.6-flash`) |
| Embeddings | Google Gemini Embedding (`gemini-embedding-001`) |
| Base vectorielle | ChromaDB |
| Environnement | Google Colab |

---

## 3. Documents utilisés (base de connaissances)

| Fichier | Type | Contenu |
|---|---|---|
| `reglement_credits.pdf` | PDF | Règlement général des crédits (types, conditions, documents requis, pénalités, assurance emprunteur) |
| `faq_bancaire.docx` | DOCX | FAQ complète service clientèle (10 questions/réponses) |
| `procedures_internes.docx` | DOCX | Procédures internes : ouverture de compte, instruction d'un dossier de crédit, gestion des réclamations |
| `produits_bancaires.csv` | CSV | Catalogue de 14 produits/services bancaires (comptes, cartes, crédits, assurances) |
| `services_tarifs.xlsx` | Excel (3 feuilles) | Agences et services, taux de crédits, frais et tarifs |
| `faq_courtes.txt` | TXT | FAQ courte (contacts, horaires, délais) |

Tous les documents sont des données **fictives**, créées pour les besoins du projet.

---

## 4. Architecture

```text
Documents (PDF, DOCX, CSV, XLSX, TXT)
            │
            ▼
    Document Loaders
   (par type de fichier)
            │
            ▼
   LangChain Documents
            │
            ▼
RecursiveCharacterTextSplitter
     (chunk_size=800, overlap=150)
            │
            ▼
        Chunks
            │
            ▼
  Gemini Embeddings
            │
            ▼
        ChromaDB
            │
            ▼
       Retriever (k=4)
            │
            ▼
 Outil : search_bank_documents
            │
            │        Outil : simulate_loan_payment
            │        Outil : search_wikipedia
            │              │
            └──────────────┼───────────────┐
                            ▼               │
                      LangChain Agent ◄─────┘
                    (system prompt métier)
                            │
                            ▼
                      Gemini (LLM)
                            │
                            ▼
                     Réponse finale
```

---

## 5. Outils de l'agent

### `search_bank_documents(query)`
Recherche sémantique dans les documents internes via ChromaDB. Utilisé pour toute question sur les produits, taux, conditions, frais ou procédures.

### `simulate_loan_payment(amount_dt, annual_rate_percent, duration_months)`
Calcul pur (formule d'amortissement à taux fixe) : mensualité, coût total, intérêts totaux. Utilisé uniquement quand l'utilisateur demande un calcul avec des chiffres précis.

### `search_wikipedia(topic)`
Recherche de connaissances générales, utilisée uniquement pour les questions hors du périmètre bancaire interne (ex : définitions économiques).

L'agent choisit lui-même l'outil (ou la combinaison d'outils) à utiliser, selon le system prompt qui définit les règles de priorité entre les trois.

---

## 6. Installation et exécution (Google Colab)

1. Ouvrir un notebook Google Colab
2. Générer une clé API sur [Google AI Studio](https://aistudio.google.com/app/apikey)
3. Ajouter la clé dans les **Secrets** de Colab sous le nom `GOOGLE_API_KEY`
4. Installer les dépendances :
   ```bash
   pip install -qU langchain langchain-community langchain-google-genai \
       langchain-chroma langchain-text-splitters chromadb pypdf docx2txt \
       pandas openpyxl requests
   ```
5. Uploader le dossier `documents/` dans l'environnement Colab
6. Exécuter le notebook dans l'ordre : chargement des documents → chunking → embeddings/ChromaDB → création des outils → création de l'agent → tests

---

## 7. Exemples de questions testées

**Question documentaire simple :**
> Quels sont les documents nécessaires pour demander un crédit immobilier ?

**Question multi-outils (RAG + calcul) :**
> Je veux emprunter 30000 DT sur 48 mois pour un crédit consommation. Quel taux Atlas Bank applique-t-il, et peux-tu me calculer la mensualité avec le taux maximum ?
→ L'agent récupère le taux réel (11,9%) via le RAG, puis calcule la mensualité (788,54 DT/mois).

**Question avec mémoire conversationnelle :**
> Quels sont les frais de tenue de compte ? → Et pour une carte Gold ?
→ Le contexte de la première question est conservé pour interpréter la seconde.

**Question hors périmètre (bascule vers Wikipédia) :**
> Explique-moi ce qu'est l'inflation.
→ L'agent détecte que ce n'est pas une question bancaire interne et utilise `search_wikipedia`.

---

## 8. Résultat obtenu

Le chatbot :
- Charge et indexe correctement 6 documents de 5 formats différents (40 documents bruts → 51 chunks)
- Répond avec des informations exactes, ancrées dans les documents (pas d'invention de taux ou de conditions)
- Combine plusieurs outils dans une même réponse quand nécessaire
- Conserve le contexte conversationnel sur plusieurs échanges
- Sait déléguer à Wikipédia les questions hors de son périmètre documentaire

---

## 9. Limites et pistes d'amélioration


- Pas de mémoire persistante entre les sessions (l'historique de conversation est perdu à la fermeture du notebook)
- Le tier gratuit de l'API Gemini impose des limites de requêtes par minute/jour, à surveiller en cas de démonstration prolongée
