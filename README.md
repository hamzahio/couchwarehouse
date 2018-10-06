# couchwarehouse

*couchwarehouse* is a command-line tool that turns your [Apache CouchDB](http://couchdb.apache.org/) database(s) into a data warehouse. It works by:

- discovering the "schema" of your CouchDB database
- creating a new local [SQLite](https://www.sqlite.org/index.html) table to match the schema
- downloading all the documents (except design documents) and inserting one row per document into SQLite
- continuously monitoring CouchDB for new changes

## Installation

[Node.js and npm](https://nodejs.org/en/) are required:

```sh
npm install -g couchwarehouse
```

## Running

By default, your CouchDB installation is expected to be on "http://localhost:5984". Override this with the `--url`/`-u` parameter and specify the database name with `--database`/`-db`:

```sh
$ couchwarehouse --url https://U:P@host.cloudant.com --db mydb
Run the following command to query your data warehouse:

  $ sqlite3 couchwarehouse.sqlite

Then in sqlite3, you can run queries e.g.:

  sqlite3> SELECT * FROM cities LIMIT 10;

Have fun!
p.s Press ctrl-C to stop monitoring for further changes
downloading mydb [======------------------------] 20% 27.7s
```

After downloading is complete, *couchwarehouse* will continuously poll the target database for any changes and update the database accordingly. 

Press "Ctrl-C" to exit.

## Accessing the data warehouse

Simply run the `sqlite3` command-line tool which may be pre-installed on your computer, otherwise [download here](https://www.sqlite.org/download.html).

```sh
$ sqlite3 couchwarehouse.sqlite
sqlite3> SELECT name,latitude,longitude,country,population FROM mydb LIMIT 10;
name                    latitude    longitude   country     population
----------------------  ----------  ----------  ----------  ----------
Brejo da Madre de Deus  -8.14583    -36.37111   BR          27369.0   
Pindaré Mirim           -3.60833    -45.34333   BR          22933.0   
Moju                    -1.88389    -48.76889   BR          21510.0   
Matriz de Camaragibe    -9.15167    -35.53333   BR          18705.0   
Fatikchari              22.68768    91.78123    BD          33200.0   
Picos                   -7.07694    -41.46694   BR          57495.0   
Balsas                  -7.5325     -46.03556   BR          68056.0   
Jaguaruana              -4.83389    -37.78111   BR          21790.0   
Pilar                   -9.59722    -35.95667   BR          30617.0   
Patos                   -7.02444    -37.28      BR          92575.0 
```

SQLite has an [extensive query language](https://www.sqlite.org/lang.html) including aggregations, joins and much more. You could create warehouses on multiple CouchDB databases to create multiple SQLite tables and join them with queries!

```sql
SELECT country, SUM(population) FROM mydb GROUP BY 1 ORDER BY 2 DESC LIMIT 10;
```

## Schema discovery

CouchDB is a JSON document store and as such, the database does not have a fixed schema. The *couchwarehouse* utility takes a look at a handful of documents and *infers* a schema from what it sees. This is clearly only of use if your CouchDB documents that have similar documents.

Let's take a typical document that looks like this:

```js
{
  "_id": "afcc37fbe6ff4dd35ecf06be51e45724",
  "_rev": "1-d076609f1a507282af4e4eb52da6f4f1",
  "name": "Bob",
  "dob": "2000-05-02",
  "employed": true,
  "grade": 5.6,
  "address": {
    "street": "19 Front Street, Durham",
    "zip": "88512",
    "map": {
      "latitude": 54.2,
      "longitude": -1.5
    }
  },
  "tags": [
    "dev",
    "front end"
  ]
}
```

*couchwarehouse* will infer the following schema:

```js
{
  "id": "string",
  "rev": "string",
  "name": "string",
  "dob": "string",
  "employed": "boolean",
  "grade": "number",
  "address_street": "string",
  "address_zip": "string",
  "address_map_latitude": "number",
  "address_map_longitude": "number",
  "tags": "string"
}
```

Notice how:

- the sub-objects are "flattened" e.g. `address.map.latitude` --> `address_map_latitude`
- arrays are turned into strings
- _id/_rev become id/rev

The keys of the schema become the column names of the SQLite table.

## Removing unwanted SQLite tables

Unwanted tables can be easily removed using the `sqlite3` prompt:

```sh
sqlite> DROP TABLE mydb;
```

## What is this useful for?
