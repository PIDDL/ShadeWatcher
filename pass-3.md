# Weekly Project Progress and Issues Report

## Progress
- Successfully parsed the DARPA dataset for further processing.
- Generated initial knowledge graph files, including:
  - `procfact_tmp.txt` for process entities.
  - `filefact_tmp.txt` for file entities.
  - `socketfact_tmp.txt` for socket entities.
  - `edgefact_tmp.txt` for relationship/edge data between entities.
- Enhanced multi-threading setup with error handling to improve stability and prevent crashes when incorrect input or no input is given for the thread count.

### Darpa Parsed Dataset 

 ![darpa passed](images/darpa%20parsd.png)
 

### File contents

| Function        | Purpose                  | Output Files                     | Details Saved in Each File                                     | Encoding                                                                                  |
|-----------------|--------------------------|----------------------------------|----------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| `StoreProc`     | Store process entities   | `procfact_tmp.txt`               | `id pid exe ppid args`                                        | Label encoding (type `1` in `nodefact_tmp.txt`)                                           |
|                 |                          | `nodefact_tmp.txt`               | `id 1` (indicates process)                                    |                                                                                           |
| `StoreFile`     | Store file entities      | `filefact_tmp.txt`               | `id name version`                                             | Label encoding (type `2` in `nodefact_tmp.txt`)                                           |
|                 |                          | `nodefact_tmp.txt`               | `id 2` (indicates file)                                       |                                                                                           |
| `StoreSocket`   | Store socket entities    | `socketfact_tmp.txt`             | `id name`                                                     | Label encoding (type `3` in `nodefact_tmp.txt`)                                           |
|                 |                          | `nodefact_tmp.txt`               | `id 3` (indicates socket)                                     |                                                                                           |
| `StoreEdge`     | Store edge relationships | `edgefact_tmp_<file_id>.txt`     | `e_id n1_hash n2_hash relation sequence session timestamp` | Label encoding (`relation`), String serialization (`e_id`, `sequence`, `n1_hash`, `n2_hash`) |



## Issues
- Encountered an error when using `std::stoi` for thread count conversion, which would cause crashes if an invalid or missing thread count argument was provided. Added `try-catch` blocks to handle `std::invalid_argument` and `std::out_of_range` exceptions, allowing for a safe fallback to the default thread count.
- Missing interaction extraction code needed to generate critical output files (`entity2id.txt`, `inter2id_o.txt`, and `train2id.txt`). This issue has halted some progress as these files are essential for further stages of the project.
