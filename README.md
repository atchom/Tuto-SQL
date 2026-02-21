# Tuto-SQL

## SUPPRESSION DES DOUBLONS
 Voici un exemple de création de la table employes avec des données qui permettront de tester le script de détection des doublons basé sur nom, prenom et la date_embauche la plus récente.
 ### Creation de la table Employes
 ```
-- Création de la table employes
CREATE TABLE employes (
    id INT IDENTITY(1,1) PRIMARY KEY,
    nom NVARCHAR(50) NOT NULL,
    prenom NVARCHAR(50) NOT NULL,
    date_embauche DATE NOT NULL,
    poste NVARCHAR(100),
    salaire DECIMAL(10,2)
);
GO

-- Insertion de données d'exemple (avec doublons sur nom + prénom)
INSERT INTO employes (nom, prenom, date_embauche, poste, salaire) VALUES
-- Jean Dupont : deux entrées (la plus récente est le 15/03/2022)
('Dupont', 'Jean', '2020-01-10', 'Développeur', 35000.00),
('Dupont', 'Jean', '2022-03-15', 'Lead Développeur', 48000.00),

-- Claire Martin : une seule entrée
('Martin', 'Claire', '2021-06-01', 'Chef de projet', 52000.00),

-- Pierre Durand : deux entrées (la plus récente est le 10/10/2023)
('Durand', 'Pierre', '2019-11-20', 'Technicien', 28000.00),
('Durand', 'Pierre', '2023-10-10', 'Technicien Senior', 32000.00),

-- Sophie Lefebvre : une seule entrée
('Lefebvre', 'Sophie', '2022-09-05', 'Comptable', 38000.00),

-- Thomas Bernard : trois entrées (la plus récente est le 01/12/2024)
('Bernard', 'Thomas', '2020-03-12', 'Analyste', 40000.00),
('Bernard', 'Thomas', '2021-07-19', 'Analyste confirmé', 44000.00),
('Bernard', 'Thomas', '2024-12-01', 'Expert analytique', 55000.00);
GO
```
### Explication
 - La colonne id est une clé primaire auto-incrémentée.  ### (IDENTITY) pour identifier chaque ligne de manière unique.
 - Les colonnes nom et prenom sont les critères de partitionnement du ROW_NUMBER().
 - La colonne date_embauche est utilisée dans l'ordre décroissant pour classer les enregistrements au sein de chaque groupe (nom + prénom).
 - Les colonnes supplémentaires (poste, salaire) ne sont pas nécessaires pour le script mais illustrent un cas réel.
### Detection des Doublons
```
SELECT nom, prenom, COUNT(*) AS nb
FROM employes
GROUP BY nom, prenom
HAVING COUNT(*) > 1;
```
### Deuxieme methode
```
-- Voir tous les doublons avec un numéro
WITH Duplicates AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY nom, prenom 
               ORDER BY date_embauche DESC  -- Garde le plus récent en premier
           ) AS rn
    FROM employes
)
SELECT * FROM Duplicates ORDER BY nom, prenom, rn;
```
### Explication :
 - PARTITION BY nom, prenom : crée des groupes pour chaque combinaison nom/prénom

 - ORDER BY date_embauche DESC : trie par date la plus récente d'abord

 - rn = 1 : le premier de chaque groupe (le plus récent)

 - rn > 1 : les doublons à supprimer
### Étape 2 : Supprimer les doublons
```
-- Supprimer les doublons (garder seulement rn = 1)
WITH Duplicates AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY nom, prenom 
               ORDER BY date_embauche DESC
           ) AS rn
    FROM employes
)
DELETE FROM Duplicates WHERE rn > 1;
```
### Version avec création d'une nouvelle table propre
```
-- 1. Créer une nouvelle table sans doublons
SELECT 
    nom,
    prenom,
    poste,
    date_embauche
INTO employes_propre  -- Crée une nouvelle table
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY nom, prenom 
               ORDER BY date_embauche DESC
           ) AS rn
    FROM employes
) AS temp
WHERE rn = 1;  -- Garder seulement le premier de chaque groupe

-- 2. Vérifier
SELECT * FROM employes_propre;
```
