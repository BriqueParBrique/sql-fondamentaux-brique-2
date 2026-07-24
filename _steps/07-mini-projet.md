---
title: "Mini-projet"
---

## Mini-projet : Tableau de bord de la bibliothèque

Félicitations, tu maîtrises maintenant les agrégations, les regroupements et les jointures ! Mettons tout ça en pratique avec un tableau de bord complet.

### Le contexte

Tu es responsable d'une petite bibliothèque. Tu dois créer un **tableau de bord** pour répondre aux questions de ta direction. Tu travailles avec les trois tables créées au début de ce niveau (`auteurs`, `livres`, `emprunts`).

Si tu as besoin de recréer la base, retourne à l'étape 1 pour copier le code SQL de création.

### Les défis

Le nombre de lignes attendu est indiqué entre parenthèses.

**Défi 1 — Vue d'ensemble** : Affiche le nombre total de livres, le prix moyen et
le nombre total de pages, avec des alias clairs. Affiche **séparément** le nombre
d'auteurs. *(2 requêtes, 1 ligne chacune.)*

> Pourquoi deux requêtes ? Parce que « combien de livres ? » et « combien
> d'auteurs ? » portent sur deux tables différentes. Vouloir tout mettre dans une
> seule requête obligerait à joindre `livres` et `auteurs`, et la jointure
> écarterait Marguerite Duras — on compterait 5 auteurs au lieu de 6. Deux
> requêtes simples valent mieux qu'une requête fausse.

**Défi 2 — Catalogue par genre** : Pour chaque genre, affiche le nombre de livres, le prix moyen et le nombre de pages moyen (arrondis). Trie par nombre de livres décroissant. *(4 lignes)*

**Défi 3 — Top auteurs** : Affiche le nom complet de chaque auteur avec le nombre de livres écrits et le nombre total de pages. Ne garde que les auteurs ayant écrit plus de 2 livres. *(2 lignes)*

**Défi 4 — Livres populaires** : Quels livres ont été empruntés plus d'une fois ? Affiche le titre et le nombre d'emprunts. *(1 ligne)*

**Défi 5 — Livres oubliés** : Quels livres n'ont jamais été empruntés ? Affiche leur titre et le nom de leur auteur. *(5 lignes)*

**Défi 6 — Lecteurs actifs** : Affiche les emprunteurs qui ont au moins 2 emprunts, avec le nombre de livres empruntés et le nombre de livres non encore rendus. *(3 lignes)*

**Défi 7 — Rapport par nationalité** : Pour chaque nationalité, affiche le nombre
d'auteurs, le nombre total de livres et le prix moyen des livres. Trie par nombre
de livres décroissant. **Toutes les nationalités doivent apparaître, et chaque
auteur doit être compté même s'il n'a aucun livre.** *(3 lignes)*

### Solutions

<details>
<summary>Voir les solutions</summary>

```sql
-- Défi 1 : Vue d'ensemble — deux requêtes, chacune sur sa table.
SELECT
    COUNT(*) AS total_livres,
    ROUND(AVG(prix), 2) AS prix_moyen,
    SUM(pages) AS total_pages
FROM livres;

SELECT COUNT(*) AS total_auteurs FROM auteurs;   -- 6

-- Défi 2 : Catalogue par genre
SELECT
    genre,
    COUNT(*) AS nb_livres,
    ROUND(AVG(prix), 2) AS prix_moyen,
    ROUND(AVG(pages)) AS pages_moyennes
FROM livres
GROUP BY genre
ORDER BY nb_livres DESC;

-- Défi 3 : Top auteurs
SELECT
    a.prenom || ' ' || a.nom AS auteur,
    COUNT(l.id) AS nb_livres,
    SUM(l.pages) AS total_pages
FROM auteurs AS a
INNER JOIN livres AS l ON a.id = l.auteur_id
GROUP BY a.id, a.nom, a.prenom
HAVING COUNT(l.id) > 2;

-- Défi 4 : Livres populaires
SELECT
    l.titre,
    COUNT(e.id) AS nb_emprunts
FROM livres AS l
INNER JOIN emprunts AS e ON l.id = e.livre_id
GROUP BY l.id, l.titre
HAVING COUNT(e.id) > 1;

-- Défi 5 : Livres jamais empruntés
SELECT l.titre, a.prenom || ' ' || a.nom AS auteur
FROM livres AS l
INNER JOIN auteurs AS a ON l.auteur_id = a.id
LEFT JOIN emprunts AS e ON l.id = e.livre_id
WHERE e.id IS NULL;

-- Défi 6 : Lecteurs actifs
SELECT
    e.emprunteur,
    COUNT(e.id) AS nb_emprunts,
    SUM(CASE WHEN e.date_retour IS NULL THEN 1 ELSE 0 END) AS non_rendus
FROM emprunts AS e
GROUP BY e.emprunteur
HAVING COUNT(e.id) >= 2;

-- Défi 7 : Rapport par nationalité.
--    LEFT JOIN (et non INNER JOIN) pour que Marguerite Duras compte dans les
--    auteurs de nationalité française : avec INNER JOIN on obtiendrait 1 seule
--    autrice française au lieu de 3.
--    COUNT(l.id) donne bien 0 livre pour une ligne sans correspondance, et
--    AVG(l.prix) ignore les valeurs vides — la moyenne reste juste.
SELECT
    a.nationalite,
    COUNT(DISTINCT a.id) AS nb_auteurs,
    COUNT(l.id) AS nb_livres,
    ROUND(AVG(l.prix), 2) AS prix_moyen
FROM auteurs AS a
LEFT JOIN livres AS l ON a.id = l.auteur_id
GROUP BY a.nationalite
ORDER BY nb_livres DESC;
```

</details>

### Bravo !

Tu sais maintenant :

- **Agréger** des données avec COUNT, SUM, AVG, MIN, MAX
- **Distinguer** `COUNT(*)`, `COUNT(colonne)` et `COUNT(DISTINCT colonne)`
- **Regrouper** par catégorie avec GROUP BY
- **Filtrer les groupes** avec HAVING
- **Croiser des tables** avec INNER JOIN et LEFT JOIN
- **Renommer** colonnes et tables avec des alias
- **Présenter** un résultat avec ROUND, `||` et SUM(CASE WHEN …)

Et surtout, deux réflexes qui distinguent une requête juste d'une requête
plausible :

- Une agrégation répond à une question chiffrée. Pour savoir **quelle ligne**
  porte ce chiffre, il faut trier ou faire une sous-requête.
- `INNER JOIN` fait disparaître les lignes sans correspondance. Quand ces lignes
  comptent, c'est `LEFT JOIN` — accompagné de `COUNT(colonne_de_droite)` plutôt
  que `COUNT(*)`.

Au **Niveau 3**, tu apprendras à **écrire et structurer** tes propres bases de données : créer des tables, insérer des données, et organiser le tout proprement.
