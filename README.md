# Epidemic Contact Tracing Through Graph Insights

How can we better understand and react to an epidemic using digital tools? When a virus spreads within a community, it's essential to identify who might be infected, which places carry the most risk, and how the contagion evolves through social contacts.

Rather than relying on flat tables and databases, we leverage **graph modeling** to gain a clearer, more dynamic view.

---

## 🔍 Discovering the Network Behind Infections

Imagine each individual as a **node**, and each location visited as another node. When a person visits a location, a **VISITS** connection is created. From these visits, we infer new links — such as **MEETS**, when two people are present at the same place at the same time.

This structure allows us to analyze how the disease spreads across the network.

---

## 🎯 Key Questions Explored

- **Contact Tracing**: Who has potentially been exposed to the virus by visiting the same places as infected individuals?
- **Superspreaders**: Are there people who are central to many transmission paths?
- **Risky Locations**: Which locations have been visited most often by sick individuals?
- **Communities at Risk**: Are there subgroups where the virus circulates intensely?

We address these with network analytics, identifying patterns and high-risk zones.

---

## 📊 Graph-Based Analysis

Using the power of graph theory, we apply:
- **PageRank** to highlight influential individuals in the network.
- **Betweenness Centrality** to find people who connect separate groups.
- **Community Detection (Louvain)** to identify clusters with intense transmission.
- **SCC (Tarjan)** to detect tightly connected components.

Each metric brings a new lens for epidemic monitoring.

---

## 🧪 Performance Insights

We measured the performance of our analyses with Python, comparing how quickly each algorithm runs and what insights it reveals. Visualizations help interpret the results:

- PageRank is fast and informative.
- Betweenness is powerful but slower.
- SCC found isolated nodes — suggesting gaps or limits in network connectivity.

---

## 🧭 What You'll Find in This Project

- `queries.md` – Core graph queries used in the analysis.
- `install.md` – Step-by-step guide to set up and run the project locally.
- Visualizations of graphs and execution time histograms.

---

## 👤 Author

Project developed by **Khayar Anas**  
Passionate about data science, networks, and cybersecurity.
