---
title: "Les alias (AS)"
---

## Renommer pour la lisibilité

Les alias permettent de donner des **noms temporaires** aux colonnes et aux tables dans une requête. Cela rend les résultats plus clairs et le code plus court.

### Alias de colonnes

Par défaut, le nom de colonne dans le résultat reprend exactement ce que tu as écrit dans le `SELECT`. Avec des fonctions d'agrégation, ça peut être peu lisible :

```sql
-- Sans alias : le résultat affiche "COUNT(*)" comme nom de colonne
SELECT genre, COUNT(*) FROM livres GROUP BY genre;

-- Avec alias : bien plus clair !
SELECT genre, COUNT(*) AS nombre_de_livres FROM livres GROUP BY genre;
```

| genre | nombre_de_livres |
|-------|-----------------|
| Fantasy | 3 |
| Policier | 2 |
| Roman | 6 |

### Exemples d'alias de colonnes

```sql
SELECT
    AVG(prix) AS prix_moyen,
    MIN(prix) AS prix_minimum,
    MAX(prix) AS prix_maximum
FROM livres;
```

```sql
SELECT
    genre,
    COUNT(*) AS total,
    AVG(pages) AS pages_moyennes
FROM livres
GROUP BY genre;
```

### Alias de tables

Quand on écrit des `JOIN`, les noms de tables rendent les requêtes longues. Les alias de tables les raccourcissent :

```sql
-- Sans alias : verbeux
SELECT livres.titre, auteurs.prenom, auteurs.nom
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id;

-- Avec alias : compact
SELECT l.titre, a.prenom, a.nom
FROM livres AS l
INNER JOIN auteurs AS a ON l.auteur_id = a.id;
```

### Le mot AS est optionnel

En SQL, le mot `AS` est optionnel pour les alias. Ces deux écritures sont équivalentes :

```sql
-- Avec AS
SELECT titre AS nom_du_livre FROM livres;

-- Sans AS (fonctionne aussi)
SELECT titre nom_du_livre FROM livres;
```

La convention recommandée est d'utiliser `AS` pour la clarté.

### Alias dans ORDER BY

On peut utiliser un alias de colonne dans `ORDER BY` :

```sql
SELECT genre, COUNT(*) AS total
FROM livres
GROUP BY genre
ORDER BY total DESC;
```

### Exemple complet avec alias

```sql
SELECT
    a.nom AS auteur,
    COUNT(l.id) AS livres_ecrits,
    AVG(l.pages) AS pages_moyennes,
    SUM(l.prix) AS valeur_totale
FROM livres AS l
INNER JOIN auteurs AS a ON l.auteur_id = a.id
GROUP BY a.id, a.nom
ORDER BY livres_ecrits DESC;
```

### Exercice

1. Affiche le nombre total de livres avec l'alias `total_livres`
2. Affiche chaque genre avec son prix moyen (alias `prix_moyen`) et son nombre de livres (alias `nb_livres`)
3. Réécris cette requête avec des alias de tables :
   `SELECT livres.titre, auteurs.nom FROM livres INNER JOIN auteurs ON livres.auteur_id = auteurs.id WHERE auteurs.nationalite = 'Française';`
4. Affiche les emprunteurs avec le nombre d'emprunts (alias `nb_emprunts`), triés par `nb_emprunts` décroissant
5. Affiche le titre du livre, le nom de l'auteur et le nombre d'emprunts, en utilisant des alias de tables et de colonnes

<details>
<summary>Voir les réponses</summary>

```sql
-- 1. Total avec alias
SELECT COUNT(*) AS total_livres FROM livres;

-- 2. Genre avec prix moyen et nombre
SELECT genre, AVG(prix) AS prix_moyen, COUNT(*) AS nb_livres
FROM livres
GROUP BY genre;

-- 3. Requête avec alias de tables
SELECT l.titre, a.nom
FROM livres AS l
INNER JOIN auteurs AS a ON l.auteur_id = a.id
WHERE a.nationalite = 'Française';

-- 4. Emprunteurs avec nombre d'emprunts
SELECT emprunteur, COUNT(*) AS nb_emprunts
FROM emprunts
GROUP BY emprunteur
ORDER BY nb_emprunts DESC;

-- 5. Livres, auteurs et emprunts avec alias
SELECT l.titre, a.nom AS auteur, COUNT(e.id) AS nb_emprunts
FROM livres AS l
INNER JOIN auteurs AS a ON l.auteur_id = a.id
LEFT JOIN emprunts AS e ON l.id = e.livre_id
GROUP BY l.id, l.titre, a.nom
ORDER BY nb_emprunts DESC;
```

</details>
