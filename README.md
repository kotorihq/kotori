# kotori

Hybrid CMS based on headless systems/static generators.

## Stack

- Database
  - CosmosDB/DocumentDB

- Elastic search (optional)
  - Azure Search 

- Blob storage (optional)
  - Azure storage

## Components

- [kotori-core](https://github.com/kotorihq/kotori-core) - core of the system, .NET Core
- [kotori-sample-data](https://github.com/kotorihq/kotori-sample-data) - sample (static) project data 
- [kotori-server](https://github.com/kotorihq/kotori-server) - server REST API, .NET Core
- [kotori-cli-go](https://github.com/kotorihq/kotori-cli-go) - CLI, Go
- [kotori-cli](https://github.com/kotorihq/kotori-cli) - CLI, .NET Core [ON HOLD]

## Main principles

Well, we are not secret agents flying in cloaked spaceships so we are not in need of some ultra heavy auth system. That's why we use __simple API keys__.

Auth business is being done in Web API only.

## Key types

### Master key

Master keys are configured for instance of Web API.

Master keys allow you:
- Create a new project
- Delete an existing project
- Add/delete/modify project keys

### Project key

Project keys are stored in database. They are two types of project keys:
- full access keys
  - add/modify/delete documents
  - get/search documents
- readonly keys
  - get/search documents

## Web API server configuration

You can use "classic" configuration file [``appsettings.json``](https://raw.githubusercontent.com/kotorihq/kotori-server/master/KotoriServer/appsettings.json). ENV variables are supported too. So you can set easily application settings like this:

```
Kotori__Configuration__Instance           MyInstance
Kotori__Configuration__MasterKeys__0__Key Foo
etc.
```

If you change instance name you won't be able to access any documents. There's no trigger implemented for taking care of it anyhow.

## Domains

Each running Web API instance is identified by so called ``instance``. All records stored in database are thus identified with ``instance`` name as well. You can have more ``instances`` configured against the same CosmosDb/DocumentDb collection.

### Project

```yaml
instance: <string>,
name: <string>
identifier: <string>
projectKeys: [ { key: <string>, isReadonly: <bool> }, ... ]
```

``instance`` - name of the instance (configured in Web API server)

``name``- human friendly name of the project

``identifier`` - id of the project

``projectKeys`` - a collection of project keys



