# Epidemic Contact Tracing Using Graph Analytics (Neo4j)

## 🧩 What problem are we trying to solve?

During a pandemic, health agencies need to answer urgent questions:

- Who was in contact with infected individuals?
- Are there superspreaders in the population?
- Which places are likely contamination hotspots?
- Can we detect communities where the virus circulates rapidly?

Traditional tabular analysis doesn't capture these interactions efficiently.

We explore how **graph databases and algorithms** help answer these questions **in real time**.

---

## 🧠 Why use a graph model?

Each person becomes a **node**. Each place visited becomes another node. The **VISITS** relationships represent interactions. From this, we derive new relationships like **MEETS**, when two people were at the same place at the same time.

This graph structure allows us to ask powerful questions with Cypher.

---

## 🚀 Questions we solve with Cypher & Neo4j

### 1. What is the structure of our epidemic graph?

We start by exploring the graph schema, node types and relationships using:

```cypher
CALL db.schema.visualization();
MATCH (n) RETURN labels(n), keys(n);
```

---

### 2. Who might be infected by contact?

If someone is sick, who has visited the same places?

```cypher
MATCH (:Person {healthstatus: 'Sick'})-[:VISITS]->(:Place)<-[:VISITS]-(p:Person)
RETURN DISTINCT p;
```

We progressively reconstruct potential exposure chains.

---

### 3. Are some individuals key in the epidemic spread?

We compute **PageRank** and **Betweenness Centrality** to find:

- Individuals who act as major spreaders
- People central to many transmission paths

```cypher
CALL gds.pageRank.write({...});
CALL gds.alpha.betweenness.stream({...});
```

---

### 4. Can we identify high-risk communities?

By analyzing interaction durations between people, we apply:

- **Louvain**: to detect transmission communities
- **Tarjan's SCC**: to detect isolated or cyclic zones

This allows us to see **clusters of infection**.

---

### 5. How efficient are these algorithms?

We benchmarked PageRank, Betweenness, Louvain, and SCC using Python + Neo4j driver.  
Results were visualized in histograms (see `/screenshots/`).

Key finding:
- **PageRank is fast and effective**
- **Betweenness is powerful but computationally heavier**
- **SCC failed to find large components**, revealing structural gaps in the graph

---

## 💡 Conclusion

This project shows how **graph thinking** transforms epidemic analysis:
- It enables fast, rich querying
- It helps visualize social structures and risk patterns
- It’s adaptable to real-time data and decision-making

Graph databases like Neo4j are powerful tools in the fight against pandemics.

---

## 📁 Project Contents

- `queries.md` – All Cypher queries with explanations
- `rapport_GEDA.pdf` – Analysis and interpretation
- `projet_KHAYAR_GEDA_2024.txt` – Raw ENSIIE deliverable
- `screenshots/` – Graph and histogram visualizations

---

## 👤 Author

Project developed by **Khayar Anas**  
[LinkedIn](https://www.linkedin.com)  
Passionate about data science, graph analytics, and cybersecurity.
