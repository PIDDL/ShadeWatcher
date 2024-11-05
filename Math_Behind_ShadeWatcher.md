# Math behind ShadeWatcher

## 1. First-order Information (KG Embeddings using TransR)

### Key Idea:
The goal here is to represent system entities (such as network sockets or processes) as vectors using knowledge graph (KG) embeddings. These embeddings capture the behavioral and semantic similarities between entities based on their interactions and relations in the system.

### TransE Embedding:
- **TransE** is a method where if a triple (h, r, t) holds true in a KG (i.e., entity `h` is connected to entity `t` via relation `r`), then the embedding vectors of `h`, `r`, and `t` should satisfy:
  \[
  e_h + e_r \approx e_t
  \]
  This means that the sum of the embedding of `h` and the embedding of the relation `r` should be close to the embedding of `t`. This makes the embeddings of entities with similar behaviors or properties (like network sockets performing similar actions) close in the vector space.

### TransR Embedding:
- The issue with **TransE** is that a single relation type can correspond to multiple entities, causing problems in handling 1-to-N, N-to-1, or N-to-N relationships. For instance, a single IP address can be related to multiple ports (1-to-N).
- **TransR** solves this by embedding entities and relations into different spaces:
  \[
  e^r_h = e_h W_r, \quad e^r_t = e_t W_r
  \]
  - Here, `W_r` is a relation-specific projection matrix that transforms the entity embedding `e_h` from its entity space (d-dimensional) to a relation-specific space (k-dimensional).
  - This allows the same entity (like a process or IP address) to have different embeddings depending on the relation it's involved in.

### Plausibility Score:
- To measure how likely a triple (h, r, t) is to hold in the KG, TransR uses the L1-norm distance between the transformed embeddings:
  \[
  f(h, r, t) = \|e^r_h + e_r - e^r_t\|
  \]
  - A lower score means the triple is more likely to exist in the KG, i.e., it captures a valid connection or relationship between entities in the system.

### Loss Function:
- The model is trained using a **margin-based pairwise ranking loss**:
  \[
  L_{\text{first}} = \sum_{\text{valid (h, r, t)}} \max(0, f(h, r, t) - f(\text{corrupt (h', r', t')}) + \gamma)
  \]
  - The loss compares the score of valid triples to corrupted triples, where one entity is randomly replaced. The goal is to minimize the score for valid triples and maximize the score for corrupted ones, ensuring that valid relationships are learned correctly.
  - **γ** (gamma) is the margin hyper-parameter that controls the gap between the valid and corrupted triples.

### Why This Step is Important:
This step models **direct relationships** (first-order connections) between system entities, capturing similarities and interactions. For example, two network sockets used by the same process are embedded close to each other, which mirrors their semantic and behavioral similarity.

## 2. Higher-order Information (Using GNN)

### Key Idea:
First-order embeddings (like TransR) only capture direct connections between entities (one-hop neighbors). However, in cybersecurity, entities can influence each other across multi-hop connections. For example, a two-hop path might indicate how sensitive data flows through multiple nodes.

### GNN Propagation and Aggregation:
- To capture multi-hop relationships, a **Graph Neural Network (GNN)** is applied. A GNN recursively updates the representation of a system entity `h` by aggregating information from its neighbors.
  \[
  z^{(l)}_h = g(z^{(l-1)}_h, z^{(l-1)}_{N_h})
  \]
  - `z^{(l)}_h` is the updated representation of entity `h` at layer `l`, where `N_h` refers to the neighbors of `h`. The initial embedding `z^{(0)}_h` comes from TransR embeddings.
  - The function `g()` is an **aggregation function** that combines the entity’s current representation with the information from its neighbors. This allows the model to integrate information from higher-order neighbors (multi-hop).

### Attention Mechanism:
- Since not all neighbors contribute equally to an entity's representation, an **attention mechanism** is introduced to weigh the importance of neighboring entities:
  \[
  z^{(l-1)}_{N_h} = \sum_{(h, r, t) \in N_h} \alpha_{hr} z^{(l-1)}_t
  \]
  - Here, `α_{hr}` is the attention score that determines how much information is propagated from neighbor `t` to entity `h`. The attention score is computed as:
    \[
    \alpha_{hr} = \text{softmax}(\text{tanh}(e^r_t + e_r_h + e_r))
    \]
  - This score is higher for more relevant neighbors, ensuring that the model focuses on important connections and ignores irrelevant ones.

### Aggregation:
- After propagation, the **GraphSage Aggregator** is used to combine the entity’s current representation with the information from its neighbors:
  \[
  g(z^{(l-1)}_h, z^{(l-1)}_{N_h}) = \text{LeakyReLU}([z^{(l-1)}_h \| z^{(l-1)}_{N_h}] W^{(l)})
  \]
  - The concatenation operator `\|` combines the entity’s representation with its neighbors, and `W^{(l)}` is a learnable transformation matrix for layer `l`. The final updated embedding is passed through a LeakyReLU activation.

### Why This Step is Important:
This step allows the model to capture **higher-order relationships** in the graph, such as multi-hop paths that can reveal hidden connections between system entities. For example, it can highlight how a sensitive file is accessed by a process that later connects to an external IP.

## 3. Threat Detection

### Key Idea:
After obtaining the entity representations through the GNN, the next step is to classify interactions as normal or adversarial.

### Interaction Score:
- Given two entities `h` and `t`, we compute the inner product of their representations to predict the likelihood of an interaction:
  \[
  y_{ht} = z_h \cdot z_t
  \]
  - If the score `y_{ht}` is above a predefined threshold, the interaction is flagged as a potential cyber threat.

### Loss Function:
- To train the model, a **pairwise loss** is used to classify interactions:
  \[
  L_{\text{higher}} = \sum_{(h, r, t) \in \text{valid}} \max(0, y_{ht} - y_{\text{corrupt}} + \gamma)
  \]
  - This loss ensures that valid interactions (from normal audit records) are scored lower (benign) than unobserved interactions, which are treated as potentially malicious.

## 4. Final Objective Function

- The total loss function combines the first-order modeling and higher-order modeling:
  \[
  L = L_{\text{first}} + L_{\text{higher}} + \lambda \| \theta \|^2
  \]
  - Here, `λ` is a hyper-parameter that controls L2 regularization to prevent overfitting, and `\theta` refers to the model’s trainable parameters.

