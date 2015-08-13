# Cypher Examples

Selection of example Cypher queries for Histograph data. Expects a Neo4j instance with Histograph data from the following two sources:

- [https://github.com/histograph/data](https://github.com/histograph/data)
- [https://github.com/erfgoed-en-locatie/data](https://github.com/erfgoed-en-locatie/data)

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
// find source, target nodes
MATCH (m:_) WHERE m.id = "urn:hg:tgn:1047973"
MATCH (n:_) WHERE n.id = "urn:hg:tgn:7016845"

// find corresponding equivalence classes
OPTIONAL MATCH m <-[:`=`]- (mConcept:`=`)
OPTIONAL MATCH n <-[:`=`]- (nConcept:`=`)

// choose the right node (EC if there, otherwise only member)
WITH m, coalesce(nConcept, n) AS to,
     coalesce(mConcept, m) AS from

// ensure we have a path
MATCH p = allShortestPaths(from -[:`hg:liesIn`|`=`|`=i` * 1 .. 5]-> to)
RETURN p
```
