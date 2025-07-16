
# Epidemiological Surveillance using Neo4j

This project explores graph-based analysis of disease spread using Neo4j. By leveraging graph algorithms, we can uncover insights about contact tracing, individual importance, and community structures.

## Graph Schema Discovery

```cypher
CALL db.schema.visualization();
CALL db.labels();
CALL db.relationshipTypes();

MATCH (n)
WITH DISTINCT labels(n) AS labels, keys(n) AS properties
RETURN labels, properties;

MATCH ()-[r]->()
WITH DISTINCT type(r) AS relationshipType, keys(r) AS properties
RETURN relationshipType, properties;

CALL apoc.meta.nodeTypeProperties()
YIELD nodeType, propertyName, propertyTypes
RETURN nodeType, propertyName, propertyTypes;

CALL apoc.meta.relTypeProperties()
YIELD relType, propertyName, propertyTypes
RETURN relType, propertyName, propertyTypes;
```

## Proximity and Contact Queries

```cypher
MATCH (:Person {healthstatus: 'Sick'})-[:VISITS]->(:Place)<-[:VISITS]-(p:Person)
RETURN DISTINCT p;

MATCH (p:Person {healthstatus: 'sick'})-[:VISITS]->(v:Visit)
WHERE v.starttime > p.confirmedtime
RETURN p;

MATCH (p:Person {healthstatus: 'sick'})-[:VISITS]->(pl:Place)
WITH p, pl, COUNT(*) AS visitCount
WHERE visitCount > 1
RETURN p, pl, visitCount;

MATCH (p:Person)
RETURN p.healthstatus, COUNT(p) AS count;

MATCH (p:Person {healthstatus: 'sick'})-[:VISITS]->(pl:Place)
RETURN pl, COUNT(*) AS visitCount
ORDER BY visitCount DESC;
```

## Relationship Construction – MEETS

```cypher
MATCH (p1:Person)-[v1:VISITS]->(pl:Place)<-[v2:VISITS]-(p2:Person)
WHERE id(p1)<id(p2)
WITH p1, p2,
 apoc.coll.max([v1.starttime.epochMillis, v2.starttime.epochMillis]) AS maxStart,
 apoc.coll.min([v1.endtime.epochMillis, v2.endtime.epochMillis]) AS minEnd
WHERE maxStart <= minEnd
WITH p1, p2, sum(minEnd-maxStart) AS meetTime
CREATE (p1)-[:MEETS {meettime: duration({seconds: meetTime/1000})}]->(p2);
```

## PageRank and Betweenness Centrality

```cypher
CALL gds.pageRank.write({
  nodeProjection: 'Person',
  relationshipProjection: {
    MEETS: {
      type: 'MEETS',
      orientation: 'UNDIRECTED'
    }
  },
  writeProperty: 'pagerank'
})
YIELD nodePropertiesWritten, ranIterations;

MATCH (p:Person)
RETURN p.name AS Person, p.pagerank AS PageRank
ORDER BY p.pagerank DESC
LIMIT 10;

CALL gds.alpha.betweenness.stream({
  nodeProjection: 'Person',
  relationshipProjection: {
    MEETS: {
      type: 'MEETS',
      orientation: 'UNDIRECTED'
    }
  }
})
YIELD nodeId, centrality
WITH gds.util.asNode(nodeId) AS person, centrality
SET person.betweenness = centrality;

MATCH (p:Person)
RETURN p.name AS Person, p.betweenness AS BetweennessCentrality
ORDER BY p.betweenness DESC
LIMIT 10;
```

## Community Detection

```cypher
MATCH p=()-[r:MEETS]->()
SET r.meettimeinseconds = r.meettime.seconds;

CALL gds.louvain.write({
  nodeProjection: 'Person',
  relationshipProjection: {
    MEETS: {
      type: 'MEETS',
      orientation: 'UNDIRECTED',
      properties: 'meettimeinseconds'
    }
  },
  writeProperty: 'community'
})
YIELD communityCount, modularity;

MATCH (p:Person)
RETURN p.community AS Community, COUNT(*) AS NumberOfMembers
ORDER BY NumberOfMembers DESC;

CALL gds.alpha.scc.write({
  nodeProjection: 'Person',
  relationshipProjection: {
    MEETS: {
      type: 'MEETS',
      orientation: 'UNDIRECTED'
    }
  },
  writeProperty: 'scc'
})
YIELD setCount, maxSetSize, minSetSize;
```

## Performance Analysis (Python)

```python
from neo4j import GraphDatabase
import matplotlib.pyplot as plt

uri = "bolt://localhost:7687"
username = "neo4j"
password = "Startup2015*"

driver = GraphDatabase.driver(uri, auth=(username, password))

def execute_query(query):
    with driver.session() as session:
        result = session.run(query)
        return result.consume().result_available_after + result.consume().result_consumed_after

queries = {
    "Louvain Community Detection": """
    CALL gds.louvain.write({
      nodeProjection: 'Person',
      relationshipProjection: {
        MEETS: {
          type: 'MEETS',
          orientation: 'UNDIRECTED',
          properties: 'meettimeinseconds'
        }
      },
      writeProperty: 'community'
    })
    YIELD communityCount, modularity
    """,
    "Identify Communities and Member Count": """
    MATCH (p:Person)
    RETURN p.community AS Community, COUNT(*) AS NumberOfMembers
    ORDER BY NumberOfMembers DESC
    """,
    "Scc algorithm": """
    CALL gds.alpha.scc.write({
      nodeProjection: 'Person',
      relationshipProjection: {
        MEETS: {
          type: 'MEETS',
          orientation: 'UNDIRECTED'
        }
      },
      writeProperty: 'scc'
    })
    YIELD setCount, maxSetSize, minSetSize
    """
}

execution_times = {name: execute_query(query) for name, query in queries.items()}
driver.close()

plt.figure(figsize=(12, 6))
plt.bar(execution_times.keys(), execution_times.values(), color=['blue', 'green', 'red', 'purple'])
plt.xlabel('Query')
plt.ylabel('Execution Time (ms)')
plt.title('Execution Time for Community Detection Queries')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

## Observations

- **PageRank** is efficient and fast, ideal for identifying key individuals.
- **Betweenness Centrality** is slower but highlights critical intermediaries.
- **Louvain** outperforms SCC for community detection in this dataset.
