---
title: "Building Sento Crawler"
date: 2019-03-31T22:18:33+02:00
draft: false
---

During the development of Sento Crawler I had to face some technical
challenges and take important steps towards the definition and implementation
of the data pipeline.

# The technical goals

At the start of development a initial set of requirements were set:

- **Easily configurable**: the user should not need to read the code
  or modify it in order to set up an instance.
- **Asynchronous**: the tool was expected to perform heavy usage of
  network communications, asynchronous programming was a strong requirement,
  as we always want loose no time on blocked I/O operations.
- **Geospatial ready**: the ultimate goal in Sento is to produce a geospatial
  application, geometric data also needs to be stored.

# Core tools used

The following libre software was used to develop Sento Crawler:

- [Peony Twitter](https://github.com/odrling/peony-twitter): an async Python
  wrapper over Twitter's public HTTP API. This tool was used to perform
  requests to Twitter's API and abstract from certain aspects such as rate
  limiting and response processing.
- [asyncpg](https://github.com/MagicStack/asyncpg): an async Python database
  interface for PostgreSQL databases. Used to connect to Sento's database and
  perform read/write queries.
- [alembic](https://github.com/sqlalchemy/alembic): a database migrations tool
  to be used in conjunction with SQLAlchemy. Used to define with Python
  classes all the data models required by Sento.

# Difficulties to solve

During the development of this tool I had to face these difficulties:

## Using Twitter's Search endpoint

In order to retrieve tweets from different locations,
[Twitter](https://developer.twitter.com/en/docs/tweets/search/api-reference/get-search-tweets)
requires to pass as parameters the longitude and the latitude of a point and a
radius in kilometers, with both parameters you can specify a _search area_.
However one cannot determine a radius manually for each location,
Crawler is meant to work with minimal input from the user.
In order to determine the most appropiate _search area_ for each location
I used OpenStreetMap's Nominatim API. Thanks to the data provided by
its contributors I could determine the most appropiate parameters.

Nominatim is a geocoding service that allows you to retrieve locations'
geometries providing their names. Once the Crawler obtained a location's
geometry, the center and the radius of the minimum bounding circle is
calculated and stored in the database, this calculation only needs to be
performed the first time. The center and the radius is later used when
sending the request to Twitter's Search endpoint.

## Avoiding lost of data when updating models

Crawler retrieves large ammounts of data, updating the data model implies
that an instance must be stopped in order to securely backup data. With the
objective to avoid those problems and keep the system running the longest
possible a migration system was employed. With Alembic the data model
was defined in easy to understand and mantain Python classes.

# How Sento Crawler works

The following flowchart describes how Crawler works:

![Diagram](/sento-blog/images/crawler-diagram.svg)
