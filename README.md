# kotori

> WORK IN PROGRESS. VERSION 2.0 IS ABOUT TO BE FIRST VERSION READY FOR PRODUCTION!

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

:hatching_chick: As you can see there's no typical hiearchy of keys. __Master keys__ are used only for project manipulations but they cannot be in the project context!

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

```
  +-----------+  +-----------------+  +------------+  +-----------+
->| project 1 |->| document type A |->| document 1 |->| version 0 |
  +-----------+  +-----------------+  +------------+  +-----------+
              |
              |  +-----------------+  +------------+  +-----------+
              .->| document type B |->| document 1 |->| version 0 |
                 +-----------------+  +------------+  +-----------+
                                   |
                                   |  +------------+  +-----------+
                                   .->| document 2 |->| version 0 |
                                      +------------+  +-----------+
                                                   |
                                                   |  +-----------+
                                                   .->| version 1 |
                                                      +-----------+
  +-----------+                                       
->| project 2 |
  +-----------+
```        

### Project

```yaml
instance: <string>
identifier: <string>
name: <string>
projectKeys: [ { key: <string>, isReadonly: <bool> }, ... ]
```

``instance`` - name of the instance (configured in Web API server)

``identifier`` - id of the project

``name``- human friendly name of the project

``projectKeys`` - a collection of project keys

Project itself is quite straightforwad. ``Name`` is just for future usage (web administration or something).
If you want to delete a project you need to delete or document types and documents referenced.

### Document type

```yaml
instance: <string>
projectId: <string>
identifier: <string>
type: <enum>
indexes: [ { from: <string>, to: <enum> }, ... ]

```

The fields of ``document type`` are not a big interest of you. Basically ``document type`` is auto created or updated. Buy keep in mind that all document operations can be performed against one particular document type.

``Identifier`` is basically a path like ``_content/movie`` or ``_data/directors``. ``Type`` is set based on the first segment of the path to: ``content`` or ``data``.

``Indexes`` are tricky. They are created automatically and you can't modifiy them anyway. In fact ``document type`` can be only requested to being ``get``, ``get all for project`` or ``deleted``. This is implemented by design. ``Indexexes`` are meant only for search operations (Azure Search). Because Azure Search has a limited index capabilities - max number of index fields or index themselves ``Kotori`` does so called ``index mappings``. We have one universal index and each document type maps to different fields accordingly. Plus there are some reserved indexes for properties like ``slug`` or ``date``.

Example:

```csharp
var meta = new
{
 food = new List<string> { "eggs", "ham", "potatoes" },
 clever = true,
 bad = new { foo = "bar" }
};
```

We will get mapping like that for example:

```
food -> tags2
clever -> flag1
```

As you can see ``bad`` is missing. Only primitive types like ``string``, ``bool``, ``int``, ... and ``a collection of strings`` are supported. The rest are auto omitted.

### Document

As we said there are two types of documents:
- ``content``
- ``data`` (not supported at the moment)

The main difference is that ``content`` is just a typical article contains **front matter** (aka meta) and **body** (typically markdown). You can set of course without front matter or without content but at least one part must be set. Front matter can be set as ``YAML`` or ``JSON``.

Article may look like this:
```markdown
---
title: hello
frontPage: !!bool true
---
Well...
```

:hatching_chick: As YAML standard says parsing of types is up to applications. Kotori doesn't use any type conversion at the moment. So you need to specify a proper ``tag`` (see above).

or with JSON front matter like that:
```markdown
---
{ "title": "hello", "frontPage": true }
---
Well...
```

Content will be typically in ``markdown`` format but you are free to use whatever you want (other template systems, ...).

#### Internal properties

Typically when using static generators you can specify ``date`` whith a special format of the document filename like that:

``2017-01-26-document.md``

But you can set in the front matter other so called internal properties.

|date |description|
|-----|-----------|
|``$date``|date
|``$slug``|slug (url)

All internal properties start with a char ``$`` and you can use only the ones listed above. If you try for example set ``$foo`` you will receive an error.

:hatching_chick: If you do not set date by naming your file or including an internal property current date will be used. All documents must have date set.

Slug is part of the URL for the document. Normally it's autogenerated from the filename (identification). Date part is removed and so an extension if present.

Examples:

```
abracadabra.md -> abracadabra
2017-01-01-abracadabra.md -> abracadabra
_2017-01-01-abracadabra.md -> abracadabra
```

But sometimes you would like to specify it.

Example ``2017-01-01-king-kong-1976.md``:

```markdown
---
$slug: king-kong
$date: 2017-01-02
---
Fluffy gorilla says Hi!
```

:hatching_chick: All properties are being "camel-cased" by default. So it doesn't matter if you set ``Title`` or ``title``. It means there are built-in checkups for duplicated properties. But keep in mind when you use searching to just use camel casing!
