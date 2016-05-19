# timeviewer

sViewer enhancement to display time layer.


## Drescription:

Travail basé sur le sviewer développé par Geobretagne: https://github.com/georchestra/sviewer

Il s'agit de publier des couches temporelles via Geoserver (WMS TIME) et de les visualiser via un slider ajouté au sviewer.


## Principe de fonctionnement:

L'exemple utilisé ici est celui de données journalières sur la qualité de l'air en Alsace (source: ASPA - http://www.atmo-alsace.net).

### Dans PostGres/PostGIS:

Il est nécessaire de disposer de 2 tables:

- Une table des géométries des objets à représenter (ex.: communes d'Alsace = "COMMUNES_ALSACE"). Elle doit contenir outre la géométrie, au minimum:
    - Un champ de jointure (ex.: PK = code INSEE des communes = "CODE_INSEE")
- Une table des données temporelles (ex.: qualité de l'air des communes d'Alsace = "IQA_COMMUNES_ALSACE"). Elle doit contenir au minimum: 
    - Un champ de jointure (ex.: FK = code INSEE des communes = "CODE_INSEE")
    - Un champ date (ex.: "DATE") 
    - Un champ de valeur (indicateur à représenter, ex.: indicateur de la qualité de l'air = "IQA").

Il faut ensuite créer une vue par jointure entre les 2 tables.

### Dans Geoserver:

Créer une couche sous la forme d'une vue (code à adapter en fonction de la structure des tables utilisées):

```
select pk as id, name_commune, to_timestamp(date, 'DD/MM/YYYY') as date, iqa as atmo_iqa, bb.the_geom from cigal.iqa left join cigal.commune_atmo on id_commune = code_insee;
```

Activer la dimension temporelle sur le champ DATE (présentation = Liste).


### Dans le code du sviewer:

Le principe est de détecter le paramètre d'URL "timeLayer". S'il est présent, alors la liste des dates est récupérée à partir du getcapabilities du WMS temporelle de la couche et le slider est ajouté à la page.

2 fichiers du sviewer ont été modifiés:
- index. html
- js/sviewer.js

Les parties de code ajoutées/modifiées sont précédées d'un commentaire débutant par ```<!-- GRK``` (html) ou ```// GRK``` (js).


### Limites et améliorations à prévoir:

Etudier l'indexation des champs pour éviter la surcharge de Geoserver.  
Le getcapabilities contient toutes les dates offertes par la couche.


### Utilisation:

Ajouter dans l'URL du WMS, la date sous la forme "&TIME=2015-09-18".  
Attention: utiliser une date qui existe dans la base de données!


### Démos:

Version A: (champ date = input)
https://www.cigalsace.org/tools/timeviewer2/?timeLayer=cigal:CIGAL_demo_atmo_iqa_temporel_Alsace_C48

Version B: (champ date = select)
https://www.cigalsace.org/tools/timeviewer/?timeLayer=cigal:CIGAL_demo_atmo_iqa_temporel_Alsace_C48
