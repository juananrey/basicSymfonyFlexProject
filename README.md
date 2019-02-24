Default DDD PHP project With Docker + PHPUnit + PHPCS + Symfony Flex (as an infrastructure artifact)
----------------------------------------------------------------------------------------------

Basic skeleton containing just a `/_healthcheck` endpoint, and including:
- Dockerized startup (with PHP-FPM, check `/docker` folder and the `README.md` in there).
- PHPUnit (run `bin/phpunit` to pass the unit tests).
- PHPCS (symlink to `/vendor/bin/phpcs`, run `bin/phpcs` to run a code sniffer based on PSR2 standard).

If you want to include Redis in your project, run `docker-compose -f docker-compose-with-redis.yml up -d`

## Requirements
- Docker

## Installation
1) On the root folder, run: `docker-compose up -d` and wait for all required packages to install. 
2) Get into the machine (`docker exec -ti mylocalproject-php-fpm bash`) and run a `composer install` from inside
3) Go to [localhost:8000/_healthcheck](http://localhost:8000/_healthcheck), where you should be able to see this output:
```json
{
    "status": "i am ok"
}
```

Now, it is completely up to you how do you want your project to grow :)

## Going to production
If you consider the dockerized environment good enough to production, remember these two steps:
1) Run `composer install --no-dev --classmap-authoritative --apcu-autoloader`
2) Remember to change `APP_ENV` to `prod` within the `.env` file.

You can see a reasonably good performance by executing
[Apache Bench tool](https://httpd.apache.org/docs/2.4/programs/ab.html),
with around 100s of requests in 10s concurrent:
`ab -n 100 -c 10 http://localhost:8000/_healthcheck`

## Project architecture
This project is architectured according to a DDD + CQRS pattern, which will contain three main parts:
- __Domain__: represents business concepts and business logic. It should not know anything about the other parts
- __Application__: defines jobs and orchestrate domain objects to solve problems. It just can know about `Domain`
- __Infrastructure__: responsible of Ui (Controllers) / Console / 3rd parties concerns. 
This can know about `Application` and `Domain`.

## Project structure
Extending the previous architecture, this is the main folder structure within the project

```
Domain
└─── Exception: Contains `AppException`, responsible for any exception about the application which
|                will contain an internal error code, plus a collection of error codes along the app
|
└─── Model: Core business logic within the aplication, which will also contain validations
|
└─── Repository: Interfaces to access the data model that shall be viewed as a Collection,  with a composition as follows:
     └─── `findOfId`: Find a specific Model from a given unique ID. Ideally returns a Domain exception when not found
     └─── `findOfXxx`: Find the Model through an unique ID
     └─── `save`: responsible of create/update the model
     └─── `delete`: responsible of delete the model from the collection


Application
└─── Command: Executes an use case by orchestrating domain objects, and ideally produces an output in the shape of an event
|
└─── Query: Interfaces of specialized queries (anythin different of a `findOfId`) to the model.
     |      Note: those are part of the Application because the Domain should not be aware at all about the expected Responses
     └─── Responses: Set of Value Objects which we expect to obtain as a result of the query, ideally implementing a `Serializable`


Infrastructure
└───  EventListener: Even though those could be part of the Application layer (as actors regarding different events), 
|                    right now they are an infrastructure concern as the infra too is implemented as an Event schema.
|
└───  Query: Contains folders with the different sources of implementation per query (e.g., `MySqlPremiumUsersQuery`)
|
└───  Repository: Contains folders with the different sources of implementation per repository (e.g, `MySqlUsersRepository`)
|
└───  Service: Third-party specific implementations of services and connectors (REST, MongoDB, Stripe...)
|
└───  Ui: Contains the user interface communication, mainly Console commands (cron/daemons) and Web commands (controllers)          
 
 
```