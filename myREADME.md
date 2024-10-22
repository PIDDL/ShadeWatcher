# Code Analysis Progress

## High-Level Analysis

- Had a meeting with Ahmed, and he asked me to go through the code first to develop an understanding, as it's non-producible due to a missing component.
- Have gone through the coding files to understand what's happening, performing a high-level analysis of the code.
- Cross-checking if everything mentioned in the paper has been applied in the code.
- code analysis done manually and through cursor AI matches whats in ReproducibilityAnalysis.md in PIDDL repo.

## Missing Component

- The missing component is **interaction extraction**.

- ![missing comp2](images/missing%20comp%202.png)

- ![missing comp](images/missing%20component.png)


- Currently looking for the missing component in **KGAT** (link sent by Dr. Ashish).

- Missing component not found as Shadewatcher divides its provenance graph into subgraphs for better anomaly detection, while a high level overview of KGAT is that it focuses on using the entire graph's high-order relationships for predicting recommendations.
