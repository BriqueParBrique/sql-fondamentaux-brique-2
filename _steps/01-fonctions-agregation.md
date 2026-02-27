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

INSERT INTO auteurs VALUES
(1, 'Hugo', 'Victor', 'Française'),
(2, 'Rowling', 'J.K.', 'Britannique'),
(3, 'Murakami', 'Haruki', 'Japonaise'),
(4, 'Camus', 'Albert', 'Française'),
(5, 'Christie', 'Agatha', 'Britannique');

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
-- Quel est le livre le moins cher ?
SELECT MIN(prix) FROM livres;

-- Quel est le livre le plus long (en pages) ?
SELECT MAX(pages) FROM livres;
```

### Combiner avec WHERE

On peut filtrer avant d'agréger :

```sql
-- Prix moyen des livres de Fantasy
SELECT AVG(prix) FROM livres WHERE genre = 'Fantasy';
```

### Exercice

1. Combien d'auteurs sont dans la base ?
2. Combien de livres ont été publiés après 1990 ?
3. Quel est le nombre total de pages de tous les livres ?
4. Quel est le prix moyen des livres de genre "Roman" ?
5. Quelle est l'année de publication du livre le plus ancien ?

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

-- 5. Livre le plus ancien
SELECT MIN(annee_publication) FROM livres;
```

</details>
