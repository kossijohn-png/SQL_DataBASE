# 🦠 Hantavirus (Virus des Andes) — Épidémiologie Mondiale

> Ensemble de données professionnel de niveau recherche regroupant les données mondiales sur le **Syndrome Pulmonaire à Hantavirus (HPS)** et la **Fièvre Hémorragique avec Syndrome Rénal (HFRS)**, incluant le virus des Andes — seul hantavirus avec transmission interhumaine documentée.

![Couverture](https://img.shields.io/badge/Pays%20couverts-25-blue) ![Régions OMS](https://img.shields.io/badge/Régions%20OMS-5-green) ![Période](https://img.shields.io/badge/Période-1993--2025-orange) ![Licence](https://img.shields.io/badge/Licence-Research-lightgrey)

---

## 📦 Contenu du jeu de données

| Fichier | Description |
|---|---|
| `hantavirus_country_yearly.csv` | Données épidémiologiques annuelles par pays (1993–2025) |
| `hantavirus_outbreaks.csv` | Chronologie des événements majeurs d'épidémie |
| `hantavirus_monthly_trends.csv` | Tendances mensuelles des cas pour les principaux pays endémiques |
| `hantavirus_clinical.csv` | Présentation clinique : symptômes, gravité, résultats |
| `hantavirus_environmental.csv` | Facteurs de risque environnementaux par région et trimestre |
| `hantavirus_virus_strains.csv` | Référence des souches virales avec données génomiques |
| `hantavirus_master.csv` | Ensemble de données maître consolidé |
| `hantavirus_Andes_Global_Registry.csv` | Registre mondial spécifique au virus des Andes |
| `sources_metadata.csv` | Métadonnées et traçabilité des sources |

---

## 🔬 Méthodologie

Données synthétisées à partir de :

- **OMS / WHO** — rapports épidémiologiques officiels
- **CDC** — surveillance des cas aux Amériques
- **PAHO / ECDC / NIH** — données régionales et études de cohorte
- **Littérature PubMed** — publications évaluées par des pairs

---

## 🗄️ Exploration SQL

### 1. Explorer la table

```sql
-- Afficher toutes les données
SELECT * FROM hantavirus_andes_global_registry;

-- Afficher les colonnes
SHOW COLUMNS FROM hantavirus_andes_global_registry;

-- Compter les enregistrements
SELECT COUNT(*) FROM hantavirus_andes_global_registry;
```

---

### 2. Transformation de variables

#### Définir `record_id` comme clé primaire

```sql
ALTER TABLE hantavirus_andes_global_registry
  MODIFY record_id VARCHAR(20) PRIMARY KEY;
```

#### Créer la variable `epidemie` (Yes / No)

```sql
ALTER TABLE hantavirus_andes_global_registry
  ADD COLUMN epidemie VARCHAR(5);

UPDATE hantavirus_andes_global_registry
  SET epidemie = CASE
    WHEN outbreak_type = 'Outbreak' THEN 'Yes'
    ELSE 'No'
  END;

-- Vérification
SELECT epidemie, outbreak_type FROM hantavirus_andes_global_registry;
```

#### Recoder la variable `month` (numérique → libellé)

```sql
ALTER TABLE hantavirus_andes_global_registry
  MODIFY month VARCHAR(15);

UPDATE hantavirus_andes_global_registry
  SET month = CASE
    WHEN month = 1  THEN 'January'
    WHEN month = 2  THEN 'February'
    WHEN month = 3  THEN 'March'
    WHEN month = 4  THEN 'April'
    WHEN month = 5  THEN 'May'
    WHEN month = 6  THEN 'June'
    WHEN month = 7  THEN 'July'
    WHEN month = 8  THEN 'August'
    WHEN month = 9  THEN 'September'
    WHEN month = 10 THEN 'October'
    WHEN month = 11 THEN 'November'
    WHEN month = 12 THEN 'December'
  END;
```

---

### 3. Analyse descriptive

#### Fréquence par pays

```sql
SELECT
  country,
  COUNT(*) AS effectif,
  ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM hantavirus_andes_global_registry), 2) AS pourcentage
FROM hantavirus_andes_global_registry
GROUP BY country;
```

> 📌 **Résultat :** L'**Argentine (33,3%)** et le **Chili (20,0%)** sont les pays ayant rapporté le plus d'épisodes de Hantavirus.

---

#### Nombre moyen de cas et de décès par pays

```sql
SELECT
  country,
  AVG(cases)  AS moyenne_cas,
  AVG(deaths) AS moyenne_deces
FROM hantavirus_andes_global_registry
GROUP BY country;
```

> 📌 **Résultat :** La **Chine (388 cas)** et la **Suède (185 cas)** enregistrent le plus grand nombre moyen de cas. Concernant les décès, les **USA (31 décès)** et la **Chine (30 décès)** arrivent en tête.

---

#### Taux de létalité (CFR) par pays

```sql
SELECT
  country,
  AVG(cfr_percent) AS letalite
FROM hantavirus_andes_global_registry
GROUP BY country;
```

> ⚠️ **Résultat :** Trois pays enregistrent un taux de létalité supérieur à 50% : **USA (64,60%)**, **Bolivie (57,10%)** et **Paraguay (50,00%)**.

---

### 4. Analyse par souche virale

#### Cas, décès et létalité par espèce

```sql
SELECT
  hantavirus_species,
  AVG(cases)       AS cas_moyens,
  AVG(deaths)      AS deces_moyens,
  AVG(cfr_percent) AS letalite_moyenne
FROM hantavirus_andes_global_registry
GROUP BY hantavirus_species;
```

> 📌 **Résultat :** 6 souches virales distinctes identifiées. Le **Sin Nombre virus** présente le taux de létalité le plus élevé : **64,60%**.

---

#### Distribution des souches virales

```sql
SELECT
  hantavirus_species,
  COUNT(*) AS effectif,
  ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM hantavirus_andes_global_registry), 2) AS pourcentage
FROM hantavirus_andes_global_registry
GROUP BY hantavirus_species;
```

> 📌 **Résultat :** L'**Andes virus (> 60%)** est la souche la plus fréquemment identifiée dans le registre.

---

## 📎 Ressources complémentaires

- 📄 [Dictionnaire des variables](./Hantavirus_Dictionnaire_Variables.docx)
- 🌐 [OMS — Hantavirus](https://www.who.int/news-room/fact-sheets/detail/hantavirus-disease)
- 🌐 [CDC — Hantavirus](https://www.cdc.gov/hantavirus/)
- 🌐 [PAHO — Hantavirus](https://www.paho.org/en/topics/hantavirus)
