---
title: "LEFT JOIN et NULL"
---

## Garder toutes les lignes

### Le problème de l'INNER JOIN

Avec `INNER JOIN`, si une ligne n'a pas de correspondance dans l'autre table, elle **disparaît** du résultat. Parfois, on veut garder toutes les lignes d'une table, même celles sans correspondance.

### LEFT JOIN

`LEFT JOIN` garde **toutes les lignes de la table de gauche** (celle après `FROM`), même si elles n'ont pas de correspondance dans la table de droite (celle après `JOIN`).

```sql
-- Tous les livres, avec leur emprunt s'il existe
SELECT livres.titre, emprunts.emprunteur, emprunts.date_emprunt
FROM livres
LEFT JOIN emprunts ON livres.id = emprunts.livre_id;
```

Les livres qui n'ont jamais été empruntés apparaissent quand même, avec `NULL` dans les colonnes d'emprunt.

| titre | emprunteur | date_emprunt |
|-------|-----------|--------------|
| Les Misérables | Alice Martin | 2024-01-10 |
| Notre-Dame de Paris | Claire Petit | 2024-03-15 |
| Harry Potter à l'école des sorciers | Bob Dupont | 2024-01-15 |
| Harry Potter à l'école des sorciers | Alice Martin | 2024-02-20 |
| La Peste | NULL | NULL |
| 1Q84 | NULL | NULL |
| ... | ... | ... |

### Comprendre NULL

`NULL` signifie "pas de valeur", "inconnu". Ce n'est pas zéro, ce n'est pas une chaîne vide — c'est l'**absence** de donnée.

Dans notre base d'emprunts, `date_retour` est `NULL` quand un livre n'a pas encore été rendu.

### Tester NULL : IS NULL et IS NOT NULL

On ne peut **pas** utiliser `=` pour tester NULL. Il faut utiliser `IS NULL` ou `IS NOT NULL` :

```sql
-- ❌ Ne fonctionne pas
-- SELECT * FROM emprunts WHERE date_retour = NULL;

-- ✓ Correct
SELECT * FROM emprunts WHERE date_retour IS NULL;
```

### Trouver les livres jamais empruntés

C'est un cas d'usage classique du `LEFT JOIN` + `IS NULL` :

```sql
SELECT livres.titre
FROM livres
LEFT JOIN emprunts ON livres.id = emprunts.livre_id
WHERE emprunts.id IS NULL;
```

On fait un LEFT JOIN, puis on filtre les lignes où la jointure n'a rien trouvé (les colonnes d'emprunt sont NULL).

### Trouver les emprunts en cours

```sql
-- Livres actuellement empruntés (pas encore rendus)
SELECT livres.titre, emprunts.emprunteur, emprunts.date_emprunt
FROM emprunts
INNER JOIN livres ON emprunts.livre_id = livres.id
WHERE emprunts.date_retour IS NULL;
```

### INNER JOIN vs LEFT JOIN : résumé

| | INNER JOIN | LEFT JOIN |
|---|-----------|-----------|
| Lignes sans correspondance | Exclues | Gardées (avec NULL) |
| Cas d'usage | Quand on veut uniquement les correspondances | Quand on veut tout garder à gauche |

### Exercice

1. Affiche tous les auteurs avec le nombre de livres qu'ils ont écrits (même les auteurs sans livre)
2. Quels livres n'ont jamais été empruntés ?
3. Affiche tous les emprunts en cours (date_retour IS NULL) avec le titre du livre
4. Affiche tous les livres avec le nombre de fois qu'ils ont été empruntés (0 si jamais empruntés)
5. Quels emprunteurs ont des livres non rendus ?

<details>
<summary>Voir les réponses</summary>

```sql
-- 1. Auteurs avec nombre de livres
SELECT auteurs.nom, auteurs.prenom, COUNT(livres.id)
FROM auteurs
LEFT JOIN livres ON auteurs.id = livres.auteur_id
GROUP BY auteurs.id, auteurs.nom, auteurs.prenom;

-- 2. Livres jamais empruntés
SELECT livres.titre
FROM livres
LEFT JOIN emprunts ON livres.id = emprunts.livre_id
WHERE emprunts.id IS NULL;

-- 3. Emprunts en cours
SELECT livres.titre, emprunts.emprunteur, emprunts.date_emprunt
FROM emprunts
INNER JOIN livres ON emprunts.livre_id = livres.id
WHERE emprunts.date_retour IS NULL;

-- 4. Livres avec nombre d'emprunts
SELECT livres.titre, COUNT(emprunts.id)
FROM livres
LEFT JOIN emprunts ON livres.id = emprunts.livre_id
GROUP BY livres.id, livres.titre
ORDER BY COUNT(emprunts.id) DESC;

-- 5. Emprunteurs avec livres non rendus
SELECT DISTINCT emprunts.emprunteur
FROM emprunts
WHERE emprunts.date_retour IS NULL;
```

</details>
