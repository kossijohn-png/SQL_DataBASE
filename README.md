# 🫀 SQL — Heart Disease Database

> Exploration et analyse de données cardiologiques avec MySQL

Ce projet constitue pour moi une première mise en pratique des compétences en SQL à travers le traitement d’une première base de données médicale.

---

## 📋 Description du projet

Ce projet contient un ensemble de requêtes SQL réalisées sur la base de données **Heart Disease**, issue de l'UCI Machine Learning Repository (1988). Elle compile des données de quatre institutions : **Cleveland, Hongrie, Suisse et Long Beach V**.

L’objectif de ce travail est d’explorer, nettoyer, transformer et analyser des données médicales afin de mieux comprendre les facteurs associés aux maladies cardiaques.

---

## 🗄️ Structure de la base de données

La table principale `heart` contient **76 attributs**, dont 14 sont utilisés dans ce projet :

| Colonne | Description |
|--------|-------------|
| `id` | Identifiant patient |
| `age` | Âge du patient |
| `sex` | Sexe (0 = Femme, 1 = Homme) |
| `cp` | Type de douleur thoracique (4 valeurs) |
| `trestbps` | Tension artérielle au repos (mmHg) |
| `chol` | Cholestérol sérique (mg/dl) |
| `fbs` | Glycémie à jeun > 120 mg/dl |
| `restecg` | Résultats ECG au repos (0, 1, 2) |
| `thalach` | Fréquence cardiaque maximale atteinte |
| `exang` | Angine induite par l'exercice |
| `oldpeak` | Dépression ST induite par l'exercice |
| `slope` | Pente du segment ST d'exercice |
| `ca` | Nombre de grands vaisseaux colorés (0–3) |
| `thal` | Qualité de perfusion du muscle cardiaque |
| `target` | **Variable Dépendante** : 0 = Pas de maladie, 1 = Maladie |

---

## 📂 Contenu des exercices

---

### 1. 🔍 Exploration de base

**Afficher les 10 premières lignes de la table**
```sql
SELECT * FROM heart LIMIT 10;
SELECT * FROM heart LIMIT 10 OFFSET 5;
SELECT * FROM heart WHERE age < 50 LIMIT 10;
SELECT age FROM heart LIMIT 5;
```

**Lister toutes les colonnes de la table**
```sql
SHOW COLUMNS FROM heart;
```

**Compter le nombre total d'observations (patients)**
```sql
SELECT COUNT(*) FROM heart;
```

**Identifier les modalités distinctes de la variable `sex` et `cp`**
```sql
SELECT DISTINCT sex FROM heart;
SELECT DISTINCT cp FROM heart;
```

---

### 2. 🧹 Vérification et nettoyage des données

**Détecter les valeurs NULL et les compter dans chaque colonne**
```sql
SELECT
  SUM(CASE WHEN age IS NULL THEN 1 ELSE 0 END)       AS null_age,
  SUM(CASE WHEN sex IS NULL THEN 1 ELSE 0 END)       AS null_sex,
  SUM(CASE WHEN cp IS NULL THEN 1 ELSE 0 END)        AS null_cp,
  SUM(CASE WHEN trestbps IS NULL THEN 1 ELSE 0 END)  AS null_trestbps,
  SUM(CASE WHEN chol IS NULL THEN 1 ELSE 0 END)      AS null_chol,
  SUM(CASE WHEN fbs IS NULL THEN 1 ELSE 0 END)       AS null_fbs,
  SUM(CASE WHEN restecg IS NULL THEN 1 ELSE 0 END)   AS null_restecg,
  SUM(CASE WHEN thalach IS NULL THEN 1 ELSE 0 END)   AS null_thalach,
  SUM(CASE WHEN exang IS NULL THEN 1 ELSE 0 END)     AS null_exang,
  SUM(CASE WHEN oldpeak IS NULL THEN 1 ELSE 0 END)   AS null_oldpeak,
  SUM(CASE WHEN slope IS NULL THEN 1 ELSE 0 END)     AS null_slope,
  SUM(CASE WHEN ca IS NULL THEN 1 ELSE 0 END)        AS null_ca,
  SUM(CASE WHEN thal IS NULL THEN 1 ELSE 0 END)      AS null_thal,
  SUM(CASE WHEN target IS NULL THEN 1 ELSE 0 END)    AS null_target
FROM heart;
```

---

### 3. 🔧 Transformation des variables

**Créer une colonne `age_group` (< 40, 40-60, >= 60)**
```sql
ALTER TABLE heart ADD COLUMN age_group VARCHAR(20);

SHOW COLUMNS FROM heart;

UPDATE heart
SET age_group = CASE
  WHEN age < 40 THEN '<40 ans'
  WHEN age BETWEEN 40 AND 59 THEN '[40-60[ ans'
  ELSE '>= 60 ans'
END;

SELECT DISTINCT age_group FROM heart;
```

**Créer une colonne `tension_group` (Normale, Élevée, HTA Stade 1, HTA Stade 2)**
```sql
ALTER TABLE heart ADD COLUMN tension_group VARCHAR(10);
ALTER TABLE heart MODIFY COLUMN tension_group VARCHAR(50);

UPDATE heart
SET tension_group = CASE
  WHEN trestbps < 120 THEN 'NORMALE'
  WHEN trestbps BETWEEN 120 AND 129 THEN 'ÉLEVÉE'
  WHEN trestbps BETWEEN 130 AND 139 THEN 'HTA STADE 1'
  ELSE 'HTA STADE 2'
END;

SELECT DISTINCT tension_group FROM heart;
```

**Créer une colonne `fc_group` (Bradycardie, Normale, Tachycardie)**
```sql
ALTER TABLE heart ADD COLUMN fc_group VARCHAR(50);

UPDATE heart
SET fc_group = CASE
  WHEN thalach < 60 THEN 'Bradycardie'
  WHEN thalach BETWEEN 60 AND 120 THEN 'Normale'
  ELSE 'Tachycardie'
END;

SELECT * FROM heart LIMIT 5;
```

**Créer une colonne `ischemie`**
```sql
ALTER TABLE heart ADD COLUMN ischemie VARCHAR(50);

UPDATE heart
SET ischemie = CASE
  WHEN oldpeak = 0    THEN 'Pas d\'ischémie'
  WHEN oldpeak BETWEEN 0.1 AND 0.5  THEN 'Anomalie mineure'
  WHEN oldpeak BETWEEN 0.5 AND 1.0  THEN 'Suspicion d\'ischémie'
  WHEN oldpeak > 1.0  THEN 'Présence d\'ischémie'
END;
```

**Transformer `sex` en texte via une table jointe `heart_sex`**
```sql
-- Suppression de la colonne sex initiale
ALTER TABLE heart DROP COLUMN sex;

-- Nettoyage de la table heart_sex
DELETE FROM heart_sex WHERE id = 0 AND sex = 0;
SELECT * FROM heart_sex LIMIT 10;

-- Transformation des valeurs dans heart_sex
ALTER TABLE heart_sex MODIFY COLUMN sex VARCHAR(50);

UPDATE heart_sex
SET sex = CASE
  WHEN sex = 1 THEN 'Homme'
  WHEN sex = 0 THEN 'Femme'
END;

SELECT * FROM heart_sex LIMIT 10;

-- Réintégration dans la table heart via JOIN
ALTER TABLE heart ADD COLUMN sex VARCHAR(50);

UPDATE heart
JOIN heart_sex ON heart.id = heart_sex.id
SET heart.sex = heart_sex.sex;
```

**Renommer la colonne `max_heart` en `max_FC`**
```sql
ALTER TABLE heart CHANGE max_heart max_FC INT;
```

**Convertir `target` en texte (0 = Pas de maladie, 1 = Maladie)**
```sql
ALTER TABLE heart MODIFY COLUMN target VARCHAR(50);

UPDATE heart
SET target = CASE
  WHEN target = 0 THEN 'Pas de maladie cardiaque'
  WHEN target = 1 THEN 'Maladie Cardiaque'
END;
```

---

### 4. 📊 Analyse descriptive

**Calculer l'âge moyen des patients**
```sql
-- Âge moyen global → 54,43 ans
SELECT AVG(age) FROM heart;

-- Âge moyen par sexe → Femmes : 55,84 ans | Hommes : 53,81 ans
SELECT sex, AVG(age) AS age_moyen FROM heart GROUP BY sex;
```

**Calculer l'âge minimum et maximum**
```sql
SELECT MIN(age) FROM heart;  -- Résultat : 29 ans
SELECT MAX(age) FROM heart;  -- Résultat : 77 ans
```

**Calculer la fréquence de la variable `age_group`**
```sql
SELECT
  age_group,
  COUNT(*) AS effectif,
  ROUND((COUNT(*) * 100) / (SELECT COUNT(*) FROM heart), 2) AS pourcentage
FROM heart
GROUP BY age_group;
```

**Trouver le taux de patients ayant une maladie cardiaque**
```sql
SELECT
  target,
  COUNT(*) AS effectifs,
  ROUND((COUNT(*) * 100) / (SELECT COUNT(*) FROM heart), 2) AS pourcentage
FROM heart
GROUP BY target;
```

**Calculer la pression artérielle moyenne par sexe, tranche d'âge et maladie**
```sql
SELECT sex, AVG(trestbps) AS mean_TA FROM heart GROUP BY sex;

SELECT age_group, AVG(trestbps) AS mean_TA FROM heart GROUP BY age_group;

SELECT target, AVG(trestbps) AS mean_TA FROM heart GROUP BY target;
```

---

### 5. 🔎 Filtrage et conditions

**Sélectionner les patients âgés de plus de 50 ans avec maladie cardiaque**
```sql
SELECT * FROM heart
WHERE age > 50 AND target = 'Maladie Cardiaque'
LIMIT 5;
```

**Trouver les patients ayant un taux de cholestérol supérieur à 240**
```sql
-- Nombre de patients → 503
SELECT COUNT(*) FROM heart WHERE chol > 240;

SELECT * FROM heart WHERE chol > 240 LIMIT 10;
```

---

### 6. 🚀 Analyse avancée

**Calculer la proportion de maladie cardiaque par tranche d'âge**
```sql
SELECT
  age_group,
  COUNT(*) AS effectifs,
  COUNT(CASE WHEN target = 'Maladie Cardiaque' THEN 1 END) AS malade,
  ROUND(
    (COUNT(CASE WHEN target = 'Maladie Cardiaque' THEN 1 END) * 100.0) / COUNT(*),
    2
  ) AS pourcentage
FROM heart
GROUP BY age_group;
```

**Calculer la proportion de maladie cardiaque par sexe**
```sql
SELECT
  sex,
  COUNT(*) AS effectif,
  COUNT(CASE WHEN target = 'Maladie Cardiaque' THEN 1 END) AS malade,
  ROUND(
    COUNT(CASE WHEN target = 'Maladie Cardiaque' THEN 1 END) * 100 / COUNT(*),
    2
  ) AS pourcentage
FROM heart
GROUP BY sex;
```

**Trouver la relation entre type de douleur thoracique et maladie**
```sql
SELECT
  cp,
  COUNT(*) AS effectif,
  COUNT(CASE WHEN target = 'Maladie Cardiaque' THEN 1 END) AS malade,
  ROUND(
    COUNT(CASE WHEN target = 'Maladie Cardiaque' THEN 1 END) * 100 / COUNT(*),
    2
  ) AS pourcentage
FROM heart
GROUP BY cp;
```

---

## 🛠️ Technologies utilisées

- **MySQL** (via phpMyAdmin)
- Fonctions SQL : `SELECT`, `UPDATE`, `ALTER TABLE`, `JOIN`, `GROUP BY`, `CASE WHEN`, `COUNT`, `AVG`, `MIN`, `MAX`, `ROUND`

---

## 🚀 Utilisation

1. Importer la base de données `heart` dans votre environnement MySQL.
2. Exécuter les requêtes dans l'ordre des sections pour reproduire l'analyse.
3. Les requêtes de transformation modifient la structure de la table — penser à travailler sur une copie si nécessaire.

---

## 📈 Quelques résultats clés

- 🧑‍⚕️ Âge moyen des patients : **54,43 ans**
- 👩 Âge moyen des femmes : **55,84 ans** | 👨 Hommes : **53,81 ans**
- 📉 Âge minimum : **29 ans** | Maximum : **77 ans**
- 🩸 Patients avec cholestérol > 240 mg/dl : **503 patients**

---

## 📚 Source des données

> Heart Disease Dataset — UCI Machine Learning Repository, 1988  
> Institutions : Cleveland, Hongrie, Suisse, Long Beach V

---

## 👤 Auteur
DEGBEVI John
Projet réalisé dans le cadre d'un exercice d'analyse de données médicales avec SQL.
