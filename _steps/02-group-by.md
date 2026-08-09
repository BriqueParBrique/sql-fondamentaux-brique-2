---
title: "GROUP BY"
---

## Regrouper les données

`GROUP BY` permet de **regrouper les lignes** par catégorie avant d'appliquer une fonction d'agrégation. C'est la différence entre "combien de livres au total ?" et "combien de livres **par genre** ?".

### Le concept de regroupement

Sans `GROUP BY` :

```sql
-- Combien de livres au total ?
SELECT COUNT(*) FROM livres;
-- Résultat : 12
```

Avec `GROUP BY` :

```sql
-- Combien de livres par genre ?
SELECT genre, COUNT(*) FROM livres
GROUP BY genre;
```

| genre | COUNT(*) |
|-------|----------|
| Fantasy | 3 |
| Philosophie | 1 |
| Policier | 2 |
| Roman | 6 |

### La règle d'or

Quand on utilise `GROUP BY`, chaque colonne dans le `SELECT` doit être soit :
- La colonne de regroupement (celle dans `GROUP BY`)
- Une fonction d'agrégation (`COUNT`, `SUM`, `AVG`, etc.)

```sql
-- Correct ✓
SELECT genre, COUNT(*) FROM livres GROUP BY genre;

-- Incorrect ✗ — "titre" n'est ni un regroupement ni une agrégation
-- SELECT genre, titre, COUNT(*) FROM livres GROUP BY genre;
```

### Exemples pratiques

Prix moyen par genre :

```sql
SELECT genre, AVG(prix) FROM livres
GROUP BY genre;
```

Nombre de livres par auteur (par auteur_id) :

```sql
SELECT auteur_id, COUNT(*) FROM livres
GROUP BY auteur_id;
```

Pages totales par genre :

```sql
SELECT genre, SUM(pages) FROM livres
GROUP BY genre;
```

### Combiner GROUP BY avec WHERE

`WHERE` filtre les lignes **avant** le regroupement :

```sql
-- Nombre de livres par genre, seulement pour les livres publiés après 1950
SELECT genre, COUNT(*) FROM livres
WHERE annee_publication > 1950
GROUP BY genre;
```

### Trier les groupes

On peut utiliser `ORDER BY` sur le résultat du regroupement :

```sql
-- Genres triés par nombre de livres (décroissant)
SELECT genre, COUNT(*) FROM livres
GROUP BY genre
ORDER BY COUNT(*) DESC;
```

### Exercice

Le nombre de lignes attendu est indiqué entre parenthèses.

1. Affiche le nombre de livres par auteur, identifié par son `auteur_id`
   *(5 lignes — on croisera avec les noms à l'étape 4, avec les JOIN)*
2. Affiche le prix minimum et maximum par genre *(4 lignes)*
3. Affiche le nombre de pages moyen par genre, trié du plus grand au plus petit *(4 lignes)*
4. Combien d'emprunts a fait chaque emprunteur ? *(5 lignes)*
5. Affiche le nombre de livres publiés par décennie (indice : utilise `annee_publication / 10 * 10`) *(7 lignes)*

<details markdown="1">
<summary>Voir les réponses</summary>

```sql
-- 1. Nombre de livres par auteur_id.
--    Cinq lignes seulement, alors que la table `auteurs` en compte six :
--    Marguerite Duras n'a aucun livre, donc aucune ligne de `livres` ne la
--    mentionne, donc aucun groupe ne lui correspond. On corrigera ça à
--    l'étape 5 avec LEFT JOIN.
SELECT auteur_id, COUNT(*) FROM livres
GROUP BY auteur_id;

-- 2. Prix min et max par genre
SELECT genre, MIN(prix), MAX(prix) FROM livres
GROUP BY genre;

-- 3. Pages moyennes par genre
SELECT genre, AVG(pages) FROM livres
GROUP BY genre
ORDER BY AVG(pages) DESC;

-- 4. Emprunts par emprunteur
SELECT emprunteur, COUNT(*) FROM emprunts
GROUP BY emprunteur;

-- 5. Livres par décennie
SELECT (annee_publication / 10 * 10) AS decennie, COUNT(*) FROM livres
GROUP BY annee_publication / 10 * 10
ORDER BY decennie;
```

</details>
