# ShadeWatcher PASS 2

## Issues
- Installation setup for parser and recommendation took time.
- `libqxx` had issues while cloning from Git, so had to look into issues posted in the repo on GitHub.
- Had to clone `librdkafka` through SSH instead of HTTPS as HTTPS was showing an EOF error.
- Neo4j database connection error.
- DARPA dataset downloaded but showing errors in parsing, so working on it now.

## Progress
- Set up Vagrant virtual machine on ISPL, and it took some time to learn and navigate through it.
- Successfully completed the installation setup for parser and recommendation.
- Successfully compiled all C++ files and parsed `audibeat` data; output matches that in the repo and paper.
