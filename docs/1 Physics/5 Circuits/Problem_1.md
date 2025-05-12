# Problem 1
## Equivalent Resistance Calculation Using Graph Theory

This document presents an algorithm and its Python implementation for calculating the equivalent resistance of an electrical circuit represented as a graph. The method iteratively simplifies the circuit by identifying and reducing series and parallel combinations of resistors.

### Motivation

Traditional methods for calculating equivalent resistance by manually applying series and parallel rules can be tedious and error-prone for complex circuits. Graph theory provides a systematic and algorithmically programmable approach. By modeling junctions as nodes and resistors as weighted edges, even intricate networks can be analyzed. This approach is foundational for circuit simulation software and automated electronic design tools.

---

### 1. Algorithm Description

The circuit is modeled as a multigraph $G=(V, E)$, where $V$ is the set of junctions (nodes) and $E$ is the set of resistors (edges). Each edge $e \in E$ has a weight $r_e$ representing its resistance. Two distinct nodes, `start_node` and `end_node`, are specified as the terminals across which the equivalent resistance is sought.

The algorithm proceeds as follows:

1.  **Initialization**:
    * Construct the graph from the given circuit components.
    * Validate that `start_node` and `end_node` exist in the graph. If `start_node` == `end_node`, the resistance is $0 \Omega$.

2.  **Iterative Reduction**: Repeatedly apply the following reduction rules until no more simple series or parallel reductions are possible, or the graph is reduced to a single edge between `start_node` and `end_node`.

    * **Parallel Reduction**:
        * **Identification**: Find any pair of nodes $(u, v)$ connected by two or more edges (resistors $R_1, R_2, \dots, R_k$).
        * **Reduction**: Replace these $k$ parallel resistors with a single equivalent resistor $R_p$ between $u$ and $v$. The equivalent resistance is calculated as:
            $$R_p = \left( \sum_{i=1}^{k} \frac{1}{R_i} \right)^{-1}$$
            If any $R_i = 0$, $R_p = 0$. If all $R_i = \infty$, $R_p = \infty$.
        * **Action**: Remove the original $k$ edges and add a new edge $(u,v)$ with resistance $R_p$. If $R_p = \infty$, effectively remove the connection unless it's the only path.
        * After a reduction, restart the scan for further simplifications as the graph structure has changed.

    * **Series Reduction**:
        * **Identification**: Find any node $w$ such that:
            1.  $w$ is not the `start_node` or `end_node`.
            2.  The degree of $w$, $\text{deg}(w)$, is exactly 2. Let the two edges incident to $w$ be $(u,w)$ with resistance $R_1$ and $(w,v)$ with resistance $R_2$.
            3.  Ensure $u \neq v$. (If $u=v$, it's a loop, not a simple series element in a path from `start_node` to `end_node`).
        * **Reduction**: Replace the two series resistors $R_1$ and $R_2$ and the intermediate node $w$ with a single equivalent resistor $R_s$ between $u$ and $v$. The equivalent resistance is:
            $$R_s = R_1 + R_2$$
            If $R_1 = \infty$ or $R_2 = \infty$, then $R_s = \infty$.
        * **Action**: Remove node $w$ (which also removes edges $(u,w)$ and $(w,v)$). Add a new edge $(u,v)$ with resistance $R_s$. If $R_s = \infty$, effectively this path segment becomes an open circuit.
        * After a reduction, restart the scan.

3.  **Termination and Result**:
    * The loop terminates if no series or parallel reductions were made in a full pass.
    * **Ideal Case**: If the graph is reduced to a single edge between `start_node` and `end_node`, its resistance is the equivalent resistance.
    * **Final Parallel Case**: If the graph consists of only `start_node` and `end_node` and multiple edges directly connecting them, these are in parallel. Calculate their combined parallel resistance.
    * **Non-Reducible Case**: If the graph cannot be simplified further by these rules to one of the above states, the circuit may contain non-series-parallel configurations (e.g., a Wheatstone bridge not in balance). The algorithm, in this form, will report that it cannot fully reduce the circuit.
    * **Open Circuit**: If, at any point or at the end, there is no path between `start_node` and `end_node`, the equivalent resistance is $\infty$.
    * **Short Circuit**: If an equivalent resistance of $0 \Omega$ is found.

#### Pseudocode

```plaintext
FUNCTION CalculateEquivalentResistance(edge_list, start_node, end_node):
  IF start_node == end_node:
    RETURN 0.0

  graph = CreateGraph(edge_list) // Use a MultiGraph

  IF NOT graph.has_node(start_node) OR NOT graph.has_node(end_node):
    RETURN "Invalid terminals"

  max_iterations = CalculateMaxIterations(graph) // e.g., based on N_nodes + N_edges
  iterations = 0

  WHILE iterations < max_iterations:
    iterations++
    made_reduction_in_pass = FALSE

    // --- 1. Attempt Parallel Reduction ---
    FOR EACH pair of distinct nodes (u, v) in graph:
      parallel_edges = get_edges_between(graph, u, v)
      IF count(parallel_edges) > 1:
        resistances = [edge.resistance FOR edge IN parallel_edges]
        equivalent_R_parallel = CalculateParallelResistance(resistances)
        
        remove_edges(graph, parallel_edges)
        IF equivalent_R_parallel IS NOT INFINITY:
            add_edge(graph, u, v, equivalent_R_parallel)
        
        made_reduction_in_pass = TRUE
        BREAK // Restart scans from the beginning

    IF made_reduction_in_pass:
      CONTINUE // Restart WHILE loop

    // --- 2. Attempt Series Reduction ---
    FOR EACH node w in graph.nodes (copy of list):
      IF graph.has_node(w) AND w IS NOT start_node AND w IS NOT end_node AND graph.degree(w) == 2:
        incident_edges = get_incident_edges(graph, w) // Should be two edges, say e1 and e2
        node_u = get_other_node(e1, w)
        node_v = get_other_node(e2, w)

        IF node_u IS NOT node_v: // Ensure it's not a self-loop structure simplified this way
          R1 = e1.resistance
          R2 = e2.resistance
          equivalent_R_series = CalculateSeriesResistance(R1, R2)
          
          graph.remove_node(w) // Removes w and incident edges e1, e2
          IF equivalent_R_series IS NOT INFINITY:
            add_edge(graph, node_u, node_v, equivalent_R_series)
          
          made_reduction_in_pass = TRUE
          BREAK // Restart scans

    IF NOT made_reduction_in_pass:
      BREAK // No more simple reductions possible, exit WHILE

  // --- 3. Final Evaluation ---
  IF graph has exactly one edge BETWEEN start_node and end_node AND graph.nodes == {start_node, end_node}:
    RETURN resistance_of_edge(graph, start_node, end_node)
  ELSE IF graph has only nodes start_node, end_node AND all edges are between them:
    terminal_edges = get_edges_between(graph, start_node, end_node)
    IF count(terminal_edges) > 0:
      RETURN CalculateParallelResistance([edge.resistance FOR edge IN terminal_edges])
    ELSE: // No edges left between terminals
      RETURN INFINITY
  ELSE IF graph.has_node(start_node) AND graph.has_node(end_node) AND NOT nx.has_path(graph, start_node, end_node):
      RETURN INFINITY
  ELSE:
    RETURN "Circuit not reducible by simple series/parallel methods to a single value."

FUNCTION CalculateParallelResistance(resistances_list):
  sum_conductance = 0
  has_zero_resistor = FALSE
  num_infinite_resistors = 0
  FOR R IN resistances_list:
    IF R == 0: has_zero_resistor = TRUE; BREAK
    IF R == INFINITY: num_infinite_resistors++; CONTINUE
    sum_conductance += 1.0 / R
  IF has_zero_resistor: RETURN 0.0
  IF num_infinite_resistors == length(resistances_list): RETURN INFINITY
  IF sum_conductance == 0: RETURN INFINITY // All remaining paths are open
  RETURN 1.0 / sum_conductance

FUNCTION CalculateSeriesResistance(R1, R2):
  IF R1 == INFINITY OR R2 == INFINITY:
    RETURN INFINITY
  RETURN R1 + R2
```

#### Handling Nested Combinations

The iterative nature of the algorithm, particularly the restart after each successful reduction, ensures that nested combinations are handled correctly. An inner combination (e.g., two resistors in parallel) is reduced first. This simplification changes the graph structure, potentially revealing that the newly formed equivalent resistor is now in series with another. The subsequent pass of the algorithm will then identify and reduce this new series combination. This process continues, simplifying the circuit from the innermost combinations outwards.

---

### 2. Full Python Implementation

The following Python code uses the `networkx` library to manage the graph structure.

```python
import networkx as nx
import itertools
import math

def calculate_equivalent_resistance(edges_with_resistances, start_node, end_node):
    """
    Calculates the equivalent resistance of a circuit graph using iterative
    series and parallel reductions.

    Args:
        edges_with_resistances (list of tuples): A list where each tuple
            represents a resistor and is of the form (node1, node2, resistance_value).
            Resistance can be float('inf') for an open path.
            Example: [('A', 'B', 10), ('B', 'C', 20)]
        start_node (any hashable type): The starting terminal node.
        end_node (any hashable type): The ending terminal node.

    Returns:
        float: The equivalent resistance (can be float('inf')).
        str: An error message if the circuit is not reducible by this method
             or if the terminals are invalid.
    """
    if start_node == end_node:
        return 0.0

    graph = nx.MultiGraph() # Allows multiple edges between two nodes (parallel resistors)
    for u, v, r in edges_with_resistances:
        if not isinstance(r, (int, float)):
            raise ValueError(f"Resistance value must be a number. Found: {r} for edge ({u}-{v})")
        if r < 0:
            raise ValueError(f"Resistance values must be non-negative. Found: {r} for edge ({u}-{v})")
        graph.add_edge(u, v, resistance=float(r))

    if not graph.has_node(start_node) or not graph.has_node(end_node):
        # This check is for nodes defined in edges. If terminals aren't in edges, they won't be in graph.
        # If edges_with_resistances is empty, graph is empty.
        if not edges_with_resistances: # No edges, so open circuit if start != end
             return float('inf') if start_node != end_node else 0.0
        return "Start or end node not in the provided edges."
    
    # Initial check for path existence only if graph has edges.
    # If no edges, but different start/end nodes, it's an open circuit.
    if graph.number_of_edges() == 0 and start_node != end_node: # Already handled if edges_with_resistances is empty
        return float('inf')
    # Check path only if both nodes exist and there are edges.
    if graph.number_of_edges() > 0 and (not nx.has_path(graph, start_node, end_node)):
         return float('inf')


    iteration_count = 0
    # Heuristic limit for iterations to prevent infinite loops in unexpected graph states
    max_iterations = (graph.number_of_nodes() + graph.number_of_edges()) * 2 + 10 

    while iteration_count < max_iterations:
        iteration_count += 1
        reduced_in_iteration = False

        # --- 1. Attempt to reduce parallel resistors ---
        # Iterate over unique pairs of nodes that have edges between them
        potential_parallel_pairs = list(set(tuple(sorted(edge_nodes)) for edge_nodes in graph.edges()))
        
        for u, v in potential_parallel_pairs:
            if not graph.has_node(u) or not graph.has_node(v): continue # Nodes might have been removed

            num_edges_uv = graph.number_of_edges(u, v)
            if num_edges_uv > 1:
                edge_data_list = graph.get_edge_data(u, v) # Dict: {key: {'resistance': val}}
                resistances = [attrs['resistance'] for attrs in edge_data_list.values()]

                # Calculate equivalent parallel resistance
                conductance_sum = 0.0
                has_zero_resistor = any(r == 0.0 for r in resistances)
                num_infinite_resistors = sum(1 for r in resistances if r == float('inf'))

                if has_zero_resistor:
                    equivalent_r_parallel = 0.0
                elif num_infinite_resistors == len(resistances): # All are open circuits
                    equivalent_r_parallel = float('inf')
                else:
                    for r_val in resistances:
                        if r_val > 0 and r_val != float('inf'): # Positive, finite resistance
                            conductance_sum += 1.0 / r_val
                    if conductance_sum == 0: # Only infinite resistors remained among non-zeros, or an issue.
                        equivalent_r_parallel = float('inf')
                    else:
                        equivalent_r_parallel = 1.0 / conductance_sum
                
                # Remove all existing edges between u and v by their keys
                keys_to_remove = list(edge_data_list.keys()) # Make a copy of keys
                for key_val in keys_to_remove:
                    if graph.has_edge(u, v, key=key_val): # Check if edge still exists
                        graph.remove_edge(u, v, key=key_val)
                
                # Add the new equivalent edge if not an open circuit
                if equivalent_r_parallel != float('inf'):
                    graph.add_edge(u, v, resistance=equivalent_r_parallel)
                
                reduced_in_iteration = True
                break # Restart scans as graph structure changed
        
        if reduced_in_iteration:
            continue # Go to next iteration of the WHILE loop

        # --- 2. Attempt to reduce series resistors ---
        # Iterate over a copy of nodes list as graph.nodes() can change
        nodes_to_check = list(graph.nodes())
        for w in nodes_to_check:
            if not graph.has_node(w): continue # Node might have been removed

            # A middle node in a series connection (not a terminal) must have degree 2
            if w != start_node and w != end_node and graph.degree(w) == 2:
                incident_edges_to_w = list(graph.edges(w, data=True, keys=False)) # data=True gives (u,v,dict)
                # We expect exactly two edges due to degree(w) == 2 check.
                
                # Determine the other nodes (u and v) connected through w
                # e.g. edge1 = (w, u_node, data1), edge2 = (w, v_node, data2)
                # Need to get the nodes that are NOT w
                
                neighbor1_data = incident_edges_to_w[0] # (w, neighbor1, attr_dict1) or (neighbor1, w, attr_dict1)
                neighbor2_data = incident_edges_to_w[1] # (w, neighbor2, attr_dict2) or (neighbor2, w, attr_dict2)

                u_node = neighbor1_data[1] if neighbor1_data[0] == w else neighbor1_data[0]
                v_node = neighbor2_data[1] if neighbor2_data[0] == w else neighbor2_data[0]
                
                r1 = neighbor1_data[2]['resistance']
                r2 = neighbor2_data[2]['resistance']

                if u_node == v_node: 
                    # This case (e.g. u --R1-- w --R2-- u) means R1 and R2 form a loop on node u.
                    # The equivalent of this loop is R1+R2. This R1+R2 is effectively in parallel
                    # with any other paths connected to node u.
                    # For our series reduction pass-through logic, we primarily look for u != v.
                    # If we were to reduce this, it would become a self-loop edge on u_node.
                    # Handling self-loops adds complexity (e.g. a self-loop on a terminal node is parallel to the main path).
                    # For now, we skip series reduction if u_node == v_node to keep focus on path simplification.
                    continue 

                # Standard series calculation: R_s = R1 + R2
                # Handle float('inf') correctly: inf + x = inf; x + inf = inf
                if r1 == float('inf') or r2 == float('inf'):
                    equivalent_r_series = float('inf')
                else:
                    equivalent_r_series = r1 + r2

                # Remove node w (and its incident edges)
                graph.remove_node(w)
                
                # Add the new equivalent series edge between u_node and v_node
                # Only add if not an open circuit path
                if equivalent_r_series != float('inf'):
                    graph.add_edge(u_node, v_node, resistance=equivalent_r_series)
                
                reduced_in_iteration = True
                break # Restart scans
        
        if not reduced_in_iteration:
            break # Exit WHILE loop if no reduction was made in this full pass

    # --- 3. Final check and result ---
    if not graph.has_node(start_node) or not graph.has_node(end_node):
        # This means one or both terminals were removed, likely part of a series reduction
        # that led to a disconnected structure or an ill-posed problem.
        # If the overall circuit became disconnected from the terminals, it's an open circuit.
        return float('inf') 

    # Case 1: Graph is reduced to exactly start_node and end_node, with one or more edges between them
    # This condition also means no other nodes exist in the graph.
    if set(graph.nodes()) == {start_node, end_node}:
        if graph.number_of_edges(start_node, end_node) > 0:
            # All remaining edges are between start_node and end_node. Treat as parallel.
            terminal_resistances = [data['resistance'] for data in graph.get_edge_data(start_node, end_node).values()]
            
            conductance_sum_final = 0.0
            has_zero_resistor_final = any(r == 0.0 for r in terminal_resistances)
            num_infinite_final = sum(1 for r in terminal_resistances if r == float('inf'))

            if has_zero_resistor_final:
                return 0.0
            if num_infinite_final == len(terminal_resistances): # All paths between terminals are open
                return float('inf')
            
            for r_val in terminal_resistances:
                if r_val > 0 and r_val != float('inf'): # Positive, finite resistance
                    conductance_sum_final += 1.0 / r_val
            
            if conductance_sum_final == 0: # Should be covered if all were infinite, or implies error
                return float('inf')
            return 1.0 / conductance_sum_final
        else: # No edges left between start and end, though they are the only nodes
            return float('inf') # Open circuit

    # Case 2: Graph is more complex / not reducible, or terminals are disconnected in the final graph
    # Check if there's still a path between start and end node in the potentially complex remaining graph
    if nx.has_path(graph, start_node, end_node):
        # If a path exists, but it's not simplified to the above terminal-only case,
        # it implies the circuit structure is non-series-parallel between these terminals.
        return (f"Circuit not fully reducible by simple series/parallel between {start_node} and {end_node}. "
                f"Remaining nodes: {list(graph.nodes())}, Remaining edges: {list(graph.edges(data=True))}")
    else: # No path between terminals in the final graph structure
        return float('inf') # Open circuit

    # Fallback for max_iterations reached without clean resolution (should ideally be caught by above logic)
    if iteration_count >= max_iterations: # Should be " >= "
         # Try one last check if it coincidentally resolved on the last iteration
        if set(graph.nodes()) == {start_node, end_node} and graph.number_of_edges(start_node, end_node) > 0:
             # (Simplified re-check for brevity, actual calculation is above)
             return calculate_equivalent_resistance( # Recursive call on potentially simplified graph
                [(u,v,d['resistance']) for u,v,d in graph.edges(data=True)], start_node, end_node
             )


         return (f"Maximum iterations ({max_iterations}) reached. Circuit might be too complex or in an unhandled configuration. "
                f"Final check: Resistance is likely {float('inf')} or irreducible. "
                f"Remaining graph: Nodes: {list(graph.nodes())}, Edges: {list(graph.edges(data=True))}")

    # This part should ideally not be reached if all cases are covered.
    return "Could not determine equivalent resistance (unknown state)."
```

---

### 3. Handling of Complex Circuit Configurations: Examples

#### Example 1: Simple Series and Parallel Combination

* Circuit: R1 (10Ω) A-B, R2 (20Ω) B-C, R3 (30Ω) B-C. Terminals A-C.
* Expected: $R_{BC,p} = (1/20 + 1/30)^{-1} = 12\Omega$. $R_{AC,total} = R_{AB} + R_{BC,p} = 10 + 12 = 22\Omega$.

```python
edges1 = [('A', 'B', 10), ('B', 'C', 20), ('B', 'C', 30)]
start1, end1 = 'A', 'C'
R_eq1 = calculate_equivalent_resistance(edges1, start1, end1)
print(f"Example 1 ({start1}-{end1}): {R_eq1} Ohms")
# Output: Example 1 (A-C): 22.0 Ohms
```


**Handling**:
1.  The algorithm first identifies R2 (20Ω) and R3 (30Ω) as parallel between B and C. They are replaced by a single $12\Omega$ resistor (B-C).
2.  In the next pass, R1 (10Ω A-B) and the new $12\Omega$ (B-C) resistor are identified as series because node B has degree 2 and is not a terminal. They are replaced by $10+12 = 22\Omega$ resistor (A-C).
3.  The graph is now a single $22\Omega$ resistor between terminals A and C. The algorithm terminates, returning 22.0.

#### Example 2: Nested Configuration

* Circuit: R1(5Ω) A-B. (R2(10Ω) B-C || R3(10Ω) B-C). R4(20Ω) C-D. This entire branch A-D is in parallel with R5(30Ω) A-D. Terminals A-D.
* Expected: $R_{BC,p} = (1/10 + 1/10)^{-1} = 5\Omega$. Upper branch $R_{upper} = 5 + 5 + 20 = 30\Omega$. Total $R_{AD} = (R_{upper} || R5) = (1/30 + 1/30)^{-1} = 15\Omega$.

```python
edges2 = [('A', 'B', 5), ('B', 'C', 10), ('B', 'C', 10), ('C', 'D', 20), ('A', 'D', 30)]
start2, end2 = 'A', 'D'
R_eq2 = calculate_equivalent_resistance(edges2, start2, end2)
print(f"Example 2 ({start2}-{end2}): {R_eq2} Ohms")
# Output: Example 2 (A-D): 15.0 Ohms
```


**Handling**:
1.  Parallel: R2 and R3 (B-C, both 10Ω) are reduced to a single $5\Omega$ resistor between B and C.
2.  Series: The $5\Omega$ (A-B) and the new $5\Omega$ (B-C) are in series via node B. They are reduced to a $10\Omega$ resistor (A-C).
3.  Series: This $10\Omega$ (A-C) and the $20\Omega$ (C-D) are in series via node C. They are reduced to a $30\Omega$ resistor (A-D).
4.  Parallel: The graph now has two $30\Omega$ resistors between A and D (the one just formed and the original R5). These are reduced to a single $15\Omega$ resistor (A-D).
5.  The graph is a single $15\Omega$ resistor between terminals A and D. Result is 15.0.

#### Example 3: Complex Graph (Non-Series-Parallel - Cube)

* Circuit: A cube where each edge is a $1\Omega$ resistor. Terminals are diagonally opposite corners (e.g., 0 to 7).
* Expected: This circuit is not reducible by simple series-parallel operations alone. The known result for a unit resistor cube body diagonal is $5/6 \Omega \approx 0.8333\Omega$. The algorithm should report its inability to reduce it to a single value using only series-parallel steps.

```python
edges3 = [
    (0,1,1), (0,2,1), (0,4,1), (1,3,1), (1,5,1), (2,3,1), 
    (2,6,1), (3,7,1), (4,5,1), (4,6,1), (5,7,1), (6,7,1)
]
start3, end3 = 0, 7
R_eq3 = calculate_equivalent_resistance(edges3, start3, end3)
print(f"Example 3 (Cube {start3}-{end3}): {R_eq3}")
# Expected Output (actual might vary slightly in node/edge order in message):
# Example 3 (Cube 0-7): Circuit not fully reducible by simple series/parallel between 0 and 7. Remaining nodes: [0, 1, 2, 4, 3, 5, 6, 7], Remaining edges: [(0, 1, {'resistance': 1.0}), (0, 2, {'resistance': 1.0}), (0, 4, {'resistance': 1.0}), (1, 3, {'resistance': 1.0}), (1, 5, {'resistance': 1.0}), (2, 3, {'resistance': 1.0}), (2, 6, {'resistance': 1.0}), (3, 7, {'resistance': 1.0}), (4, 5, {'resistance': 1.0}), (4, 6, {'resistance': 1.0}), (5, 7, {'resistance': 1.0}), (6, 7, {'resistance': 1.0})]
```


**Handling**:
The algorithm initializes the graph with the 12 resistors of the cube.
1.  Parallel Scan: No two nodes are connected by more than one resistor initially. No parallel reduction.
2.  Series Scan:
    * Node 0 (start) and 7 (end) are terminals, not considered for series middle-points.
    * All other nodes (1, 2, 3, 4, 5, 6) each have a degree of 3 (each connects to three other nodes).
    * Since no non-terminal node has a degree of 2, no series reduction can be performed.
With no possible series or parallel reductions in the first full pass, the `reduced_in_iteration` flag remains `False`. The `WHILE` loop terminates. The final check determines that the graph is not a single edge between terminals, nor is it a set of parallel edges only between terminals. It then reports that the circuit is not fully reducible by these methods, listing the remaining graph structure.

---

### 4. Analysis of Algorithm's Efficiency and Potential Improvements

#### Efficiency

* **Time Complexity**:
    The `while` loop continues as long as reductions are possible. The maximum number of reductions is $O(N+M)$, where $N$ is the number of nodes and $M$ is the number of edges.
    Inside the loop:
    * Parallel reduction: Iterating `potential_parallel_pairs` (derived from $O(M)$ edges) and then calling `graph.number_of_edges(u,v)` (efficient in `networkx`) and `graph.get_edge_data(u,v)`. Removing and adding edges takes time proportional to degree/local changes.
    * Series reduction: Iterating `nodes_to_check` ($O(N)$), checking degree (efficient), and then graph modifications.
    Each reduction step simplifies the graph. The `break` and `continue` statements restart the scan. In worst-case scenarios for dense graphs or specific structures, finding a reduction could take up to $O(N^2)$ or $O(M \cdot \text{avg_degree})$. If $k$ reductions occur, the total could be roughly $k \times (\text{cost_of_scan_and_reduction})$. A loose upper bound might appear high, e.g., $O((N+M)(N^2+M))$. However, for sparse graphs typical in circuits ($M \approx O(N)$), and with efficient `networkx` operations, practical performance is often better for reducible circuits. The key factor is the number of full scan restarts.

* **Space Complexity**: $O(N+M)$ for storing the graph data (nodes, edges, attributes) using `networkx`.

#### Potential Improvements

1.  **Optimized Scanning Strategy**:
    * Instead of a full restart after every single reduction, perform all possible non-overlapping reductions of one type (e.g., all identifiable parallel pairs) in one go, then all non-overlapping series, then repeat. This reduces the number of full loop iterations.
    * Maintain a queue or list of nodes/edges that become candidates for reduction after a modification, rather than re-scanning everything.

2.  **Handling of Non-Series-Parallel Circuits (Y-Δ / Δ-Y Transformations)**:
    * The current algorithm is limited to series-parallel reducible circuits. To handle more general circuits (like the cube example or unbalanced Wheatstone bridges), Y-Δ (Star-Mesh) and Δ-Y (Mesh-Star) transformations are necessary.
    * This involves:
        * Identifying 'Y' (3 edges meeting at a central node, not a terminal) or 'Δ' (3 edges forming a triangle between 3 nodes) patterns.
        * Applying the appropriate resistance transformation formulas:
            * Y ($R_a, R_b, R_c$ from center to A, B, C) to Δ ($R_{AB}, R_{BC}, R_{CA}$):
                $R_{AB} = R_a + R_b + \frac{R_a R_b}{R_c}$ (and similarly for $R_{BC}, R_{CA}$)
            * Δ ($R_{AB}, R_{BC}, R_{CA}$) to Y ($R_a, R_b, R_c$ to new center):
                $R_a = \frac{R_{AB} R_{CA}}{R_{AB} + R_{BC} + R_{CA}}$ (and similarly for $R_b, R_c$)
        * These transformations add significant complexity, including heuristic choices about when and where to apply them, as they can sometimes temporarily increase graph density.

3.  **Numerical Stability**:
    * For circuits with extreme ranges of resistance values (very small or very large), standard floating-point precision (`float`) might lead to inaccuracies, especially in the parallel resistance formula (sum of conductances). Using Python's `Decimal` type could offer higher precision at the cost of performance.

4.  **General Circuit Solvers**:
    * For arbitrary resistive networks not solvable by S-P or Y-Δ methods, more general techniques like Modified Nodal Analysis (MNA) or mesh/loop current analysis are used. These involve setting up and solving systems of linear equations based on Kirchhoff's laws.

5.  **Edge Case Refinements**:
    * The handling of $0 \Omega$ resistors (direct shorts) and $\infty \Omega$ resistors (open circuits) is incorporated but always requires careful validation across all reduction steps.
    * The series reduction rule currently skips `u_node == v_node` cases (a loop created by two series resistors). A more advanced version might explicitly reduce this to a self-loop on `u_node` and then handle self-loops (e.g., a self-loop on a terminal node is effectively in parallel with the rest of the circuit connected to that terminal).

This implementation provides a robust method for series-parallel reducible circuits and correctly identifies its limitations for more complex networks.
