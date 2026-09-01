# Ecosystem — the graph path to success

```mermaid
flowchart LR
  subgraph PRODUCER["PRODUCER — Metanorma estate"]
    ADOC["Metanorma authoring"] --> DOC["metanorma-document"]
    DOC -- "cite_as + identity" --> MKOX["MKO exporter (MN 116)"]
    MKOX --> MKOGEM["metanorma-mko (proposed)<br/>gem + @metanorma/mko-schema"]
  end
  subgraph CONTRACTS["CONTRACTS"]
    MKOGEM --> BUNDLE["MKO bundle"] --> CHUNKS["chunk metadata schema<br/>(serving profile)"]
    BUNDLE --> ANSWERC["answer contract v2"]
  end
  subgraph PACKAGE["metanorma/ai"]
    CONTRACTS --> PKG["architecture · pipeline ·<br/>deploy · configuration · mapping"]
  end
  subgraph DEPLOY["DEPLOYMENTS"]
    PKG --> OIML["OIML reference (live)"]
    PKG --> OTHERS["your deployment"]
  end
  subgraph SYSTEM["THE SYSTEM"]
    INGEST["ingest: export→chunk→enrich→index"] --> INDEX[("vectors · lexical ·<br/>payloads · graph")]
    INDEX --> SERVE["serve: understand∥retrieve→<br/>rank→generate→verify"]
    SERVE --> UI["UI: citations · typed blocks"]
  end
  OIML --> SYSTEM
  EVAL["eval: golden+witness · faithfulness ·<br/>variance · feedback"] -. gates .-> SERVE
  SERVE -. surfaces corpus defects .-> UP["upstream fixes"] --> PRODUCER
```

The critical path: producer emits the contract → the format gets a stable
versioned home → deployments consume it → evaluation gates every change →
serving surfaces corpus defects → upstream fixes → a better corpus. The
reference deployment proves each hop with measurements (see README).
