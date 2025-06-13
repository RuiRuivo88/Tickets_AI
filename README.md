# Tickets_AI

## 1. Informations Générales

**Nom du projet :** Tickets\_AI
**Responsable du projet :** Rui Ruivo
**Objectif principal :** Mettre en place un système local de classification automatique de tickets d'incident EasyVista selon la qualité de leur rédaction (de 1 à 4), en utilisant un modèle de langage LLaMA fine-tuné.

---

## 2. Objectifs Détaillés

### 2.1 Objectifs fonctionnels

* Nettoyer et normaliser les fichiers CSV issus d'EasyVista.
* Appliquer une classification automatique selon des règles définies.
* Entraîner un modèle LLaMA à partir des tickets.
* Réentraîner le modèle avec de nouvelles données.
* Réaliser des inférences pour prédire automatiquement la qualité de rédaction de nouveaux tickets.

### 2.2 Objectifs non fonctionnels

* Traitement local (aucune donnée ne quitte l'environnement).
* Utilisation du GPU si disponible (support CUDA).
* Compatibilité avec l'environnement Windows.
* Modularité des scripts Python pour extension future (interface vocale, dashboard).

---

## 3. Prérequis Techniques

* **Langage :** Python 3.10+
* **Librairies :**

  * `transformers`, `datasets`, `sklearn`, `torch`, `pandas`, `chardet`, `tokenizers`
* **Matériel recommandé :**

  * GPU NVIDIA (minimum 8 Go VRAM)
  * RAM : 32 Go
  * CPU : 8 cœurs+
* **Système :** Windows avec virtualisation activée

---

## 4. Architecture du Projet

```text
Tickets_AI/
├── cleancsv.py                     # Nettoyage et standardisation des fichiers CSV
├── classnclean4training.py         # Application des règles de classification
├── train_llama_ticket.py           # Entraînement initial du modèle LLaMA
├── finetune.py                     # Réentraînement sans contrainte GPU
├── finetune_gpu.py                 # Réentraînement optimisé pour GPU
├── runinference.py                 # Prédiction sur de nouveaux tickets
├── README.md                       # Documentation projet
├── Output_converted/               # CSV nettoyés et classifiés
├── llama_ticket_model/             # Modèle LLaMA entraîné
└── llama_ticket_model_updated/     # Modèle mis à jour
```

---

## 5. Description des Modules

### 5.1 cleancsv.py

* Détection automatique du séparateur CSV
* Renommage des colonnes : passage de noms francophones vers un format normalisé
* Nettoyage des champs vides : remplissage par "null"

### 5.2 classnclean4training.py

* Classification initiale des tickets selon règles heuristiques :

  * Exemple : `R_` + `nom_ci = null` + `description+solution` présents → classe 4
* Attribution de la valeur `correction` en fonction de ces règles

### 5.3 train\_llama\_ticket.py

* Préparation du dataset (fusion des champs texte, nettoyage)
* Création d’un tokenizer BPE personnalisé
* Entraînement du modèle LLaMA `from scratch`
* Export du modèle dans un format compatible `ollama`

### 5.4 finetune.py / finetune\_gpu.py

* Chargement du modèle existant
* Utilisation des nouvelles données labellisées
* Entraînement avec évaluation automatique
* Possibilité d'utiliser `corrected` pour override de `correction`

### 5.5 runinference.py

* Prédiction de la note de correction sur des tickets non vus
* Génération des probabilités associées à chaque classe
* Export d’un CSV enrichi

---

## 6. Règles de Classification (classnclean4training.py)

| Condition                                | Classe |
| ---------------------------------------- | ------ |
| R\_ & nom\_ci=null & desc+sol OK         | 4      |
| I\_ & nom\_ci=null & desc+sol OK         | 3      |
| I\_ & nom\_ci=null & desc OU sol vide    | 1      |
| I\_ & nom\_ci présent & desc+sol OK      | 4      |
| R\_ & nom\_ci présent & desc OU sol vide | 1      |

---

## 7. Flux de Données (Pipeline)

1. Fichier brut (EasyVista) → `cleancsv.py`
2. CSV propre → `classnclean4training.py` → Ajout de `correction`
3. Fichier classifié → `train_llama_ticket.py` ou `finetune_gpu.py`
4. Modèle entraîné → `runinference.py` → Prédictions

---

## 8. Données attendues

### Champs requis dans le CSV :

* `ticket_id`, `nom_ci`, `sujet`, `description`, `solution`, `correction`, `corrected`

### Données de sortie :

* `correction` prédite (entre 1 et 4)
* Probabilités : `proba_1`, `proba_2`, `proba_3`, `proba_4`

---

## 9. Contraintes

* Traitement strictement local (aucune donnée dans le cloud)
* Modèle LLaMA entraîné à partir de zero avec compatibilité Ollama
* Support GPU obligatoire pour `finetune_gpu.py`
* Fichiers CSV en UTF-8 ou UTF-8-SIG

---

## 10. Améliorations futures

* Intégration dans JARVIS pour traitement vocal
* Visualisation via interface Streamlit ou autre
* Ajout de feedback humain pour correction manuelle et renforcement
* Intégration d'une base vectorielle pour la mémoire contextuelle

---

## 11. Suivi Git

```bash
git add *.py
git add Inputs_raw/
git add Output_converted/
git add Output_final/
git add llama_ticket_model/
git add llama_ticket_model_updated/
git add logs/
git commit -m "Ajout des fichiers d'entraînement et nettoyage"
git push origin main
```
