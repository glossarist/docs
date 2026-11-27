# SPARQL cookbook for glossarist registers

Endpoint: `/sparql` (read-only, cost-limited: wall-clock deadline 4 s, solution ceiling 200 000).
Results: SPARQL 1.1 JSON results for SELECT/ASK; Turtle for CONSTRUCT.
Prefixes are **not** predeclared — declare them in each query.
RDF 1.1 normalization applies: language tags are lowercased, simple literals are `xsd:string`.

Engine: glossarist-sparql v0.1.x (roll-our-own; no third-party engine). Every query below was
executed against `https://vocab.oimlsmart.org/sparql` on 2026-11-27 unless marked otherwise.
Queries marked **[≥ 0.1.1]** need the 0.1.1 engine (deployed build) — verified locally, pending redeploy.

Concepts in the ingested OIML corpora are typed `skos:Concept` and carry
`skos:prefLabel`, `skos:altLabel`, `skos:definition`, `skos:scopeNote`, `skos:notation`.

## 1. Core lookups

All English preferred labels, alphabetical:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?c ?label WHERE {
  ?c a skos:Concept .
  ?c skos:prefLabel ?label . FILTER (lang(?label) = "eng")
} ORDER BY ?label LIMIT 20
```

Lookup by notation (e.g. an OIML clause key), with its type:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?n ?t WHERE {
  ?c skos:notation ?n .
  ?c a ?t .
} LIMIT 5
```

French labels (golden-suite case):

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?l WHERE {
  ?c skos:prefLabel ?l .
  FILTER (lang(?l) = "fra")
} LIMIT 3
```

## 2. Combining patterns

Union across label types and languages:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?l WHERE {
  { ?c skos:prefLabel ?l . FILTER (lang(?l) = "fra") }
  UNION
  { ?c skos:altLabel ?l . FILTER (lang(?l) = "eng") }
} LIMIT 10
```

Definitions and notations that may be absent — `OPTIONAL` keeps the label row:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?c ?d ?n WHERE {
  ?c skos:prefLabel ?l . FILTER (lang(?l) = "eng")
  OPTIONAL { ?c skos:definition ?d }
  OPTIONAL { ?c skos:notation ?n }
} LIMIT 10
```

Parameterize a language with `VALUES`:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?c ?l WHERE {
  VALUES ?lang { "fra" }
  ?c skos:prefLabel ?l .
  FILTER (lang(?l) = ?lang)
} LIMIT 10
```

## 3. Filtering

Substring/anchored matching with `regex` (case-insensitive flag):

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?c ?l WHERE {
  ?c skos:prefLabel ?l .
  FILTER (regex(?l, "^quantity", "i"))
} LIMIT 10
```

Range comparisons on notation strings **[≥ 0.1.1]**:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?c ?n WHERE {
  ?c skos:notation ?n .
  FILTER (?n >= "b3-1991-3")
} LIMIT 10
```

Concepts without a scope note **[≥ 0.1.1]**:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?c ?l WHERE {
  ?c skos:prefLabel ?l . FILTER (lang(?l) = "eng")
  OPTIONAL { ?c skos:scopeNote ?s }
  FILTER (!bound(?s))
} LIMIT 10
```

## 4. Query forms

Existence check (SPARQL 1.1 JSON: `{"head":{},"boolean":true}`):

```sparql
ASK { ?c a skos:Concept }
```

Template projection to Turtle **[≥ 0.1.1]**:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
CONSTRUCT { ?c skos:prefLabel ?l }
WHERE {
  ?c skos:notation ?n .
  ?c skos:prefLabel ?l . FILTER (lang(?l) = "eng")
} LIMIT 10
```

## 5. Engine roadmap (honest non-coverage)

Not implemented in v0.x — do not use yet (the endpoint returns a Problem Details syntax error):

property paths · aggregates (`COUNT`/`SUM`…) · `GROUP BY`/`HAVING` · projection expressions (`(str(?x) AS ?y)`) ·
subqueries · `MINUS` · `EXISTS`/`NOT EXISTS` · `BIND`/`IF`/`COALESCE` ·
`FROM`/`FROM NAMED`/`GRAPH` · `DESCRIBE` · blank-node syntax (`[]`, `(...)`) ·
string functions beyond `str` · SPARQL UPDATE (the endpoint is read-only by design).

Writes belong to the register procedure (FERIN proposals), not the query endpoint.
