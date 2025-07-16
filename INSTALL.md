
# 🧪 Installation Guide – Epidemiological Surveillance in Neo4j

## Step 1 – Install Neo4j Desktop and required plugins

1. Go to [neo4j.com/download-neo4j-now](https://neo4j.com/download-neo4j-now)
2. Download **Neo4j Desktop** (.AppImage for Linux)
3. Right-click the downloaded file and:
   - Enable “Allow executing file as program”
4. Double-click to launch Neo4j Desktop

---

## Step 2 – Create a local database

1. Click `Add` → `Local DBMS`
2. Provide:
   - **Name**: any name
   - **Password**: remember it
   - **Version**: `3.5.26` (important)
3. Click `Start` to launch the database

---

## Step 3 – Install Plugins

Before opening the DB, go to the `Plugins` panel on the right:
- Install **APOC**
- Install **Graph Data Science Library**

---

## Helpful Resources

- [APOC Documentation](https://neo4j.com/labs/apoc/4.1/)
- [Graph Data Science Documentation](https://neo4j.com/docs/graph-data-science/current/)

---

## Step 4 – Import Data

Copy and paste the following Cypher queries into Neo4j:

```cypher
// Import persons
LOAD CSV WITH HEADERS FROM
"https://docs.google.com/spreadsheets/u/0/d/1R-XVuynPsOWcXSderLpq3DacZdk10PZ8v6FiYGTncIE/export?format=csv&id=1R-XVuynPsOWcXSderLpq3DacZdk10PZ8v6FiYGTncIE&gid=0" AS csv
CREATE (:Person {
  id: csv.PersonId,
  name: csv.PersonName,
  healthstatus: csv.Healthstatus,
  confirmedtime: datetime(csv.ConfirmedTime),
  addresslocation: point({x: toFloat(csv.AddressLat), y: toFloat(csv.AddressLong)})
});

// Import places
LOAD CSV WITH HEADERS FROM
"https://docs.google.com/spreadsheets/u/0/d/1R-XVuynPsOWcXSderLpq3DacZdk10PZ8v6FiYGTncIE/export?format=csv&id=1R-XVuynPsOWcXSderLpq3DacZdk10PZ8v6FiYGTncIE&gid=205425553" AS csv
CREATE (:Place {
  id: csv.PlaceId,
  name: csv.PlaceName,
  type: csv.PlaceType,
  homelocation: point({x: toFloat(csv.Lat), y: toFloat(csv.Long)})
});

CREATE INDEX ON :Place(id);
CREATE INDEX ON :Place(name);
CREATE INDEX ON :Person(id);
CREATE INDEX ON :Person(name);
CREATE INDEX ON :Person(healthstatus);
CREATE INDEX ON :Person(confirmedtime);

// Import visits
LOAD CSV WITH HEADERS FROM
"https://docs.google.com/spreadsheets/d/1R-XVuynPsOWcXSderLpq3DacZdk10PZ8v6FiYGTncIE/export?format=csv&id=1R-XVuynPsOWcXSderLpq3DacZdk10PZ8v6FiYGTncIE&gid=1261126668" AS csv
MATCH (p:Person {id: csv.PersonId}), (pl:Place {id: csv.PlaceId})
CREATE (p)-[:PERFORMS_VISIT]->(v:Visit {
  id: csv.VisitId,
  starttime: datetime(csv.StartTime),
  endtime: datetime(csv.EndTime)
})-[:LOCATED_AT]->(pl),
(p)-[vi:VISITS {
  id: csv.VisitId,
  starttime: datetime(csv.StartTime),
  endtime: datetime(csv.EndTime)
}]->(pl)
SET v.duration = duration.inSeconds(v.starttime, v.endtime),
vi.duration = duration.inSeconds(vi.starttime, vi.endtime);

// Link places to their region
CREATE (r:Region {name: "Antwerp"})-[:PART_OF]->(c:Country {name: "Belgium"})-[:PART_OF]->(co:Continent {name: "Europe"});
MATCH (r:Region {name: "Antwerp"}), (pl:Place)
CREATE (pl)-[:PART_OF]->(r);
```

---

### ⚠️ Troubleshooting URL import errors

> _Neo.ClientError.Statement.ExternalResourceFailed: Couldn't load the external resource_

1. Open your `neo4j.conf` file
2. Comment out the line:

```conf
#dbms.directories.import=import
```

3. Uncomment and modify the following line:

```conf
dbms.security.allow_csv_import_from_file_urls=true
```

4. Restart Neo4j

---

✅ You are now ready to work on the **GEDA – Epidemiological Surveillance in Neo4j** project!
