# ShadeWatcher Code Analysis

## GNN.py Analysis 

### 1. Initialization:
- The GNN class is initialized with arguments, metadata, and optional pretrained embeddings.
- It parses these inputs to set up model parameters.

### 2. Building the Model:
- Creates placeholders for input data (entities, relations, positive/negative samples).
- Initializes model weights, including entity and relation embeddings.
- Builds the interaction model (GNN), which will learn from entity interactions.
- Sets up the knowledge graph embedding model (TransR/TransE/TransH).

### 3. Setting up Loss Functions:
- Defines loss functions for both the interaction prediction and knowledge graph embedding.

### 4. Data Input:
- The model doesn't directly load logs in this file. It expects data to be fed through the placeholders.
- The data typically includes:
  - Entity interactions (e.g., user-item interactions)
  - Knowledge graph triples (head entity, relation, tail entity)

### 5. Graph Construction:
- The initial graph structure is represented by the attention matrix `A_in`.
- This matrix is likely constructed from the entity interactions and knowledge graph data before being fed into the model.

### 6. Learning Entity Representations:
- The GNN layers (`_create_gcn_embed`, `_create_graphsage_embed`, or `_create_bi_inter_embed`) learn entity representations by aggregating information from neighboring entities.
- Multiple GNN layers allow for multi-hop information propagation.

### 7. Incorporating Knowledge Graph:
- The TransR/TransE/TransH models learn embeddings for entities and relations in the knowledge graph.
- This allows the model to capture semantic relationships between entities.

### 8. Attention Mechanism:
- The `update_attentive_A` method updates the attention weights based on the current state of the model.
- This dynamically adjusts the importance of different entity connections.

### 9. Training:
- The model alternates between training the interaction prediction (`train_inter`) and knowledge graph embedding (`train_kg`).
- It periodically updates the attention mechanism.

### 10. Producing Knowledge-Based Graph:
- The final knowledge-based graph is implicitly represented by:
  1. The learned entity embeddings, which capture both interaction patterns and knowledge graph information.
  2. The attention matrix, which represents the strength of connections between entities.
- While not explicitly outputting a graph, the model can use these to make recommendations or answer queries about entity relationships.


The knowledge-based graph is not a static structure that's produced once, but rather a dynamic representation continuously refined as the model learns. It is "produced" through the combination of:
1. The initial graph structure from entity interactions.
2. The semantic information from the knowledge graph.
3. The learned entity representations from the GNN.
4. The dynamically updated attention weights.



## driver.py Analysis



### 1. main() Function:
This is the primary function that controls the entire workflow. Key steps include:

- **Initialize settings**:
  ```python
  args = init_setting()
  ```
  Loads user input and logging configurations.

- **Set GPU/CPU device**:
  ```python
  os.environ['CUDA_VISIBLE_DEVICES'] = args.gpu_id
  ```

- **Load Dataset**:
  ```python
  meta_data = MetaData(args.dataset)
  data_generator = load_data_engine(args, meta_data)
  ```
  Loads and preprocesses the dataset using the `load_data_engine` function.

- **Load pre-trained KG embeddings (if specified)**:
  ```python
  if args.pretrain == 1:
      pretrain_embedding = load_pretrain_embedding(embedding_save_path)
  ```

- **Model Selection and Initialization**:
  ```python
  if args.model_type == 'gnn' and args.embedding_type in ['transr', 'transe', 'transh']:
      model = GNN(args=args, meta_data=meta_data, pretrain_embedding=pretrain_embedding)
  ```

- **Setup TensorFlow session**:
  ```python
  sess = tf.Session(config=tf_config)
  sess.run(tf.global_variables_initializer())
  ```

- **Reload model parameters (if specified)**:
  ```python
  if args.pretrain == 2:
      saver.restore(sess, ckpt.all_model_checkpoint_paths[0])
  ```

### 2. Training Loop:
The main training loop alternates between training the GNN and the KG embedding:
```python
for epoch in range(args.epoch):
    # Train GNN
    # Train KG embedding
    # Validation
    # Early stopping check
```

### 3. Testing:
After training, the model can be tested:
```python
if args.show_test:
    rel_stat = test(sess, model, data_generator, args.threshold)
```

### 4. Save Model:
The model can be saved at the end of the training process:
```python
if args.save_model and args.early_stop == False:
    save_saver.save(sess, weight_save_path, global_step=epoch)
```

### 5. Data Loading:
The main data loading happens with the following:
```python
meta_data = MetaData(args.dataset)
data_generator = load_data_engine(args, meta_data)
```
The `load_data_engine` function is responsible for:
- Loading the interaction data (e.g., user-item interactions).
- Loading the knowledge graph data.
- Preprocessing and formatting the data for the model.
- Creating data generators for batch training.

### 6. Additional Data Loading:
- Pre-trained embeddings are loaded if specified:
  ```python
  pretrain_embedding = load_pretrain_embedding(embedding_save_path)
  ```

- Model weights are reloaded for continued training if specified:
  ```python
  saver.restore(sess, ckpt.all_model_checkpoint_paths[0])
  ```
``` 