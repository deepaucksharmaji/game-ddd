# Mermaid Syntax Test

This file tests a sample of the fixed syntax to ensure it renders correctly.

## Test 1: Multi-line node labels with quotes

```mermaid
flowchart TD
    A["Line 1
    Line 2
    Line 3"]
    B["Single line"]
    A --> B
```

## Test 2: Subgraph with quoted labels

```mermaid
flowchart LR
    subgraph TEST["Test Subgraph"]
        N1["Node 1
        With multiple lines
        And emojis 📊"]
        N2["Node 2
        Also multi-line"]
    end
    N1 --> N2
```

## Test 3: Decision node with multi-line

```mermaid
flowchart TD
    A["Input"]
    B{"Decision
    Line 1?
    Line 2?"}
    C["Output"]
    A --> B
    B --> C
```
