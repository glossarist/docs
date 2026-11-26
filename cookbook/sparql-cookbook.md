# SPARQL cookbook for glossarist registers

All queries run against `/sparql` (read-only). Default prefixes (gloss, skos, skosxl, dcterms, rdf, rdfs, xsd) are predeclared.

## 1. Preferred designations in two languages
```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?id ?en ?fr WHERE {
  ?c gloss:identifier ?id .
  ?c skos:prefLabel ?en . FILTER (lang(?en) = "eng")
  ?c skos:prefLabel ?fr . FILTER (lang(?fr) = "fra")
} ORDER BY ?id
```
## 2. Concepts superseded between editions
```sparql
SELECT ?from ?to WHERE { ?from gloss:supersededBy ?to }   -- requires lineage data; endpoints report coverage honestly when empty
```
## 3. Entries without an authoritative source
```sparql
SELECT ?c WHERE {
  ?c a gloss:Concept .
  FILTER NOT EXISTS { ?c gloss:source ?s }
}
```
## 4. Partitive members with required presence
```sparql
SELECT ?whole ?member WHERE {
  ?whole gloss:hasPartitiveRelation ?rel .
  ?rel gloss:hasMember ?member .
  ?member gloss:presence gloss:partitivePresence#required
}
```
## 5. Status inventory
```sparql
SELECT ?status (COUNT(?c) AS ?n) WHERE { ?c gloss:status ?status } GROUP BY ?status
```
(6–20: multilingual coverage gaps, deprecated designations, clause lookups via citations, edition diffs via identifiers, relations-by-type, incoming relations, formula-bearing definitions, figure references, source-less designations, per-edition growth, register-of-registers.)
