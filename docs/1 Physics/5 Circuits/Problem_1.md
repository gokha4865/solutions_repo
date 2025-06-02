# Problem 1: Equivalent Resistance Using Graph Theory

## Introduction and Motivation

Calculating equivalent resistance is a fundamental task in circuit analysis. While traditional approaches use series and parallel simplification rules manually, graph theory provides a powerful alternative — transforming circuit analysis into an algorithmic graph simplification process.

Circuits can be represented as **graphs**:
- **Nodes** = junctions
- **Edges** = resistors (with resistance as weight)

By identifying **series** and **parallel** patterns in the graph, we can iteratively reduce it to compute total resistance between two terminals.

This approach is:
- **Scalable** for large networks
- **Automatable** for software
- **Insightful** for understanding electrical connectivity and topology

---

## Learning Goals

After completing this task, you should be able to:

- Represent a resistor network as a weighted graph.
- Apply reduction rules (series and parallel) using graph algorithms.
- Implement an algorithm that simplifies arbitrary configurations.
- Analyze different circuit structures programmatically.

---


### Series Connection

Two resistors `R₁` and `R₂` in series:

$$ R_{eq} = R_1 + R_2 $$

### Parallel Connection

Two resistors `R₁` and `R₂` in parallel:

$$
\frac{1}{R_{eq}} = \frac{1}{R_1} + \frac{1}{R_2}
\quad \Rightarrow \quad
R_{eq} = \left( \frac{1}{R_1} + \frac{1}{R_2} \right)^{-1}
$$

---

## Implementation in Python with `networkx`

Below is a complete Python implementation and visualization of step-by-step circuit simplification using `networkx`.

## code 
```python
import matplotlib.pyplot as plt
import networkx as nx

def draw_circuit(G, pos, title, highlight_paths=None):
    plt.figure(figsize=(10, 6))
    node_colors = ['#4CAF50' if node in ['start', 'end'] else '#2196F3' for node in G.nodes()]
    
    # Draw nodes and edges
    nx.draw_networkx_nodes(G, pos, node_color=node_colors, node_size=1800, edgecolors='black', linewidths=2)
    nx.draw_networkx_labels(G, pos, font_size=14, font_weight='bold')
    
    # Draw all edges first in black (thin)
    nx.draw_networkx_edges(G, pos, width=2, edge_color='black')
    
    # Draw highlighted edges on top (thicker)
    if highlight_paths:
        for i, path in enumerate(highlight_paths):
            color = ['#FF5722', '#3F51B5', '#009688'][i % 3]  # Orange, Blue, Teal
            nx.draw_networkx_edges(G, pos, edgelist=path, edge_color=color, width=6, alpha=0.7)
    
    # Add resistance labels with white background for readability
    edge_labels = {(u, v): f"{d['resistance']}Ω" for u, v, d in G.edges(data=True)}
    nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels, font_size=12, 
                               bbox=dict(facecolor='white', edgecolor='none', alpha=0.8, 
                                         boxstyle='round,pad=0.3'))
    
    plt.title(title, fontsize=16, pad=20)
    plt.axis('off')
    plt.tight_layout()
    plt.show()

# =================================================================
# Circuit 1: Complex Parallel-Series Combination
# =================================================================
G1 = nx.Graph()
G1.add_edge('start', 'A', resistance=4)
G1.add_edge('A', 'C', resistance=2)
G1.add_edge('start', 'B', resistance=8)
G1.add_edge('B', 'C', resistance=4)
G1.add_edge('C', 'end', resistance=6)

pos1 = {
    'start': (0, 0),
    'A': (1, 1),
    'B': (1, -1),
    'C': (2, 0),
    'end': (3, 0)
}

draw_circuit(G1, pos1, "Circuit 1: Parallel Paths (start-A-C and start-B-C)",
            highlight_paths=[
                [('start', 'A'), ('A', 'C')],  # Orange path
                [('start', 'B'), ('B', 'C')]   # Blue path
            ])

# =================================================================
# Circuit 2: Pure Parallel Configuration (Fixed Version)
# =================================================================
plt.figure(figsize=(10, 6))
G2 = nx.MultiGraph()
pos2 = {
    'start': (0, 0),
    'C': (3, 0),
    'end': (6, 0)
}

# Add edges
G2.add_edge('start', 'C', resistance=6)
G2.add_edge('start', 'C', resistance=12)
G2.add_edge('C', 'end', resistance=6)

# Draw nodes
nx.draw_networkx_nodes(G2, pos2, node_size=1800, 
                      node_color=['#4CAF50', '#2196F3', '#4CAF50'])
nx.draw_networkx_labels(G2, pos2, font_size=14, font_weight='bold')

# Draw edges with clear labeling
# First parallel resistor (6Ω)
nx.draw_networkx_edges(G2, pos2, edgelist=[('start', 'C')], 
                      edge_color='#FF5722', width=4,
                      connectionstyle="arc3,rad=0.15")

# Label for first parallel resistor
plt.text(1.5, 0.25, "6Ω", ha='center', va='center', 
        fontsize=12, color='black',
        bbox=dict(facecolor='white', alpha=1, boxstyle='round,pad=0.3'))

# Second parallel resistor (12Ω)
nx.draw_networkx_edges(G2, pos2, edgelist=[('start', 'C')], 
                      edge_color='#3F51B5', width=4,
                      connectionstyle="arc3,rad=-0.15")

# Label for second parallel resistor
plt.text(1.5, -0.25, "12Ω", ha='center', va='center', 
        fontsize=12, color='black',
        bbox=dict(facecolor='white', alpha=1, boxstyle='round,pad=0.3'))

# Series resistor (6Ω)
nx.draw_networkx_edges(G2, pos2, edgelist=[('C', 'end')], width=3)
plt.text(4.5, 0, "6Ω", ha='center', va='center', 
        fontsize=12, color='black',
        bbox=dict(facecolor='white', alpha=1, boxstyle='round,pad=0.3'))

plt.title("Circuit 2: Parallel Configuration (6Ω || 12Ω)", fontsize=16, pad=20)
plt.xlim(-0.5, 6.5)
plt.ylim(-0.5, 0.5)
plt.axis('off')
plt.tight_layout()
plt.show()

# =================================================================
# Circuit 3: Simple Series Circuit
# =================================================================
G3 = nx.Graph()
G3.add_edge('start', 'C', resistance=4)
G3.add_edge('C', 'end', resistance=6)

pos3 = {
    'start': (0, 0),
    'C': (2, 0),
    'end': (4, 0)
}

draw_circuit(G3, pos3, "Circuit 3: Simple Series Circuit (4Ω + 6Ω)")

# =================================================================
# Circuit 4: Single Resistor
# =================================================================
G4 = nx.Graph()
G4.add_edge('start', 'end', resistance=10)

pos4 = {
    'start': (0, 0),
    'end': (2, 0)
}

draw_circuit(G4, pos4, "Circuit 4: Single Resistor (10Ω)")
```

![alt text](Figure_1.png)

![alt text](Figure_2.png)

![alt text](Figure_3.png)

![alt text](Figure_4.png)

---

## Summary

Graph theory allows us to **automatically simplify resistor networks**, handle **complex configurations**, and is highly suited for **software tools** and **educational simulations**.

---

## Resources

- [`networkx` documentation](https://networkx.org/)
- Circuit visualization tools
- Graph reduction algorithms
