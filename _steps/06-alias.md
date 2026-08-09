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

## Boîte à outils : trois fonctions pour présenter un résultat

Les alias rendent les colonnes lisibles. Voici trois outils complémentaires pour
rendre les **valeurs** lisibles. Tu en auras besoin dès le mini-projet.

### `ROUND` — arrondir les décimales

`AVG` produit volontiers des résultats comme `10.9533333333333`. `ROUND` coupe ça
au nombre de décimales voulu :

```sql
SELECT
    genre,
    AVG(prix) AS prix_brut,
    ROUND(AVG(prix), 2) AS prix_arrondi
FROM livres
GROUP BY genre;
```

| genre       | prix_brut         | prix_arrondi |
| ----------- | ----------------- | ------------ |
| Fantasy     | 9.23333333333333  | 9.23         |
| Philosophie | 6.9               | 6.9          |
| Policier    | 7.5               | 7.5          |
| Roman       | 10.95             | 10.95        |

Le second argument est le nombre de décimales : `ROUND(x, 2)` en garde deux,
`ROUND(x)` arrondit à l'entier.

### `||` — coller deux textes

L'opérateur `||` (deux barres verticales) concatène des chaînes. Pratique pour
recomposer un nom complet à partir de deux colonnes :

```sql
SELECT prenom || ' ' || nom AS auteur
FROM auteurs;
```

| auteur             |
| ------------------ |
| Victor Hugo        |
| J.K. Rowling       |
| …                  |

Note bien le `' '` au milieu : sans lui, on obtiendrait `VictorHugo`.

### `CASE WHEN` — un « si… alors » dans une requête

`CASE WHEN` permet de choisir une valeur selon une condition. Sa forme la plus
utile est le **comptage conditionnel** : compter seulement les lignes qui
remplissent un critère, à l'intérieur d'un `SUM`.

```sql
SELECT
    emprunteur,
    COUNT(*) AS nb_emprunts,
    SUM(CASE WHEN date_retour IS NULL THEN 1 ELSE 0 END) AS non_rendus
FROM emprunts
GROUP BY emprunteur;
```

Lis la ligne `SUM(...)` de l'intérieur vers l'extérieur :

1. Pour chaque emprunt, `CASE WHEN date_retour IS NULL THEN 1 ELSE 0 END` vaut
   `1` si le livre n'est pas rendu, `0` sinon.
2. `SUM` additionne ces 1 et ces 0 par groupe.
3. Le total est donc le **nombre de livres non rendus** de cet emprunteur.

| emprunteur   | nb_emprunts | non_rendus |
| ------------ | ----------- | ---------- |
| Alice Martin | 2           | 1          |
| Bob Dupont   | 2           | 1          |
| Claire Petit | 2           | 0          |
| David Moreau | 1           | 0          |
| Emma Leroy   | 1           | 1          |

C'est l'astuce à retenir : `SUM(CASE WHEN condition THEN 1 ELSE 0 END)` compte
les lignes qui vérifient une condition, **à côté** d'un `COUNT(*)` qui les compte
toutes. Un `WHERE` ne permettrait pas d'avoir les deux dans la même requête.

### Exercice

Le nombre de lignes attendu est indiqué entre parenthèses.

1. Affiche le nombre total de livres avec l'alias `total_livres` *(1 ligne)*
2. Affiche chaque genre avec son prix moyen arrondi à 2 décimales (alias `prix_moyen`) et son nombre de livres (alias `nb_livres`) *(4 lignes)*
3. Réécris cette requête avec des alias de tables *(4 lignes)* :
   `SELECT livres.titre, auteurs.nom FROM livres INNER JOIN auteurs ON livres.auteur_id = auteurs.id WHERE auteurs.nationalite = 'Française';`
4. Affiche les emprunteurs avec le nombre d'emprunts (alias `nb_emprunts`), triés par `nb_emprunts` décroissant *(5 lignes)*
5. Affiche le titre du livre, le nom de l'auteur et le nombre d'emprunts, en utilisant des alias de tables et de colonnes *(12 lignes)*
6. Affiche le nom complet de chaque auteur sur une seule colonne nommée `auteur`, au format « Prénom Nom » *(6 lignes)*
7. Pour chaque genre, affiche le nombre de livres et le nombre de livres coûtant plus de 9 € *(4 lignes — indice : `SUM(CASE WHEN …)`)*

<details markdown="1">
<summary>Voir les réponses</summary>

```sql
-- 1. Total avec alias
SELECT COUNT(*) AS total_livres FROM livres;

-- 2. Genre avec prix moyen arrondi et nombre
SELECT genre, ROUND(AVG(prix), 2) AS prix_moyen, COUNT(*) AS nb_livres
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

-- 6. Nom complet sur une colonne
SELECT prenom || ' ' || nom AS auteur
FROM auteurs;

-- 7. Comptage conditionnel par genre
SELECT
    genre,
    COUNT(*) AS nb_livres,
    SUM(CASE WHEN prix > 9 THEN 1 ELSE 0 END) AS nb_livres_chers
FROM livres
GROUP BY genre;
```

</details>
