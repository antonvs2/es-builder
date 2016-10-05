# Elasticsearch Query Builder

[![Build Status](https://travis-ci.org/antonvs2/es-builder.svg?branch=master)](https://travis-ci.org/antonvs2/es-builder)

Elasticsearch query builder for Node.js, build compatible queries with the Elasticsearch 2.x DSL. Because creating complex queries using the Query DSL is a pain.

It just build the `query` element within the search request body, this means that parameters like `size` or `from` must be added separately, as well as the likes of `sort`.

# Install

```js
npm install es-builder
```

# Features

- No production dependencies
- Chainable methods
- `built` getter method returns each time a copy of the object so it can be safely passed to foreign code

# Usage

```js
const QueryBuilder = require('./es-builder').QueryBuilder;
const leafQueries = require('./es-builder').leafQueries;

const qb = new QueryBuilder();
qb.query(leafQueries.termQuery('name', 'Kirby'))
  .query(leafQueries.matchQuery('description', 'Pink, fluffy and very hungry'))
  .queryMustNot(leafQueries.termQuery('name', 'Waddle Dee'));

const query = qb.built;
// {
//   bool: {
//     must: [{
//       term: {
//         name: 'Kirby'
//       }
//     }, {
//       match: {
//         description: {
//           query: 'Pink, fluffy and very hungry'
//         }
//       }
//     }],
//     must_not: {
//       term: {
//         name: 'Waddle Dee'
//       }
//     }
//   }
// }
```

Adding clauses to filter context is possible as well:

```js
qb.filter(leafQueries.termQuery('name', 'Kirby'));

const query = qb.built;
// {
//   bool: {
//     filter: {
//       bool: {
//         must: {
//           term: {
//             name: 'Kirby'
//           }
//         }
//       }
//     }
//   }
// }
```

NOTE: The above query is quite simple, so using the Query DSL directly it could be written shorter:

```json
{
  "bool": {
    "filter": {
      "term": {
        "name": "Kirby"
      }
    }
  }
}
```

The query being executed is not going to change, therefore using this utility becomes long for the sake of keeping the code as simple as possible.

## Shortcut for leaf queries

There is a shortcut available for leaf queries, inspired by [elasticsearch-dsl-py](https://github.com/elastic/elasticsearch-dsl-py)

`Q('terms', 'name', ['Kirby', 'Metaknight'])` and `termsQuery('name', ['Kirby', 'Metaknight'])` give the same result:

```js
{
  terms: {
    name: ['Kirby', 'Metaknight']
  }
}
```

Also, there is a one-to-one mapping between the raw query and its equivalent in the DSL, therefore adding directly raw queries is ok.

## Complex queries

Combined queries can be built nesting compound queries.

```js
const QueryBuilder = require('./es-builder').QueryBuilder;
const compoundQueries = require('./es-builder').compoundQueries;
const Q = require('./es-builder').leafQueries.shortcut;

const qb = new QueryBuilder();

qb.filter(Q('terms', 'name', ['Kirby', 'Metaknight'])).filter(Q('exists', 'age'));

const boolQuery = new compoundQueries.BoolQuery();
boolQuery.should(Q('range', 'age', 20, 25)).should(Q('prefix', 'surname', 'Pi'));

qb.filter(boolQuery.built);
const query = qb.built;
// {
//   bool: {
//     filter: {
//       bool: {
//         must: [{
//           terms: {
//             name: ['Kirby', 'Metaknight']
//           }
//         }, {
//           exists: {
//             field: 'age'
//           }
//         }, {
//           bool: {
//             should: [{
//               range: {
//                 age: { gt: 20, lt: 25 }
//               }
//             }, {
//               prefix: {
//                 surname: 'Ki'
//               }
//             }]
//           }
//         }]
//       }
//     }
//   }
// }
```

## Aliases

There are aliases available for some methods.

`const qb = new QueryBuilder();`
- `qb.query()` → `qb.queryAnd()`
- `qb.queryMustNot()` → `qb.queryNot()`
- `qb.queryShould()` → `qb.queryOr()`
- `qb.filter()` → `qb.filterAnd()`
- `qb.filterMustNot()` → `qb.filterNot()`
- `qb.filterShould()` → `qb.filterOr()`

`const boolQuery = new compoundQueries.BoolQuery();`
- `boolQuery.must()` → `boolQuery.and()`
- `boolQuery.mustNot()` → `boolQuery.not()`
- `boolQuery.should()` → `boolQuery.or()`

# API

Coming soon. 

At the moment you can take a look to the tests to see how all the methods work.

# Compatibility

- Only compatible with Elasticsearch 2.x search API
- ES2015 usage, not available for old versions of Node.js

# ToDo List

- Leaf queries like `multi_match` or `fuzzy`
- Compound queries like `constant_score` or `dis_max`
- Possibility to pass some options to leaf queries (like `boost`)
- Transpile to ES5 to make it compatible with old versions
- REPL
- Browser compatible
- And more

Pull requests or any comments are welcome.
