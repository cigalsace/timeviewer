# timeviewer

sViewer enhancement to display time layer.


## Drescription:

Travail bas� sur le sviewer d�velopp� par Geobretagne: https://github.com/georchestra/sviewer

Il s'agit de publier des couches temporelles via Geoserver (WMS TIME) et de les visualiser via un slider ajout� au sviewer.


## Principe de fonctionnement:

L'exemple utilis� ici est celui de donn�es journali�res sur la qualit� de l'air en Alsace (source: ASPA - http://www.atmo-alsace.net).

### Dans PostGres/PostGIS:

Il est n�cessaire de disposer de 2 tables:

- Une table des g�om�tries des objects � repr�senter (ex.: communes d'alsace = "COMMUNES_ALSACE"). Elle doit contenir outre la g�om�trie, au minimum:
    - Un champ de jointure (ex.: PK = code INSEE des communes = "CODE_INSEE")
- Une table des donn�es temporelles (ex.: qualit� de l'air des commmunes d'Alsace = "IQA_COMMUNES_ALSACE"). Elle doit contenir au minimum: 
    - Un champ de jointure (ex.: FK = code INSEE des communes = "CODE_INSEE")
    - Un champ date (ex.: "DATE") 
    - Un champ de valeur (indicateur � repr�senter, ex.: indicateur de la qualit� de l'air = "IQA").

Il faut ensuite cr�er une vue par jointure entre les 2 tables.

### Dans Geoserver:

Cr�er une couche sous la forme d'une vue (code � adapter en fonction de la structure des tables utilis�es):

```
select pk as id, name_commune, to_timestamp(date, 'DD/MM/YYYY') as date, iqa as atmo_iqa, bb.the_geom from cigal.iqa left join cigal.commune_atmo on id_commune = code_insee;
```

Activer la dimension temporelle sur le champ DATE (pr�sentation = Liste).


### Dans le code du sviewer:

Le principe est de d�tecter le param�tre d'URL "timeLayer". S'il est pr�sent, alors la liste des dates est r�cup�r�e � partir du getcapabilities du WMS temporelle de la couche et le slider est ajout� � la page.

2 fichiers du sviewer ont �t� modifi�s:
- index. html
- js/sviewer.js

Les parties de code ajout�es/modifi�es sont pr�c�d�es d'un commentaire d�butant par ```<!-- GRK``` (html) ou ```// GRK``` (js).


### Limites et am�liorations � pr�voir:

Etudier l'indexation des champs pour �viter la surcharge de Geoserver.  
Le getcapabilities contient toutes les dates offertes par la couche.


### Utilisation:

Ajouter dans l'URL du WMS, la date sous la forme "&TIME=2015-09-18".  
Attention: utiliser une date qui existe dans la base de donn�es!


### D�mos:

Version A: (champ date = input)
https://www.cigalsace.org/tools/timeviewer2/?timeLayer=cigal:CIGAL_demo_atmo_iqa_temporel_Alsace_C48

Version B: (champ date = select)
https://www.cigalsace.org/tools/timeviewer/?timeLayer=cigal:CIGAL_demo_atmo_iqa_temporel_Alsace_C48