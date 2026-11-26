# GLOSS as a SKOS profile (skeleton)

Formal profile per the W3C Profiles Vocabulary (`prof:`).

- **Profile IRI**: `https://www.glossarist.org/ontologies/profile/skos`
- **Base vocabulary**: SKOS + SKOS-XL
- **Constraints (SHACL)**: prefLabel required per language represented; concept → skos:Concept + gloss:Concept dual typing; designation labels as skosxl:Label with gloss type + normative status.
- **Extensions (beyond SKOS)**: per-language LocalizedConcept structure; designation grammar and types; claim-level source citations; partitive member structure (presence × count); non-verbal definitions.
- **Mapping**: every GLOSS register projects losslessly to SKOS (prefLabels from preferred designations, hierarchies from generic relations); SKOS → GLOSS is partial by design (labels cannot carry grammar/status/provenance).
