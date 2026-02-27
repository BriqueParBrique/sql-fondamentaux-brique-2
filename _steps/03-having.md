---
title: "HAVING"
---

## Filtrer les groupes

`HAVING` permet de filtrer **après** le regroupement. C'est le complément de `WHERE` qui filtre **avant**.

### WHERE vs HAVING

| | WHERE | HAVING |
|---|-------|--------|
| **Quand ?** | Avant le regroupement | Après le regroupement |
| **Filtre quoi ?** | Des lignes individuelles | Des groupes |
| **Peut utiliser des agrégations ?** | Non | Oui |

### Exemple concret

Imagine qu'on veut les genres qui ont **plus de 2 livres** :

```sql
-- On ne peut PAS écrire :
-- SELECT genre, COUNT(*) FROM livres WHERE COUNT(*) > 2 GROUP BY genre;
-- ❌ Erreur ! WHERE ne connaît pas COUNT(*)

-- On utilise HAVING :
SELECT genre, COUNT(*) FROM livres
GROUP BY genre
HAVING COUNT(*) > 2;
```

| genre | COUNT(*) |
|-------|----------|
| Fantasy | 3 |
| Roman | 6 |

### Combiner WHERE et HAVING

On peut utiliser les deux dans la même requête :

```sql
-- Genres avec plus de 1 livre, parmi les livres publiés après 1950
SELECT genre, COUNT(*) FROM livres
WHERE annee_publication > 1950
GROUP BY genre
HAVING COUNT(*) > 1;
```

L'ordre d'exécution est :
1. `WHERE` filtre les lignes
2. `GROUP BY` regroupe ce qui reste
3. `HAVING` filtre les groupes

### Autres exemples

Auteurs ayant écrit plus de 2 livres :

```sql
SELECT auteur_id, COUNT(*) FROM livres
GROUP BY auteur_id
HAVING COUNT(*) > 2;
```

Genres dont le prix moyen dépasse 9€ :

```sql
SELECT genre, AVG(prix) FROM livres
GROUP BY genre
HAVING AVG(prix) > 9;
```

### L'ordre complet des clauses

```sql
SELECT colonnes
FROM table
WHERE conditions_sur_les_lignes
GROUP BY colonnes_de_regroupement
HAVING conditions_sur_les_groupes
ORDER BY colonnes_de_tri
LIMIT nombre;
```

### Exercice

1. Affiche les genres qui contiennent exactement 2 livres
2. Affiche les auteurs (auteur_id) qui ont écrit des livres totalisant plus de 500 pages
3. Affiche les emprunteurs qui ont emprunté plus d'un livre
4. Affiche les genres dont le prix moyen est inférieur à 9€, pour les livres publiés après 1900
5. Affiche les genres ayant au moins 2 livres, triés par prix moyen décroissant

<details>
<summary>Voir les réponses</summary>

```sql
-- 1. Genres avec exactement 2 livres
SELECT genre, COUNT(*) FROM livres
GROUP BY genre
HAVING COUNT(*) = 2;

-- 2. Auteurs avec plus de 500 pages au total
SELECT auteur_id, SUM(pages) FROM livres
GROUP BY auteur_id
HAVING SUM(pages) > 500;

-- 3. Emprunteurs avec plus d'un emprunt
SELECT emprunteur, COUNT(*) FROM emprunts
GROUP BY emprunteur
HAVING COUNT(*) > 1;

-- 4. Genres avec prix moyen < 9€, livres après 1900
SELECT genre, AVG(prix) FROM livres
WHERE annee_publication > 1900
GROUP BY genre
HAVING AVG(prix) < 9;

-- 5. Genres avec au moins 2 livres, triés par prix moyen
SELECT genre, COUNT(*), AVG(prix) FROM livres
GROUP BY genre
HAVING COUNT(*) >= 2
ORDER BY AVG(prix) DESC;
```

</details>
