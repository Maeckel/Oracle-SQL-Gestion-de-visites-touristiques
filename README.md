# 🗺️ TP Oracle SQL — Gestion de visites touristiques

Travaux pratiques **Oracle SQL & PL/SQL** portant sur une base de données fictive de touristes visitant des monuments du monde entier. Le script unique `script.sql` couvre l'intégralité du cycle : création du schéma, insertion de données, requêtes d'interrogation, vues, modifications de structure, fonctions/procédures PL/SQL, triggers et analyse des plans d'exécution.

---

## 📁 Fichiers

| Fichier | Description |
|---|---|
| `script.sql` | Script Oracle SQL complet (schéma + données + requêtes + PL/SQL) |

---

## 🗄️ Schéma de la base de données

### Tables

**`TOURISTE`**

| Colonne | Type | Contrainte |
|---|---|---|
| `nom` | VARCHAR2(20) | PK |
| `prenom` | VARCHAR2(20) | |
| `age` | NUMBER(2) | CHECK age < 100 |
| `paysOrigine` | VARCHAR2(20) | |

**`MONUMENT`**

| Colonne | Type | Contrainte |
|---|---|---|
| `nom` | VARCHAR(25) | PK |
| `ville` | VARCHAR(20) | UNIQUE avec anneeConstruction |
| `pays` | VARCHAR(20) | |
| `anneeConstruction` | INTEGER | UNIQUE avec ville, nullable |

**`VISITE`** *(table de liaison)*

| Colonne | Type | Contrainte |
|---|---|---|
| `nomTouriste` | VARCHAR(20) | PK, FK → TOURISTE (ON DELETE CASCADE) |
| `nomMonument` | VARCHAR(25) | PK, FK → MONUMENT (ON DELETE CASCADE) |
| `dateVisite` | DATE | PK, CHECK ≥ 01/01/2000 |
| `prixEntree` | NUMBER(2) | CHECK ≥ 0 |

### Données de test

- **9 touristes** : Bonaparte, Tepes, Legrand, Deux, Cesar, Darc, Magne, Corcovado, Stote — issus de France, Roumanie, Grèce, Russie, Italie, Allemagne, Brésil
- **11 monuments** : Tour Eiffel, Colisée, Corcovado, Château de Dracula, Notre-Dame, Vatican, Château de Versailles, Tour de Pise, Maison du Peuple, Musée du CNAM, Cathédrale de la Sé
- **42 visites** avec des dates fixes (2009–2014) et des dates relatives à `SYSDATE`

---

## 📋 Contenu du script

### Partie 1 — Interrogation (23 requêtes SELECT)

| N° | Thème |
|---|---|
| 1 | Tous les monuments |
| 2 | Touristes français |
| 3 | Monuments roumains (nom + ville) |
| 4 | Villes avec monument construit entre 500 et 1500 (`BETWEEN`) |
| 5 | Touristes ayant visité gratuitement cette année |
| 6 | Monuments dont le nom commence par « Tour » avec prix ≥ 20 (`LIKE`) |
| 7 | Monuments triés par ville puis par année DESC (`ORDER BY`) |
| 8 | Monuments sans date de construction (`IS NULL`) |
| 9 | Jointure touriste ↔ visite |
| 10 | Villes avec au moins une visite gratuite |
| 11 | Pays d'origine des touristes ayant visité un monument au Brésil |
| 12 | Touriste dont le nom est identique au nom d'un monument visité |
| 13 | Paires de touristes du même pays (auto-jointure) |
| 14 | Touriste dont le nom coïncide avec un monument de son propre pays |
| 15 | Auto-jointure sur les visites communes |
| 16 | Union des pays de touristes et de monuments (`UNION`) |
| 17 | Touristes ayant visité un monument hors de leur pays d'origine |
| 18 | Villes avec au moins une visite payante > 15 € |
| 19 | Touristes n'ayant visité **aucun** monument de leur pays (`NOT EXISTS`) |
| 20 | Touristes plus jeunes que l'âge d'un monument qu'ils ont visité (`EXISTS`) |
| 21 | Touristes plus jeunes que le monument le plus récent (sous-requête `MIN`) |
| 22 | Touriste(s) le(s) plus jeune(s) |
| 23 | Monuments ayant le prix d'entrée minimum en 2009 |

### Partie 2 — Vues (3 vues)

| Vue | Description |
|---|---|
| `touristeFrancais` | Touristes de France ou d'Italie avec `WITH CHECK OPTION` (INSERT refusé si pays hors périmètre) |
| `sommeEntree_monument` | Total des recettes par monument (`SUM + GROUP BY`) |
| `visite_2025` | Pays d'origine des touristes ayant visité un monument dans l'année courante |

### Partie 3 — Modifications de structure (8 opérations DDL/DML)

| N° | Opération |
|---|---|
| 1 | `INSERT` d'un nouveau monument (Musée de l'Ermitage, Saint-Pétersbourg) |
| 2 | `ALTER TABLE` — ajout d'une colonne `avis VARCHAR(30)` sur `VISITE` |
| 3 | `ALTER TABLE` — modification de `ville` → `VARCHAR(40) NOT NULL` sur `MONUMENT` |
| 4 | `ALTER TABLE DROP COLUMN` — suppression de `dateVisite` |
| 5 | `DELETE` — suppression de tous les touristes |
| 6 | `DELETE` — suppression des monuments français |
| 7 | `UPDATE` — réduction de 10 % sur tous les prix d'entrée |
| 8 | `DROP TABLE` — suppression de la table `TOURISTE` |

### Partie 4 — PL/SQL (1 fonction + 2 procédures)

| Objet | Type | Description |
|---|---|---|
| `gainMonument(p_Monument)` | Fonction | Retourne le total des recettes d'un monument donné (`SUM`) |
| `gainPays` | Procédure | Affiche les recettes totales par pays via un curseur explicite |
| `sedentaire` | Procédure | Indique pour chaque touriste s'il a visité uniquement des monuments de son propre pays |

### Partie 5 — Trigger (1 trigger)

| Trigger | Événement | Description |
|---|---|---|
| `VerifAnneeConstruction` | `BEFORE INSERT OR UPDATE` sur `MONUMENT` | Refuse l'insertion d'un monument avec une année de construction dans le futur (`raise_application_error -20001`) |

### Partie 6 — Plans d'exécution (7 analyses)

Utilisation de `dbms_xplan.display_cursor` pour analyser les plans de 7 requêtes variées : filtre simple, `COUNT`, `IS NULL`, comparaison de date dynamique vs constante entière, combinaison de filtres.

---

## 🛠️ Prérequis et exécution

### Environnement

- **Oracle Database** (Oracle 19c+ recommandé)
- Client SQL : SQL*Plus, SQL Developer, ou [Oracle LiveSQL](https://livesql.oracle.com)

### Exécution

```sql
-- Dans SQL*Plus ou SQL Developer
@script.sql

-- Ou copier-coller les sections souhaitées dans Oracle LiveSQL
```

### Ordre recommandé

1. **Schéma** — les 3 `CREATE TABLE`
2. **Données** — les `INSERT INTO` (touristes, monuments, visites)
3. **Interrogation** — les 23 `SELECT` (lecture seule, sans danger)
4. **Vues** — les 3 `CREATE OR REPLACE VIEW`
5. **PL/SQL** — la fonction et les 2 procédures
6. **Trigger** — le `CREATE OR REPLACE TRIGGER`
7. **Plans d'exécution** — les 7 analyses `dbms_xplan`
8. **Modifications** — uniquement si l'on souhaite tester les DDL/DML destructeurs (DROP, DELETE…)
