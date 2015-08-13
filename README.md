# Cypher Examples

Selection of example Cypher queries for Histograph data. Expects a Neo4j instance with Histograph data from the following two sources:

- [https://github.com/histograph/data](https://github.com/histograph/data)
- [https://github.com/erfgoed-en-locatie/data](https://github.com/erfgoed-en-locatie/data)

Example queries:

- [Types per dataset](#types-per-dataset)
- [Nieuw-Amsterdam, Nederland](#nieuw-amsterdam-nederland)
- [Municipalities absorbed by Berkelland](#municipalities-absorbed-by-berkelland)
- [Places inside Municipality of Emmen](#places-inside-municipality-of-emmen)

## Types per dataset

```cypher
MATCH
  (p:_)
WHERE NOT p:_Rel AND NOT p:`=` AND NOT p:`=i`
RETURN DISTINCT
  p.dataset AS dataset,
  p.type AS type,
  COUNT(p) AS count
```

## Nieuw-Amsterdam, Nederland

```cypher
// find source and target nodes
MATCH (m:_) WHERE m.id = "urn:hg:tgn:1047973"
MATCH (n:_) WHERE n.id = "urn:hg:tgn:7016845"

// find corresponding equivalence classes (ECs)
OPTIONAL MATCH m <-[:`=`]- (mConcept:`=`)
OPTIONAL MATCH n <-[:`=`]- (nConcept:`=`)

// choose the right node (EC if there, otherwise only member)
WITH coalesce(nConcept, n) AS to,
     coalesce(mConcept, m) AS from

// ensure we have a path
MATCH p = allShortestPaths(from -[:`hg:liesIn`|`=`|`=i` * 1 .. 5]-> to)
RETURN p
```

## Municipalities absorbed by Berkelland

```cypher
// find target nodes
MATCH (n:_) WHERE n.id = "urn:hg:gemeentegeschiedenis:Berkelland"

// find corresponding equivalence classes (ECs)
OPTIONAL MATCH n <-[:`=`]- (nConcept:`=`)

// choose the right node (EC if there, otherwise only member)
WITH coalesce(nConcept, n) AS to

// find `hg:absorbedBy` paths
MATCH p = allShortestPaths(from -[:`hg:absorbedBy`|`=`|`=i` * 1 .. 5]-> to)

// fetch all nodes in path
UNWIND(nodes(p)) as k
MATCH k

// select only non-relation nodes
WHERE NOT k:_Rel AND NOT k:`=`

RETURN DISTINCT k.name, k.id, k.validSince, k.validUntil
ORDER BY k.validUntil
```

## Places inside Municipality of Emmen

```cypher
// find target nodes
MATCH (n:_) WHERE n.id = "urn:hg:geonames:2756134"

// find corresponding equivalence classes (ECs)
OPTIONAL MATCH n <-[:`=`]- (nConcept:`=`)

// choose the right node (EC if there, otherwise only member)
WITH coalesce(nConcept, n) AS to

// find paths
MATCH (place:`hg:Place`) -[:`hg:liesIn`|`=`|`=i` * 3 .. 3]-> to
RETURN DISTINCT place.name, place.id
```
