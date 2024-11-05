# ShadeWatcher PASS 2

## Issues
- Installation setup for parser and recommendation took time.
- `libqxx` had issues while cloning from Git, so had to look into issues posted in the repo on GitHub.
- Had to clone `librdkafka` through SSH instead of HTTPS as HTTPS was showing an EOF error.
- Neo4j database connection error.
- DARPA dataset downloaded but showing errors in parsing, so working on it now.

## Progress
- Gave a deep dive into kg.cpp which has the code for knowledge graph construction; interaction extraction not found.
- Set up Vagrant virtual machine on ISPL, and it took some time to learn and navigate through it.
- Successfully completed the installation setup for parser and recommendation.
- Successfully compiled all C++ files and parsed `audibeat` data; output matches that in the repo and paper.


 ![audibeat parse](images/audibeat%20parse%20result.png)

## kg.cpp Understanding
- **Node Creation and Storage:**
  - Each node (Process, File, Socket) is created with specific properties.
  - A unique hash is generated for each node based on its properties.
  - Nodes are stored in separate tables based on their type (ProcNodeTable, FileNodeTable, SocketNodeTable).
  - Hash tables enable O(1) lookup time for any node using its hash.
  
- **Edge Creation and Storage:**
  - Edges represent relationships between nodes (e.g., process reading file).
  - Each edge contains: source node, target node, relationship type, sequence number, session ID.
  - A unique edge hash is created from these properties.
  - Edges are initially stored in KGEdgeList, then moved to KGEdgeTable with their hashes.
  - KGEdgeTable provides quick lookup for any edge using its hash.
  
- **Interaction Tracking:**
  - FileInteractionTable tracks all interactions involving files.
  - ProcInteractionTable tracks all interactions involving processes.
  - Each interaction table maps an entity's hash to a vector of all its edges.
  - Purpose: Quick access to all interactions of a specific entity.
  - Used for: Security monitoring, behavior analysis, forensics.
  
- **Why Track Interactions:**
  - **Security:** Monitor file access patterns and process behavior.
  - **Analysis:** Study behavior patterns and relationships.
  - **Performance:** Quick access to entity-specific interactions.
  - **Forensics:** Reconstruct event sequences and investigate incidents.
  - **History:** Maintain complete interaction timeline for each entity.
  
- **Table Purposes:**
  - **Node Tables:** Organize nodes by type, enable quick lookup.
  - **Edge Table:** Store all relationships in the graph.
  - **Interaction Tables:** Group edges by involved entities.
  - All tables use hashes for efficient lookup and relationship tracking.
