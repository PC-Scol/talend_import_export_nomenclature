# talend_import_export_nomenclature
Scripts permettant de récupérer et d'injecter une nomenclatures pegase via une table dans un BD postGres
Dans le cadre de la RDD : Récupération de nomenclature dans un pegase pour réinjection dans un autre, ou dans un pegase réinitialisé

Introduction
-------------

Dans le cadre du passage à pegase, nous sommes amenés à rejouer un certain nombre de fois les tests afin d’alimenter et d’initialiser les environnements pegase au mieux.
L’outil de RDD permet ainsi de recuperer des données, dans notre cas d’apogee, pour les reinjecter dans pegase. Cependant, il est aussi necessaire de faire des saisies manuelles à certains moments afin de reajuster les nomenclatures dans pegase.
Les scripts ci-dessous permettent de recuperer dans un pegase les données d’un certain nombre de nomenclatures, qu’il sera ensuite possible de reinjecter dans un autre pegase ou dans un pegase reinitialisé.

Recuperation des données de pegase
----------------------------------

La recuperation des données passe dans notre cas par une base de données postgres , dans laquelle nous avons créé un schema nommé interface_si, une table nommée rdd_ref_nomenclature_generique, et un utilisateur permetant d’acceder à tout cela.


Script de creation du login :

CREATE ROLE <login>  WITH
  LOGIN
PASSWORD '<motdepasse>';


Script du schema :

CREATE SCHEMA IF NOT EXISTS interface_si
    AUTHORIZATION <login >;

GRANT ALL ON SCHEMA  interface_si TO <login >;


Script de creation de la table :

Les nomenclatures sont globalement structurées de la meme facon, s’ajoute par fois quelques informations, mais le socle de base reste le meme. 
Cette table contient donc tous les éléments de nomenclatures que nous recuperons, a savoir :
- bourse et aide financière
- régime d'inscription
- serie bac avec titre d’accès et type de bac
- situation militaire
- type de diplôme avec nature de diplôme, type de formation et cursus de formation
- type de formation
- type d'objet formation

-- Table: interface_si.rdd_ref_nomenclature_generique

-- DROP TABLE IF EXISTS interface_si.rdd_ref_nomenclature_generique;

CREATE TABLE IF NOT EXISTS interface_si.rdd_ref_nomenclature_generique
(
    "typeNomenclature" text COLLATE pg_catalog."default" NOT NULL,
    code text COLLATE pg_catalog."default" NOT NULL,
    "libelleCourt" text COLLATE pg_catalog."default" NOT NULL,
    "libelleLong" text COLLATE pg_catalog."default" NOT NULL,
    "libelleAffichage" text COLLATE pg_catalog."default",
    "prioriteAffichage" integer,
    "dateDebutValidite" date NOT NULL,
    "dateFinValidite" date,
    "temoinVisible" boolean,
    "temoinLivre" boolean,
    "categorieObjet" text COLLATE pg_catalog."default",
    "codeBcn" text COLLATE pg_catalog."default",
    "exoinsExtra" text COLLATE pg_catalog."default",
    "temoinExoneration" boolean,
    "typeBourse" text COLLATE pg_catalog."default",
    "exoinsCom" text COLLATE pg_catalog."default",
    "codeSISEIns" integer,
    "codeSISERslt" integer,
    "temoinCVEC" boolean,
    "financementPossible" boolean,
    "droitABourse" boolean,
    "typeNomenclatureCursusFormation" text COLLATE pg_catalog."default",
    "codeCursusFormation" text COLLATE pg_catalog."default",
    "typeNomenclatureNiveauFormation" text COLLATE pg_catalog."default",
    "codeNiveauFormation" text COLLATE pg_catalog."default",
    "typeNomenclatureNatureDiplome" text COLLATE pg_catalog."default",
    "codeNatureDiplome" text COLLATE pg_catalog."default",
    "typeNomenclatureTitreAcces" text COLLATE pg_catalog."default",
    "codeTitreAcces" text COLLATE pg_catalog."default",
    "typeNomenclatureTypeBac" text COLLATE pg_catalog."default",
    "codeTypeBac" text COLLATE pg_catalog."default",
    CONSTRAINT rdd_ref_nomenclature_generique_pkey PRIMARY KEY ("typeNomenclature", code, "dateDebutValidite")
)

TABLESPACE pg_default;	

COMMENT ON TABLE interface_si.rdd_ref_nomenclature_generique
    IS 'Table contenant le referentiel generique de pegase ';

GRANT ALL ON TABLE interface_si.rdd_ref_nomenclature_generique TO <login>;


Job talend de recuperation et d’injection
------------------------------------------

Les job sont sous un projet talend. Ils sont basic dans le sens ou ne sont pas gérer finement :
- les erreurs
- les logs
- le comptage des enregistrements 
- …

Ils se contentent pour les scripts de recuperation :
-	Recuperer les données via les API pegase
-	Deverser les données dans la table decrite ci-dessus
  
Ils se contentent pour les scripts d’injection:
-	Recuperer les données dans la table
-	Structurer les données pour les preparer à leur injection
-	Injecter les données dans pegase

Les zip livrés
---------------
  
- JobsRDD_recuperation_injection_nomenclatures.zip contient un export du projet
- json.zip : contient des fichiers exemple de description des json attendu par les API pegase. Nous les mettons dans un repertoire local/json dans notre projet
- lib.zip : les librairies java utilisées pour ce projet. Nous les mettons dans un repertoire local/lib dans notre projet


