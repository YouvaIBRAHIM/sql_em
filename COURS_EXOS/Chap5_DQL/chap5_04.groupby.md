# DQL

## Fonctions de groupe

Elles s'utilisent dans la clause SELECT sur une/des colonnes, elles permettent de regrouper des données. Si vous utilisez les fonctions de groupement avec une requête ne contenant pas de clause GROUP BY, cela revient à grouper toutes les lignes.

```sql
AVG([DISTINCT] exp)       -- moyenne
COUNT({*| DISTINCT] exp}) -- nombre de lignes
MAX([DISTINCT] exp)       -- max
MIN([DISTINCT] exp)       -- min
SUM([DISTINCT] exp)       -- somme
GROUP_CONCAT(exp)         -- composition d'un nombre de valeurs, concaténation de valeurs de champ a, b, c, d
VARIANCE(exp)             -- variance
STDDEV(exp)               -- écart type (standard deviation)
```
## Groupement de lignes

```sql
SELECT col1 [,col2, ...], fonction_groupe
FROM table
WHERE (conditions)
**GROUP BY clo1 [, col2, ...]**
HAVING condition_02
ORDER BY col1 [ASC | DESC] [, col2 ...]
LIMIT
```

- La clause `WHERE` exclut des lignes pour chaque groupement ou permet de rejeter des groupements entiers. Elle s'applique à la totalité de la table.

- La clause `GROUP BY` liste des colonnes de groupement.

- La clause `HAVING` permet de poser des conditions sur chaque groupement.

Attention, les colonnes présentes dans le `SELECT` doivent apparaître dans le `GROUP BY`. Seules des fonctions ou expressions peuvent exister en plus dans le `SELECT`.

Par exemple on peut se poser la question y a t il des anomymes dans la table pilots avec un group by comme suit, si la requête ne renvoie rien il n'en n'existe pas dans nos données.

```sql
 select 
 name, last_name, count(*) as c 
 from pilots 
 group by name, last_name 
 having c > 1;
```

## 01 Exercices group by

01. **Exercice moyenne des heures de vol**

Calculez la moyenne des heures de vol pour chaque compagnie.

02. **Exercice moyenne et bonus**

Calculez la moyenne des heures de vol des pilotes dont le bonus est de 500, par compagnie.

03. **Exercice nombre de pilotes**

Sélectionnez les compagnies ayant plus d'un pilote, ainsi que leur nombre de pilotes.

04. **Exercice ajout d'une colonne & valeurs**

*Si vous n'avez pas fait les exercices supplémentaires. Vous devez faire ce qui suit, sinon passer à la suite.*

Ajoutez une colonne `plane` à la table pilots de type `ENUM` ayant pour valeurs possibles :
`A380`, `A320`, `A340`.

Et mettez à jour la table avec les contraintes suivantes :

- `Alan`, `Sophie`, `Albert`, `Benoit` volent sur un *A380*.

- `Tom`, `John`, `Yi` volent sur un *A320*.

- `Yan`, `Pierre` volent sur un *A340*.

05. **Exercice sélectionner pilotes & compagnies**

Sélectionnez le nombre de pilotes par compagnie et par type d'avion.

06. **Exercice noms de pilotes**

Sélectionnez le nom des pilotes par bonus.

Sélectionnez le nom et la compagnie des pilotes par bonus.

07. **Exercice étendue**

Calculez l'étendue (différence entre MAX et MIN) du nombre d'heures de vol par compagnie.

08. **Exercice moyenne et nombre de jours**

Faites la somme du nombre de jours de vols par compagnie dont la somme est supérieure à 30.

09. **Exercice moyenne des heures de vol**

Afficher la moyenne des heures de vol pour les compagnies qui sont en France.

## ROLLUP

La clause `WITH ROLLUP` s'ajoute après un `GROUP BY` permet de créer des regroupement combinés.

Prenons un exemple standard :

```sql
SELECT company, plane, COUNT(*) AS nb_pilots
FROM pilots
GROUP BY company, plane;

-- +---------+-------+-----------+
-- | company | plane | nb_pilots |
-- +---------+-------+-----------+
-- | AUS     | A380  |         4 |
-- | FRE1    | A320  |         2 |
-- | SIN     | A320  |         1 |
-- | SIN     | A340  |         1 |
-- | CHI     | A340  |         1 |
-- +---------+-------+-----------+
```

Avec la clause `WITH ROLLUP`, cela permettra d'obtenir une ligne supplémentaire pour chaque groupe avec la valeur agrégée d'une seule colonne :

> *La valeur NULL sur une colonne indique la présence d'une ligne supplémentaire*

```sql
SELECT company, plane, COUNT(*) AS nb_pilots
FROM pilots
GROUP BY company, plane WITH ROLLUP;

-- +---------+-------+-----------+
-- | company | plane | nb_pilots |
-- +---------+-------+-----------+
-- | AUS     | A380  |         4 |
-- | AUS     | NULL  |         4 |	<-- Total du groupe 'AUS' (1 pilote)
-- | CHI     | A340  |         1 |
-- | CHI     | NULL  |         1 |  <-- Total du groupe 'CHI'
-- | FRE1    | A320  |         2 |
-- | FRE1    | NULL  |         2 |  <-- Total du groupe 'FRE1'
-- | SIN     | A320  |         1 |
-- | SIN     | A340  |         1 |
-- | SIN     | NULL  |         2 |  <-- Total du groupe 'SIN' (2 pilotes)
-- | NULL    | NULL  |         9 |	<-- Total de l'ENSEMBLE (9 pilotes)
-- +---------+-------+-----------+
```

Cela peut être pratique pour éviter des calculs supplémentaires dans la partie applicative (code serveur).

## 02 Exercices sales et découverte des procédures

1. Créez la table sales, elle représente la table des dépenses. Mettez à jour cette table avec les données suivantes :

**Création de la table sales référencée à la table companies, avec les champs suivants**

id : bigint unsigned auto increment
year_month : datetime
company : clé étrangère référencée à la table companies
profit : champ décimal de 10 chiffres avec 2 chiffres après la virgule.

2. Procédure découverte 

Nous allons découvrir une méthode pour hydrater la table sales : une procédure SQL, sorte de fonction que nous appelerons dans la console MySQL pour exécuter un script. Voici ci-dessous un exemple de procédure :

```sql
DELIMITER $$

DROP PROCEDURE IF EXISTS calculate;
CREATE PROCEDURE calculate(IN x INT, IN y INT, OUT sum INT)
BEGIN
  SET sum = x + y;
END $$;
DELIMITER ;

-- Appel de la procédure dans la console mysql
call calculate(1, 2, @s);
SELECT @s;
```

Nous vous donnons cette première procédure afin de découvrir la syntaxe SQL pour l'implémenter dans votre code, ATTENTION pensez à bien échapper le champ year_month avec les apostrophes simples

```sql

-- change le délimiter pour créer la procédure
DELIMITER $$

DROP PROCEDURE IF EXISTS set_data$$
CREATE PROCEDURE set_data(IN  comp CHAR(4))
BEGIN
  DECLARE i INT DEFAULT 1;
  DECLARE d DATE DEFAULT '1980-01-01';
  loop_data : LOOP

    IF (i = 20*12) THEN
        LEAVE loop_data;
    END IF;

    INSERT INTO 
    `sales` (created_at, company, profit) VALUES ( d, comp, ROUND(RAND()*15 * 100000, 2 ));

    SET d = DATE_ADD(d, INTERVAL 1 MONTH);
    SET i = 1 + i;
  END LOOP; 
END$$

-- rétablit le délimiter dans la console
DELIMITER ;

call set_data('AUS');
call set_data('CHI');
call set_data('SIN');
call set_data('FRE1');
call set_data('ITA');
```

```sql
DROP TABLE IF EXISTS sales;
CREATE TABLE `sales` (
    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `created_at` DATE DEFAULT '1980-01-01',
    `company` CHAR(4),
    `profit` DECIMAL(15,2),
    CONSTRAINT pk_id PRIMARY KEY (`id`)
) ENGINE=InnoDB ;

ALTER TABLE sales ADD CONSTRAINT fk_sales_company FOREIGN KEY (company) REFERENCES companies(`comp`);

```

3.  Ré-écrire la procédure en créant une table temporaire et en utilisant INSERT et SELECT pour automatiser l'hydration de la table sales. Voici comment vous allez créer une table temporaire dans votre procédure :

```sql
DROP TABLE IF EXISTS comp_companies;
CREATE TEMPORARY TABLE comp_companies SELECT '1980-01-01' as c_at , comp, ROUND(RAND()*15 * 100000, 2 ) FROM companies;
```

## 03 Exercices sales ROLLUP

1. Calculez les profits de chaque compagnie par année, ainsi que le total par année pour toutes les compagnies et enfin le total global des profits.

2. Calculez maintenant par année et par mois avec la même granularité que la question 1 respectivement par rapport à ce dernier regroupement.



