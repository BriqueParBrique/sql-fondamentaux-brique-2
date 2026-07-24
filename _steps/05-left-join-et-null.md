---
title: "LEFT JOIN et NULL"
---

## Garder toutes les lignes

### Le problème de l'INNER JOIN

Avec `INNER JOIN`, si une ligne n'a pas de correspondance dans l'autre table, elle **disparaît** du résultat. Parfois, on veut garder toutes les lignes d'une table, même celles sans correspondance.

### LEFT JOIN

`LEFT JOIN` garde **toutes les lignes de la table de gauche** (celle après `FROM`), même si elles n'ont pas de correspondance dans la table de droite (celle après `JOIN`).

### Le cas de Marguerite Duras

Souviens-toi de l'étape précédente : avec `INNER JOIN`, Marguerite Duras
disparaissait du résultat parce qu'elle n'a aucun livre au catalogue. Comparons
les deux jointures sur le même exemple.

```sql
-- INNER JOIN : 12 lignes, une par livre. Marguerite Duras est absente.
SELECT auteurs.nom, livres.titre
FROM auteurs
INNER JOIN livres ON auteurs.id = livres.auteur_id;
```

```sql
-- LEFT JOIN : 13 lignes. La 13ᵉ est celle de Marguerite Duras, avec NULL
-- comme titre.
SELECT auteurs.nom, livres.titre
FROM auteurs
LEFT JOIN livres ON auteurs.id = livres.auteur_id;
```

| nom      | titre               |
| -------- | ------------------- |
| Hugo     | Les Misérables      |
| Hugo     | Notre-Dame de Paris |
| …        | …                   |
| Christie | Dix Petits Nègres   |
| Duras    | NULL                |

C'est toute la différence : `LEFT JOIN` conserve la ligne de gauche même sans
correspondance, et remplit les colonnes de droite avec `NULL`.

### Un autre exemple : les livres jamais empruntés

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

### Compter après un LEFT JOIN : `COUNT(*)` est un piège

C'est le moment où la distinction vue à l'étape 1 devient indispensable.
Comptons les livres de chaque auteur :

```sql
-- ❌ Faux : Marguerite Duras est créditée d'un livre
SELECT auteurs.nom, COUNT(*)
FROM auteurs
LEFT JOIN livres ON auteurs.id = livres.auteur_id
GROUP BY auteurs.id, auteurs.nom;
```

| nom      | COUNT(\*) |
| -------- | --------- |
| …        | …         |
| Duras    | **1**     |

Pourquoi 1 ? Parce que le `LEFT JOIN` a bien produit **une ligne** pour
Marguerite Duras — une ligne dont les colonnes de `livres` sont vides. Et
`COUNT(*)` compte les lignes, pas les livres.

La correction consiste à compter une colonne qui vient de la table de droite :

```sql
-- ✓ Correct : Marguerite Duras est créditée de 0 livre
SELECT auteurs.nom, COUNT(livres.id)
FROM auteurs
LEFT JOIN livres ON auteurs.id = livres.auteur_id
GROUP BY auteurs.id, auteurs.nom;
```

`COUNT(livres.id)` ignore les valeurs vides : sur la ligne de Marguerite Duras,
`livres.id` est `NULL`, donc rien n'est compté, et on obtient bien **0**.

> **La règle** : après un `LEFT JOIN`, compte toujours une colonne de la table de
> droite (`COUNT(livres.id)`), jamais `COUNT(*)`.

### INNER JOIN vs LEFT JOIN : résumé

| | INNER JOIN | LEFT JOIN |
|---|-----------|-----------|
| Lignes sans correspondance | Exclues | Gardées (avec NULL) |
| Cas d'usage | Quand on veut uniquement les correspondances | Quand on veut tout garder à gauche |

### Exercice

Le nombre de lignes attendu est indiqué entre parenthèses.

1. Affiche tous les auteurs avec le nombre de livres qu'ils ont écrits, **y
   compris ceux qui n'en ont aucun** *(6 lignes, dont une à 0)*
2. Quels livres n'ont jamais été empruntés ? *(5 lignes)*
3. Affiche tous les emprunts en cours (date_retour IS NULL) avec le titre du livre *(3 lignes)*
4. Affiche tous les livres avec le nombre de fois qu'ils ont été empruntés (0 si jamais empruntés) *(12 lignes)*
5. Quels emprunteurs ont des livres non rendus ? *(3 lignes)*

<details>
<summary>Voir les réponses</summary>

```sql
-- 1. Auteurs avec nombre de livres.
--    LEFT JOIN pour garder Marguerite Duras, et COUNT(livres.id) — pas
--    COUNT(*) — pour qu'elle soit bien créditée de 0 et non de 1.
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
