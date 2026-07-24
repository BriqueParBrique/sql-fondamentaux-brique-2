---
title: "Fonctions d'agrégation"
---

## Calculer sur un ensemble de données

Les fonctions d'agrégation permettent de faire des calculs sur **plusieurs lignes** et de renvoyer **un seul résultat**. C'est comme demander "combien ?", "quel total ?", "quelle moyenne ?" à ta base de données.

### Préparer notre base de données

Pour ce niveau, on va travailler avec une **base de bibliothèque**. Copie et exécute ce code pour créer les tables :

```sql
CREATE TABLE auteurs (
    id INTEGER PRIMARY KEY,
    nom TEXT,
    prenom TEXT,
    nationalite TEXT
);

CREATE TABLE livres (
    id INTEGER PRIMARY KEY,
    titre TEXT,
    auteur_id INTEGER,
    genre TEXT,
    pages INTEGER,
    prix REAL,
    annee_publication INTEGER
);

CREATE TABLE emprunts (
    id INTEGER PRIMARY KEY,
    livre_id INTEGER,
    emprunteur TEXT,
    date_emprunt TEXT,
    date_retour TEXT
);

-- Note : Marguerite Duras est enregistrée comme autrice, mais aucun de ses
-- livres n'est encore au catalogue. C'est volontaire : ce cas nous servira à
-- l'étape 5 pour comprendre la différence entre INNER JOIN et LEFT JOIN.
INSERT INTO auteurs VALUES
(1, 'Hugo', 'Victor', 'Française'),
(2, 'Rowling', 'J.K.', 'Britannique'),
(3, 'Murakami', 'Haruki', 'Japonaise'),
(4, 'Camus', 'Albert', 'Française'),
(5, 'Christie', 'Agatha', 'Britannique'),
(6, 'Duras', 'Marguerite', 'Française');

INSERT INTO livres VALUES
(1, 'Les Misérables', 1, 'Roman', 1488, 12.50, 1862),
(2, 'Notre-Dame de Paris', 1, 'Roman', 940, 9.90, 1831),
(3, 'Harry Potter à l''école des sorciers', 2, 'Fantasy', 308, 8.90, 1997),
(4, 'Harry Potter et la chambre des secrets', 2, 'Fantasy', 364, 8.90, 1998),
(5, 'Kafka sur le rivage', 3, 'Roman', 638, 11.00, 2002),
(6, 'La Ballade de l''impossible', 3, 'Roman', 382, 9.50, 1987),
(7, 'L''Étranger', 4, 'Philosophie', 186, 6.90, 1942),
(8, 'La Peste', 4, 'Roman', 308, 7.90, 1947),
(9, 'Le Crime de l''Orient-Express', 5, 'Policier', 256, 7.50, 1934),
(10, 'Dix Petits Nègres', 5, 'Policier', 272, 7.50, 1939),
(11, 'Harry Potter et le prisonnier d''Azkaban', 2, 'Fantasy', 468, 9.90, 1999),
(12, '1Q84', 3, 'Roman', 1056, 14.90, 2009);

INSERT INTO emprunts VALUES
(1, 1, 'Alice Martin', '2024-01-10', '2024-02-10'),
(2, 3, 'Bob Dupont', '2024-01-15', '2024-02-01'),
(3, 7, 'Claire Petit', '2024-02-01', '2024-02-15'),
(4, 3, 'Alice Martin', '2024-02-20', NULL),
(5, 5, 'David Moreau', '2024-03-01', '2024-03-20'),
(6, 9, 'Emma Leroy', '2024-03-05', NULL),
(7, 11, 'Bob Dupont', '2024-03-10', NULL),
(8, 2, 'Claire Petit', '2024-03-15', '2024-04-01');
```

### COUNT — Compter

`COUNT` compte le nombre de lignes :

```sql
-- Combien de livres dans la bibliothèque ?
SELECT COUNT(*) FROM livres;
```

Résultat : **12**

```sql
-- Combien de livres ont plus de 500 pages ?
SELECT COUNT(*) FROM livres WHERE pages > 500;
```

#### `COUNT(*)` ou `COUNT(colonne)` ?

Les deux existent et ne comptent pas la même chose :

| Écriture          | Ce qui est compté                                            |
| ----------------- | ------------------------------------------------------------ |
| `COUNT(*)`        | Toutes les lignes                                            |
| `COUNT(colonne)`  | Les lignes où **cette colonne a une valeur** (non vide)       |
| `COUNT(DISTINCT colonne)` | Les valeurs **différentes** de cette colonne         |

Sur notre table `livres`, toutes les colonnes sont remplies, donc `COUNT(*)` et
`COUNT(prix)` donnent le même résultat : 12. Mais sur la table `emprunts`, la
colonne `date_retour` est vide pour les livres non encore rendus :

```sql
SELECT
    COUNT(*),            -- 8 : tous les emprunts
    COUNT(date_retour)   -- 5 : seulement ceux qui ont été rendus
FROM emprunts;
```

Cette différence est un outil, pas un piège : `COUNT(date_retour)` est la façon
la plus courte de compter les emprunts terminés.

Et pour compter les valeurs distinctes :

```sql
-- Combien de genres différents ? (et non : combien de livres)
SELECT COUNT(DISTINCT genre) FROM livres;
```

#### Les agrégations ignorent les valeurs vides

`SUM`, `AVG`, `MIN` et `MAX` **sautent** les cases vides au lieu de les compter
comme des zéros. C'est presque toujours ce qu'on veut — une moyenne calculée sur
les valeurs connues est plus juste qu'une moyenne où les inconnues valent 0 —
mais il faut le savoir : `AVG` divise par le nombre de valeurs *présentes*, pas
par le nombre de lignes. On reparle des valeurs vides à l'étape 5.

### SUM — Additionner

`SUM` calcule la somme d'une colonne numérique :

```sql
-- Quel est le prix total de tous les livres ?
SELECT SUM(prix) FROM livres;
```

### AVG — Calculer la moyenne

```sql
-- Quel est le prix moyen d'un livre ?
SELECT AVG(prix) FROM livres;
```

### MIN et MAX — Trouver les extrêmes

```sql
-- Quel est le prix le plus bas du catalogue ?
SELECT MIN(prix) FROM livres;

-- Quel est le nombre de pages le plus élevé ?
SELECT MAX(pages) FROM livres;
```

### Le piège classique : « quel livre ? » ≠ « quel prix ? »

Relis bien les deux requêtes ci-dessus : elles répondent à *« quel prix ? »* et
*« combien de pages ? »*. Elles ne disent **pas** de quel livre il s'agit.

```sql
SELECT MIN(prix) FROM livres;
```

Résultat : une seule cellule, `6.9`. Pas de titre, pas d'auteur — juste le
nombre.

Et il est tentant d'écrire ceci pour obtenir le titre :

```sql
-- ❌ N'écris pas ça : le résultat est trompeur
SELECT titre, MIN(prix) FROM livres;
```

SQLite accepte cette requête, mais **d'autres bases la refusent**, et surtout la
logique est douteuse : on demande une valeur unique (`MIN(prix)`) à côté d'une
colonne qui a 12 valeurs différentes (`titre`). Rien ne garantit que le titre
affiché soit celui du livre le moins cher.

Pour obtenir **la ligne** et pas seulement la valeur, on utilise ce qu'on sait
déjà du niveau 1 — trier et prendre le premier :

```sql
-- ✓ Le livre le moins cher, avec son titre
SELECT titre, prix FROM livres
ORDER BY prix
LIMIT 1;
```

| titre       | prix |
| ----------- | ---- |
| L'Étranger  | 6.9  |

> **Retiens la distinction** : une fonction d'agrégation répond à une question
> *chiffrée* (« combien ? », « quelle moyenne ? », « quel maximum ? »). Pour
> savoir *quelle ligne* porte ce chiffre, il faut trier (`ORDER BY … LIMIT 1`)
> ou, comme on le verra au niveau 3, écrire une sous-requête.
>
> Et comme au niveau 1 : s'il y a des ex æquo, `LIMIT 1` en choisit un
> arbitrairement. Deux livres sont à 7.50 dans notre table — un `ORDER BY prix,
> titre` rendrait le résultat reproductible.

### Combiner avec WHERE

On peut filtrer avant d'agréger :

```sql
-- Prix moyen des livres de Fantasy
SELECT AVG(prix) FROM livres WHERE genre = 'Fantasy';
```

### Exercice

Le résultat attendu est indiqué entre parenthèses.

1. Combien d'auteurs sont dans la base ? *(6)*
2. Combien de livres ont été publiés après 1990 ? *(5)*
3. Quel est le nombre total de pages de tous les livres ? *(6666)*
4. Quel est le prix moyen des livres de genre "Roman" ? *(environ 10.95)*
5. Quelle est l'année de publication la plus ancienne ? *(1831)*
6. Combien de genres différents le catalogue contient-il ? *(4)*
7. Combien d'emprunts n'ont pas encore été rendus ? *(3 — indice : `COUNT(*)` moins `COUNT(date_retour)`, ou bien la condition vue à l'étape 5)*
8. Affiche le titre et le nombre de pages du livre le plus long *(1 ligne — attention, `MAX(pages)` seul ne suffit pas)*

<details>
<summary>Voir les réponses</summary>

```sql
-- 1. Nombre d'auteurs
SELECT COUNT(*) FROM auteurs;

-- 2. Livres après 1990
SELECT COUNT(*) FROM livres WHERE annee_publication > 1990;

-- 3. Total des pages
SELECT SUM(pages) FROM livres;

-- 4. Prix moyen des Romans
SELECT AVG(prix) FROM livres WHERE genre = 'Roman';

-- 5. Année de publication la plus ancienne
SELECT MIN(annee_publication) FROM livres;

-- 6. Nombre de genres différents
SELECT COUNT(DISTINCT genre) FROM livres;

-- 7. Emprunts non rendus : total moins ceux qui ont une date de retour
SELECT COUNT(*) - COUNT(date_retour) FROM emprunts;

-- 8. Le livre le plus long, avec son titre.
--    MAX(pages) donnerait le nombre (1488) mais pas le titre : il faut trier.
SELECT titre, pages FROM livres
ORDER BY pages DESC
LIMIT 1;
```

</details>
