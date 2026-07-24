# SQL Fondamentaux — Niveau 2

> Apprendre à transformer et combiner les données avec SQL — [BriqueParBrique](https://briqueparbrique.github.io)

Ce dépôt contient le parcours d'apprentissage « SQL Fondamentaux — Niveau 2 »,
conçu comme un mini-onboarding en sept étapes pour passer de la simple lecture de
données à leur analyse : agréger, regrouper, et croiser plusieurs tables.

**Site** : [briqueparbrique.github.io/sql-fondamentaux-brique-2](https://briqueparbrique.github.io/sql-fondamentaux-brique-2/)

**Prérequis** : le [niveau 1](https://briqueparbrique.github.io/sql-fondamentaux-brique-1/)
(`SELECT`, `WHERE`, `ORDER BY`, `LIMIT`).

## Parcours

1. Fonctions d'agrégation
2. GROUP BY
3. HAVING
4. INNER JOIN
5. LEFT JOIN et NULL
6. Les alias (AS)
7. Mini-projet

Le fil rouge est une **base de bibliothèque** (`auteurs`, `livres`, `emprunts`),
utilisée du début à la fin du niveau.

À la fin, on sait agréger avec `COUNT`/`SUM`/`AVG`/`MIN`/`MAX`, regrouper avec
`GROUP BY`, filtrer des groupes avec `HAVING`, croiser des tables avec
`INNER JOIN` et `LEFT JOIN`, gérer les valeurs `NULL`, et présenter un résultat
avec des alias, `ROUND`, `||` et `SUM(CASE WHEN …)`.

## Niveaux voisins

- Précédent : [Niveau 1](https://briqueparbrique.github.io/sql-fondamentaux-brique-1/) — lire les données
- Suivant : [Niveau 3](https://briqueparbrique.github.io/sql-fondamentaux-brique-3/) — écrire et structurer une base

## Contribuer

Les contributions sont les bienvenues ! Consultez [CONTRIBUTING.md](CONTRIBUTING.md) pour savoir comment améliorer ce parcours.

## Développement

Le site utilise [Jekyll](https://jekyllrb.com/) et est hébergé sur GitHub Pages. Consultez [DEVELOPER.md](DEVELOPER.md) pour mettre en place l'environnement de développement local.

## Licence

Ce projet est sous licence [MIT](LICENSE).
