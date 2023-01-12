<!-- .slide -->

# Les objets principaux

* Clusters
* Bases de données
* Collections
* Documents
* Champs

* MongoDB stocke les documents en *BSON* = Binary JSON
  * Étend les types classiques JSON
  * Ajoute de nouveaux types comme ObjectID ou les dates

Notes:
- Clusters organisés en single node (dev only), replicasets ou shards
- En opposition évidemment aux tables / lignes / colonnes
- Quelques bases de données réservées (admin, config, oplog)
- En réalité, plus ont s'éloigne des 16Mo, mieux c'est

##==##
<!-- .slide: class="with-code"-->

```javascript[|3,4,9,11,12,16,18-20|25|26|5,7-23|13|2,6]
{
	"_id": ObjectID("507c7f79bcf86cd7994f6c0e"),
	"firstname": "Sophie",
	"lastname": "Fonfec",
    "tags": ["chi", "fou", "mi"],
    "company": ObjectID("507c7f79bcf86cd7993f5c0b"),
    "address": [
        {
            "type": "work",
            "number": NumberInt(74),
            "street": "rue des arts",
            "zipcode": "59000",
            "location": { "type": "Point", "coordinates": [ 50.6397, 3.0655 ] }
        },
        {
            "type": "home",
            "number": NumberInt(1),
            "ext": "B",
            "street": "Pl. des Patiniers",
            "zipcode": "59000",
            "city": "Lille"
        }
    ],
    "dates": {
        "opt-in": ISODate("2021-08-08T13:37:00Z"),
        "opt-out": null
    }
}
```
<!-- .element: class="full-height" -->

##==##

# Pour bien commencer

* *dynamic schema* plutôt que *schemaless*<br><br>
* Regrouper les champs ensemble s'ils ont vocation à l'être
  * Les *arrays*, *nested* fields et *nested* docs sont des atouts du modèle document
  * Gain de lisibilité et logique métier visible immédiatement
  * Gain de performance potentiel si correctement structuré<br/><br/>
* Se poser les bonnes questions avant de stocker une donnée dans un document
  * Est-ce bien sa place ?
  * Est-ce que cette donnée devra être modifiée ? Souvent ?
  * Est-ce que cette donnée doit être associée à un timestamp ? Doit être historisée ?
  * Est-ce qu'il faut la répliquer à plusieurs endroits ?
  
Notes:
- Dynamic Schema parce qu'avec le temps, le contenu des collections va évoluer et que schemaless insite trop à faire n'importe quoi
- Les validators = des contraintes, avec un coût en performnce  
- Exemple: une commande, les lignes de commandes, les produits
- Exemple: une liste de tags, est-ce intéressant de les trier au stockage, de les trier par une date d'entrée ?  
- Exemples de modélisation à suivre

##==##

# Simple rules

## *"Ca n'est pas parce qu'on peut le faire qu'on doit le faire"*

* Ne *jamais* stocker différents types de données dans le même champ
  * Ne pas mélanger les types de nombres (`Int64`, `Float`, `Decimal`, etc.)
  * Ne pas stocker des dates en `string` et des objets ISODate dans le même champ
  * Ne pas mélanger des éléments de types différents dans les mêmes tableaux<br><br>
* Ne pas stocker les date/datetime en `string`, utiliser toujours le type `ISODate`
  * Moins lourd, plus rapide et offre toutes les possibilités de manipulation, dont les timezones<br><br>
* Utiliser la notation *GeoJSON* avec les données géographiques<br/><br/>
* Ne pas donner des noms à rallonge inutilement aux champs, et toujours des noms clairs
  * "a_decide_de_se_desabonner": true
  * "who": "repeat"
  
Notes:
- Il existe des solutions, dans le code ou via des validators JSON Schema

##==##

# Les risques si on oublie ces règles

* Requêtes qui ne fonctionnent pas
  * "1980-04-28" != ISODate("1980-04-28T00:00:00Z")
  * "2022" != 2022  
  * MongoDB retournera un résultat vide, mais ne va pas planter
<br/><br/>
* Problèmes de performance
  * Manipuler le bon type, qui pèse le bon poids, est toujours + performant
<br/><br/>
* Maintenance compliquée pour corriger les erreurs

Notes:
- En SQL, le typage est aussi vérifié à la requête, pas uniquement à l'écriture
- Maintenance compliquée car aller corriger des données dans un modèle existant et déjà exposé est souvent très pénible
