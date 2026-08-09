---
title: "INNER JOIN"
---

## Croiser deux tables

Jusqu'ici, nos requêtes ne touchaient qu'une seule table. Mais les données sont souvent réparties dans **plusieurs tables liées entre elles**. `JOIN` permet de les relier.

### L'analogie des feuilles Excel

Imagine deux feuilles Excel :
- **Feuille 1** : les livres (avec un numéro d'auteur)
- **Feuille 2** : les auteurs (avec leurs noms)

Pour voir le nom de l'auteur à côté de chaque livre, tu devrais faire un "RECHERCHEV" entre les deux feuilles. En SQL, c'est exactement ce que fait `JOIN`.

### Le lien entre les tables : la clé étrangère

Dans notre base, la table `livres` contient une colonne `auteur_id` qui fait référence à la colonne `id` de la table `auteurs`. C'est une **clé étrangère**.

```
livres.auteur_id  →  auteurs.id
```

### Syntaxe de l'INNER JOIN

```sql
SELECT livres.titre, auteurs.prenom, auteurs.nom
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id;
```

| titre | prenom | nom |
|-------|--------|-----|
| Les Misérables | Victor | Hugo |
| Notre-Dame de Paris | Victor | Hugo |
| Harry Potter à l'école des sorciers | J.K. | Rowling |
| ... | ... | ... |

### Comment ça marche ?

1. SQL prend chaque ligne de `livres`
2. Il cherche dans `auteurs` la ligne où `auteurs.id` = `livres.auteur_id`
3. Il **fusionne** les deux lignes en une seule

Si aucune correspondance n'est trouvée, la ligne est **exclue** du résultat (c'est la spécificité de `INNER JOIN`).

### Une conséquence à ne pas manquer

Notre table `auteurs` contient **6** autrices et auteurs, mais Marguerite Duras
n'a aucun livre au catalogue. Comptons les lignes du join :

```sql
SELECT COUNT(*)
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id;
```

Résultat : **12**, soit le nombre de livres — et non 13. Marguerite Duras a
purement et simplement disparu du résultat, parce qu'aucune ligne de `livres` ne
la mentionne.

Ce n'est ni un bug ni un oubli : c'est la définition de `INNER JOIN`, qui ne
garde que les lignes **appariées des deux côtés**. Mais cela veut dire qu'une
requête comme celle-ci est fausse :

```sql
-- ❌ Ne compte pas les auteurs, mais les auteurs QUI ONT AU MOINS UN LIVRE
SELECT COUNT(DISTINCT auteurs.id)
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id;
```

Elle renvoie 5 au lieu de 6. Pour compter les auteurs, il faut interroger la
table `auteurs` seule (`SELECT COUNT(*) FROM auteurs`), ou utiliser le
`LEFT JOIN` de l'étape suivante.

### Préciser la table d'origine

Quand deux tables ont des colonnes qui portent le même nom (comme `id`), il faut préciser de quelle table on parle :

```sql
-- ❌ Ambigu : quelle colonne "id" ?
-- SELECT id, titre FROM livres INNER JOIN auteurs ON ...

-- ✓ Précis
SELECT livres.id, livres.titre, auteurs.nom
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id;
```

### Combiner JOIN avec WHERE, ORDER BY, etc.

```sql
-- Livres d'auteurs français, triés par titre
SELECT livres.titre, auteurs.prenom, auteurs.nom
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id
WHERE auteurs.nationalite = 'Française'
ORDER BY livres.titre;
```

### JOIN avec agrégation

```sql
-- Nombre de livres par auteur (avec le nom !)
SELECT auteurs.nom, auteurs.prenom, COUNT(*)
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id
GROUP BY auteurs.id, auteurs.nom, auteurs.prenom;
```

### Exercice

Le nombre de lignes attendu est indiqué entre parenthèses.

1. Affiche le titre de chaque livre avec le prénom et le nom de son auteur *(12 lignes)*
2. Affiche les livres d'auteurs britanniques *(5 lignes)*
3. Affiche le nombre de livres par nationalité d'auteur *(3 lignes — et non 3 nationalités × tous les auteurs : réfléchis à qui manque)*
4. Affiche le titre du livre et le nom de l'emprunteur pour chaque emprunt (JOIN entre `emprunts` et `livres`) *(8 lignes)*
5. Quel auteur a écrit le plus de pages au total ? Affiche son nom et le total *(1 ligne)*

<details markdown="1">
<summary>Voir les réponses</summary>

```sql
-- 1. Titre avec auteur
SELECT livres.titre, auteurs.prenom, auteurs.nom
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id;

-- 2. Livres d'auteurs britanniques
SELECT livres.titre, auteurs.nom
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id
WHERE auteurs.nationalite = 'Britannique';

-- 3. Livres par nationalité
SELECT auteurs.nationalite, COUNT(*)
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id
GROUP BY auteurs.nationalite;

-- 4. Emprunts avec titre du livre
SELECT livres.titre, emprunts.emprunteur, emprunts.date_emprunt
FROM emprunts
INNER JOIN livres ON emprunts.livre_id = livres.id;

-- 5. Auteur avec le plus de pages
SELECT auteurs.nom, auteurs.prenom, SUM(livres.pages) AS total_pages
FROM livres
INNER JOIN auteurs ON livres.auteur_id = auteurs.id
GROUP BY auteurs.id, auteurs.nom, auteurs.prenom
ORDER BY total_pages DESC
LIMIT 1;
```

</details>
