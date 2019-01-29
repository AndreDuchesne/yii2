Quick Start
===========

Yii fournit un ensemble d'outils permettant de simplifier la mise en œuvre des API de service Web RESTful.
En particulier, Yii prend en charge les fonctionnalités suivantes relatives aux API RESTful:


* Prototypage rapide avec prise en charge des API communes pour [Active Record] (db-active-record.md);
* Négociation du format de réponse (prenant en charge JSON et XML par défaut);
* Sérialisation d'objet personnalisable avec prise en charge de champs de sortie sélectionnables;
* Mise en forme correcte des données de collecte et des erreurs de validation;
* Prise en charge de [HATEOAS] (http://en.wikipedia.org/wiki/HATEOAS);
* Routage efficace avec vérification du verbe HTTP appropriée;
* Prise en charge intégrée des verbes `OPTIONS` et` HEAD`;
* Authentification et autorisation;
* Mise en cache des données et mise en cache HTTP;
* Limitation du débit;


Dans ce qui suit, nous utilisons un exemple pour illustrer comment vous pouvez créer un ensemble d'API RESTful avec un effort de codage minimal.

Supposons que vous souhaitiez exposer les données utilisateur via des API RESTful. Les données utilisateur sont stockées dans la table `user` de la base de données,
et vous avez déjà créé la classe [app \ models \ User`] de la [active record] (db-active-record.md) pour accéder aux données de l'utilisateur.


## Création du contrôleur <span id="creating-controller"></span>


Tout d’abord, créez un [contrôleur] (structure-controllers.md) de classe `app \ controllers \ UserController` comme suit,

```php
namespace app\controllers;

use yii\rest\ActiveController;

class UserController extends ActiveController
{
    public $modelClass = 'app\models\User';
}
```

La classe de contrôleur s'étend à partir de [[yii \ rest \ ActiveController]], qui implémente un ensemble commun d'actions RESTful.
En spécifiant [[yii \ rest \ ActiveController :: modelClass | modelClass]]
en tant que `app \ models \ User`, le contrôleur sait quel modèle peut être utilisé pour récupérer et manipuler des données.


## Configuration des règles d'URL <span id="configuring-url-rules"></span>


Ensuite, modifiez la configuration du composant `urlManager` dans la configuration de votre application:

```php
'urlManager' => [
    'enablePrettyUrl' => true,
    'enableStrictParsing' => true,
    'showScriptName' => false,
    'rules' => [
        ['class' => 'yii\rest\UrlRule', 'controller' => 'user'],
    ],
]
```


La configuration ci-dessus ajoute principalement une règle d’URL pour le contrôleur `utilisateur` afin que les données de l’utilisateur
peut être consulté et manipulé avec de jolies URL et des verbes HTTP significatifs.


## Activation de l'entrée JSON <span id="enabling-json-input"></span>


Pour que l'API accepte les données d'entrée au format JSON, configurez la propriété [[yii \ web \ Request :: $ parsers | parsers]] de
le `request` [composant d'application] (structure-application-components.md) pour utiliser le [[yii \ web \ JsonParser]] pour l'entrée JSON:

```php
'request' => [
    'parsers' => [
        'application/json' => 'yii\web\JsonParser',
    ]
]
```

> Info: La configuration ci-dessus est facultative. Sans la configuration ci-dessus, l'API ne reconnaîtrait que
  Formats d'entrée `application / x-www-form-urlencoded` et` multipart / form-data`.


## Vérification <span id="trying-it-out"></span>

À partir des informations précédentes, vous avez déjà terminé votre tâche de création des API RESTful.
pour accéder aux données de l'utilisateur. Les API que vous avez créées incluent:

* `GET / users`: liste tous les utilisateurs page par page;
* `HEAD / users`: affiche les informations générales de la liste d'utilisateurs;
* `POST / utilisateurs`: créer un nouvel utilisateur;
* `GET / users / 123`: renvoie les détails de l'utilisateur 123;
* `HEAD / users / 123`: affiche les informations générales de l'utilisateur 123;
* `PATCH / users / 123` et` PUT / users / 123`: met à jour l'utilisateur 123;
* `DELETE / users / 123`: supprime l'utilisateur 123;
* `OPTIONS / users`: affiche les verbes supportés concernant le noeud final` / users`;
* `OPTIONS / users / 123`: affiche les verbes supportés concernant le noeud final` / users / 123`.

> Info: Yii mettra automatiquement plusieurs noms de contrôleur à utiliser dans les terminaux.
> Vous pouvez configurer ceci en utilisant la propriété [[yii \ rest \ UrlRule :: $ pluralize]] -.


Vous pouvez accéder à vos API avec la commande `curl` comme suit:

```
$ curl -i -H "Accept:application/json" "http://localhost/users"

HTTP/1.1 200 OK
...
X-Pagination-Total-Count: 1000
X-Pagination-Page-Count: 50
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://localhost/users?page=1>; rel=self, 
      <http://localhost/users?page=2>; rel=next, 
      <http://localhost/users?page=50>; rel=last
Transfer-Encoding: chunked
Content-Type: application/json; charset=UTF-8

[
    {
        "id": 1,
        ...
    },
    {
        "id": 2,
        ...
    },
    ...
]
```


Essayez de changer le type de contenu acceptable en `application / xml`, et vous verrez le résultat
est retourné au format XML:

```
$ curl -i -H "Accept:application/xml" "http://localhost/users"

HTTP/1.1 200 OK
...
X-Pagination-Total-Count: 1000
X-Pagination-Page-Count: 50
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://localhost/users?page=1>; rel=self, 
      <http://localhost/users?page=2>; rel=next, 
      <http://localhost/users?page=50>; rel=last
Transfer-Encoding: chunked
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<response>
    <item>
        <id>1</id>
        ...
    </item>
    <item>
        <id>2</id>
        ...
    </item>
    ...
</response>
```


La commande suivante créera un nouvel utilisateur en envoyant une demande POST avec les données de l'utilisateur au format JSON:

```
$ curl -i -H "Accept:application/json" -H "Content-Type:application/json" -XPOST "http://localhost/users" -d '{"username": "example", "email": "user@example.com"}'

HTTP/1.1 201 Created
...
Location: http://localhost/users/1
Content-Length: 99
Content-Type: application/json; charset=UTF-8

{"id":1,"username":"example","email":"user@example.com","created_at":1414674789,"updated_at":1414674789}
```

> Tip: Vous pouvez également accéder à vos API via un navigateur Web en entrant l'URL `http: // localhost / users`.
  Cependant, vous aurez peut-être besoin de plugins de navigateur pour envoyer des en-têtes de requête spécifiques.

Comme vous pouvez le constater, les en-têtes de réponse contiennent des informations sur le nombre total, le nombre de pages, etc.
Il existe également des liens qui vous permettent de naviguer vers d'autres pages de données. Par exemple, `http: // localhost / users? Page = 2`
vous donnerait la page suivante des données de l'utilisateur.


En utilisant les paramètres `fields` et` expand`, vous pouvez également spécifier quels champs doivent être inclus dans le résultat.
Par exemple, l'URL `http: // localhost / users? Fields = id, email` renverra uniquement les champs` id` et `email`.


> Info: Vous avez peut-être remarqué que le résultat de `http: // localhost / users` inclut des champs sensibles 
> tels que `password_hash`,` auth_key`. Vous ne voulez certainement pas que cela apparaisse dans le résultat de votre API.
> Vous pouvez et devez filtrer ces champs comme décrit dans la section [Ressources] (rest-resources.md).


## Résumé <span id="summary"></span>


À l'aide de la structure de l'API Yii RESTful, vous implémentez un noeud final d'API en termes d'action de contrôleur, et vous utilisez
un contrôleur pour organiser les actions qui implémentent les noeuds finaux pour un seul type de ressource.

Les ressources sont représentées sous forme de modèles de données issus de la classe [[yii \ base \ Model]].
Si vous travaillez avec des bases de données (relationnelles ou NoSQL), il est recommandé d'utiliser [[yii \ db \ ActiveRecord | ActiveRecord]].
représenter des ressources.

Vous pouvez utiliser [[yii \ rest \ UrlRule]] pour simplifier le routage vers vos points de terminaison d'API.

Bien que cela ne soit pas obligatoire, il est recommandé de développer vos API RESTful en tant qu'application distincte,
votre Web front-end et back-end pour faciliter la maintenance.
