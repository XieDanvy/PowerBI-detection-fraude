# Projet d'Analyse et de Détection de la Fraude Financière

## Présentation du Projet
Ce projet consiste en la mise en place d'une chaîne de traitement de données complète (**Pipeline ETL**) et d'un **Tableau de Bord interactif** permettant d'analyser, modéliser et détecter les comportements frauduleux au sein d'un volume massif de transactions financières.

L'objectif principal est d'identifier les vecteurs d'attaque, de comprendre le comportement temporel des fraudeurs et de fournir des indicateurs clés (KPIs) exploitables pour une équipe de sécurité financière ou de direction des risques.

Le projet s'appuie sur le dataset **PaySim**, une simulation de transactions mobiles basée sur des données réelles, regroupant plus de **6,36 millions de lignes**.

---

## TELECHARGER LE PROJET
> **[Cliquez ici pour télécharger le fichier Power BI (.pbix)](https://drive.google.com/file/d/1KmY8UfR4VfEUJrRVwEfiGDZnZkJY2Q1v/view?usp=sharing)** > *(Note : Le fichier est hébergé sur Google Drive en raison de sa taille de 388 Mo, ce qui dépasse la limite d'envoi direct de GitHub).*

<img width="1528" height="855" alt="Capture d&#39;écran 2026-05-19 003603" src="https://github.com/user-attachments/assets/c7539315-5b1d-429a-bfd6-f510788574aa" />


## Architecture Technique (Stack)
La solution a été pensée pour simuler un environnement de production d'entreprise :
* **Base de Données Relationnelle :** `PostgreSQL` (Stockage massif, indexation et requêtage optimal).
* **Administration DB :** `pgAdmin 4`.
* **ETL & Préparation :** `Power Query` (Connexion sécurisée, filtrage, typage et création de dimensions).
* **Modélisation :** Schéma en Étoile (*Star Schema*).
* **Langage de Calcul :** `DAX` (*Data Analysis Expressions*) pour la création de mesures d'intelligence analytique.
* **Restitution / Restitutions Visuelles :** `Power BI Desktop`.

---

## Étapes de Réalisation

### 1. Stockage & Injection (PostgreSQL)
* Création d'une base de données dédiée localement.
* Configuration et structuration de la table principale `transactions` pour accueillir les 6,3 millions d'enregistrements sans perte de performance.
* Gestion de l'authentification et sécurisation de l'accès au serveur local (`localhost`).

### 2. Phase ETL & Modélisation (Power Query)
Afin de ne pas surcharger la mémoire de l'application et de respecter les bonnes pratiques de la Business Intelligence, le jeu de données brut a été scindé en un **Schéma en Étoile** :
* **`Fact_Transactions` (Table de Faits) :** Contient les informations quantitatives (montants, soldes initiaux/finaux de l'émetteur et du destinataire) et les clés de liaison. Les colonnes inutiles comme `isFlaggedFraud` ont été supprimées pour optimiser la compression.
* **`Dim_Type_Transaction` (Table de Dimension) :** Table normalisée isolant les types uniques de transactions (`TRANSFER`, `CASH_OUT`, `PAYMENT`, `DEBIT`, `CASH_IN`).
* **`Dim_Temps` (Table de Dimension) :** Générée via un script de code **M** personnalisé dans Power Query afin de convertir les blocs temporels bruts (`step`) en données analytiques exploitables :
    * Heure réelle de la journée (de 0h à 23h).
    * Jour chronologique du projet (de 1 à 31 jours).
    * Segmentation par tranches horaires cognitives (Nuit, Matin, Après-midi, Soirée).

### 3. Intelligence Analytique (Mesures DAX)
Les calculs ont été isolés au sein d'une table dédiée (`_Mesures`) sous forme de mesures dynamiques et optimisées :
* **Volume Global :**
    ```dax
    Nombre Transactions = COUNTROWS(Fact_Transactions)
    ```
* **Impact Financier :**
    ```dax
    Montant Fraude = CALCULATE(SUM(Fact_Transactions[amount]), Fact_Transactions[isFraud] = 1)
    ```
* **Sévérité et Ratio :**
    ```dax
    Taux de Fraude = DIVIDE(CALCULATE(COUNTROWS(Fact_Transactions), Fact_Transactions[isFraud] = 1), [Nombre Transactions])
    ```

---

## Insights & Conclusions Clés du Dashboard

Le tableau de bord final offre une interactivité totale (Filtres par boutons dynamiques de type vignettes) et révèle des informations stratégiques majeures :

1.  **Concentration Absolue des Risques :** Sur les 5 types de transactions disponibles, la fraude se concentre **exclusivement** sur deux typologies : les virements (**`TRANSFER`**) et les retraits (**`CASH_OUT`**). Les paiements de proximité ou dépôts ne présentent aucun incident.
2.  **Chirurgie vs Masse :** Le taux global de fraude est très faible (**0,13 %**), mais son impact financier est colossal (**12,06 milliards d'unités monétaires**). Cela démontre que les fraudeurs ciblent de très gros montants unitaires lors d'attaques ciblées plutôt que de multiplier les petites transactions.
3.  **Patterns Temporels (Heatmap) :** L'analyse cartographique matricielle met en évidence que l'activité frauduleuse est continue, y compris la nuit (période où les transactions légitimes chutent drastiquement). Des pics d'attaques massifs et coordonnés ont notamment été isolés lors des jours 17 et 18 du projet aux alentours de 2h du matin.

---

## Structure des Fichiers du Dépôt
* `/Script_PostgreSQL/` : Contient les requêtes SQL de création de table et d'optimisation.
* `/Modèle_PowerBI/` : Contient le fichier de sauvegarde `.pbix` du rapport final structuré.
* `README.md` : Guide d'explication du projet (ce fichier).

---

##  Comment Exécuter le Projet ?
1.  Télécharger le dataset PaySim sur Kaggle via ce lien : [Kaggle - Synthetic Financial Datasets for Fraud Detection](https://www.kaggle.com/datasets/ealaxi/paysim1)
2.  Télécharger le fichier `Dashboard_Fraude_Finanicere.pbix` depuis le lien **Google drive**.
3.  Importer les données dans votre instance locale **PostgreSQL** à l'aide de pgAdmin.
4.  Ouvrir le fichier `Dashboard_Fraude_Finanicere.pbix` dans **Power BI Desktop**.
5.  Si nécessaire, modifier les paramètres de la source de données pour pointer vers votre base locale PostgreSQL et cliquer sur **Actualiser**.
