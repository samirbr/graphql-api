Symfony-doctrine-graphql-api
=====

# Description
This library provides an single and simple entry point to read all your db using Graphql syntax.
To achieve that we will need to export our db model into php entities, which will be represented
  by doctrine annotations, taking advantage of this doctrine annotations this library will generate
  the graphql schema to accessed using Graphql query syntax.

## Setup

### Include repository in `composer.json`

```
"repositories": [
....
    {
        "type": "git",
        "url": "https://github.com/ggarri/symfony-doctrine-graphql"
    },
....

"require": {
    "ggarri/symfony-graphql-api": "^0.6-dev"
}
....
```

### Installing dependencies
```
composer install/update
```

### Loading bundle
```
# AppKernel.php

use GraphqlApiBundle\GraphqlApiBundle;
...
public function registerBundles()
    {
        $bundles = array(
            ....
            new GraphqlApiBundle(),
....
```

### Routing
Including graphql entry point in 'app/config/routing.yml'
```
# Adding graphql routes (127.0.0.1:8000/graphql/api?query={graphql_query}
graphql_routing:
    resource: .
    type: extra
```


#### Setup db connection
If you didn't do it yet you will need to set up doctrine database connection This library will only require an valid doctrine connection as the one above:
```
doctrine:
    dbal:
        driver:   "%database_driver%"
        host:     "%database_host%"
        port:     "%database_port%"
        dbname:   "%database_name%"
        user:     "%database_user%"
        password: "%database_password%"
        charset:  UTF8

    orm:
        default_entity_manager: default
        auto_generate_proxy_classes: "%kernel.debug%"
        naming_strategy: doctrine.orm.naming_strategy.underscore
        auto_mapping: true
```

## Importing your DB Model

Once everything is setup you can start converting your db into php entity classes.
For that, and as it was mentioned above, your db model will be imported from connection defined under Doctrine.dbal.default.

The above command will do all the work for you.  

```
php app/console graphql-api:graphql:generate-schema --namespace "AppBundle\\Entity" annotation ./src/AppBundle/Entity --from-database --force
```

(*) we had use `"AppBundle\\Entity"` as namespace for model to be imported, and `./src/AppBundle/Entity` 
as location folder to place each of the entity files.
(*) Possible problems appear above


## Usage

That is all, it is time to start using start reading your db with your new custimized graphql api.

(We used `127.0.0.1:8000` as server, you might have it on different name)

#### Curl sample
```
curl -XPOST -H 'Content-Type:application/graphql' -d 'query={  boardtype(id: 2) {  id, name, vatClass { amount }  } }' http://127.0.0.1:8000/graphql/api
```

#### Browser sample
```
http://127.0.0.1:8000/graphql/api?query={boardtype(id:2){id,name,vatClass{amount}}}
```

If you want to know more about Graphql syntax, go to: http://graphql.org/learn/

# Troubleshooting

## Using POSTGRES 9.5
In case your db is Postgres and includes `Blob` or `jsonb` types you might need to patch the 
postgres driver due to `Blob` or `jsonb` wasn't included in latest version of driver.

#### Patching Postgres driver to include new types
``` 
cp -r vendor/ggarri/graphql-api/src/Doctrine src/
patch vendor/doctrine/dbal/lib/Doctrine/DBAL/Driver/AbstractPostgreSQLDriver.php vendor/ggarri/graphql-api/src/Doctrine/patch/AbstractPostgreSQLDriver.patch
```
(*) Please, check if patch was applied correctly, because some types it does't.
Around line 110 it has to have the following lines:
```
switch(true) {
    case version_compare($version, '9.4', '>='):
        return new PostgreSQL95Platform();  // THIS IS THE IMPORTANT ONE
    case version_compare($version, '9.2', '>='):
        return new PostgreSQL92Platform();
    case version_compare($version, '9.1', '>='):
        return new PostgreSQL91Platform();
    default:
        return new PostgreSqlPlatform();
}
```
