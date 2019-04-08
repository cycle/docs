# Cycle ORM - Documentation
[![Latest Stable Version](https://poser.pugx.org/cycle/orm/version)](https://packagist.org/packages/cycle/orm)
[![Build Status](https://travis-ci.org/cycle/orm.svg?branch=master)](https://travis-ci.org/cycle/orm)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/cycle/orm/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/cycle/orm/?branch=master)
[![Codecov](https://codecov.io/gh/cycle/orm/graph/badge.svg)](https://codecov.io/gh/cycle/orm)

Cycle is PHP DataMapper and ORM engine designed to work in long-running PHP applications (like [RoadRunner](https://github.com/spiral/roadrunner)).

Table of Content
----------------
* About Cycle
  * [About Cycle ORM](intro/about.md)
  * [Installation](into/installation.md)
  * [Configuration](info/configuration.md)
  * [Contributing](into/contributing.md)
  * [LICENSE](license.md)
* Annotated Entities
  * [Prerequisites](annotated/prerequisites.md)
  * [Entities](annotated/entity.md)
  * [Relations](annotated/relations.md)
* Basics
  * [Mapping](basic/mapping.md)
  * [Create, Update, Delete](basic/crud.md)
* Relations
  * [Has One](relation/has-one.md)
  * [Has Many](relation/has-many.md)
  * [Belongs To](relation/belongs-to.md)
  * [Refers To](relation/refers-to.md)
  * [Many Thought Many](relation/many-though-many.md)
* Repositories
  * [Select Entity](repository/select.md)
  * [Relations](repository/relations.md)
  * [Custom Repository](repository/custom.md)
* Query Builder
  * [Basics](query-builder/basic.md)
  * [Complex Conditions](query-builder/complex.md)
  * [Expressions](query-builder/expressions.md)
  * [Relations](query-builder/relations.md)
* Architecture
  * [Overview](architecture/overview.md)
  * [Heap and States](architecture/heap.md)
  * [Mappers](architecture/mapper.md)
  * [Commands](architecture/command.md)
  * [Transactions (Unit of Work)](architecture/transaction.md)
  * [Sources and Databases](architecture/source.md)
  * [References, Promises and Proxies](architecture/promise.md)
  * [Query Builder](architecture/query-builder.md)
  * [Constrains](architecture/constrain.md)
  * [Node Parser](architecture/node-parser.md)
* Advanced Usage
  * [Single Table Inheritance](how-to/single-table-inheritance.md)
  * [Custom Mappers](how-to/custom-mapper.md)
  * [Dynamic Schema (StdClass)](how-to/dynamic-schema.md)
  * [Custom Relation](how-to/custom-reation.md)
  * [Entity Constrains](how-to/constrain.md)
  * [Caching](how-to/caching.md)
  * [References and Proxies](how-to/references.md)
  * [Testing](how-to/testing.md)
  * [Reconnects and Database Exceptions](how-to/exception.md)
* Components
  * [Data Mapper Core](component/core.md)
  * [Schema Builder](component/schema-builder.md)
  * [Annotations Parser](component/annotated.md)
  * [Migrations](component/migrations.md)
  * [Proxy Factory](component/proxy-factory.md)
