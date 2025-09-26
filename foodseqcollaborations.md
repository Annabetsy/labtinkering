```mermaid
flowchart TD
  %% ==============================
  %% FoodSeq: Collaboration → Paper
  %% ==============================

  %% Visual classes
  classDef action fill:#e8f0fe,stroke:#5a8dee,color:#111;
  classDef decision fill:#fff4e5,stroke:#ffa726,color:#111;
  classDef data fill:#e6f4ea,stroke:#34a853,color:#111;
  classDef handoff fill:#f1f3f4,stroke:#9aa0a6,color:#111,stroke-dasharray: 6 4;
  classDef status fill:#fafafa,stroke:#bdbdbd,color:#111,stroke-dasharray: 2 2;

  start([Start]):::action

  %% ------------------------------
  %% 1) Collaboration & Scoping
  %% ------------------------------
  subgraph S1[Collaboration & Scoping]
    direction TB
    A1[Inbound/Outbound collaborator contact<br/>& initial scoping]:::action
    A2[Approvals & Agreements:<br/>IRB/DUA/MTA, budget, SOW, DMP]:::action
    A3[Assign Project Code (e.g., FSQ-PRJ123)]:::data
    A4[Set up project spaces:<br/>Basecamp card → status "Acquiring"<br/>Box folder, GitHub repo (mb-pipeline tag), RACI]:::action
    A1 --> A2 --> A3 --> A4
  end

  %% ------------------------------
  %% 2) Intake & Sample Logistics
  %% ------------------------------
  subgraph S2[Intake & Sample Logistics]
    direction TB
    B1[Kit prep & barcoding (if applicable)]:::action
    B2[Ship kits / receive samples]:::action
    B3[Accession & label samples:<br/>assign LabIDs; chain-of-custody]:::action
    B4[Capture metadata in DB/LIMS<br/>(sample sheet, plate map)]:::data
    B5[Randomize extraction order]:::action
    B6[Update Basecamp → "Needs Extraction and/or Randomization"]:::status
    B1 --> B2 --> B3 --> B4 --> B5 --> B6
  end

  %% ------------------------------
  %% 3) Wet Lab (FoodSeq)
  %% ------------------------------
  subgraph S3[Wet Lab (FoodSeq)]
    direction TB
    C1[DNA extraction (batched)<br/>+ controls: blanks/positives]:::action
    C2[DNA QC (yield & purity)]:::action
    D1{Pass DNA QC?}:::decision
    C3[Re-extract / document issue]:::action
    C4[PCR amplification of FoodSeq markers<br/>with dual indices]:::action
    C5[Library QC (size/conc)]:::action
    D2{Amplicon ok?}:::decision
    C6[Re-amplify / adjust conditions]:::action
    C7[Normalize & pool libraries]:::action
    C8[Plan sequencing run & sample sheet]:::action
    C9[Basecamp → "In Progress"]:::status

    C1 --> C2 --> D1
    D1 -- "No" --> C3 --> C2
    D1 -- "Yes" --> C4 --> C5 --> D2
    D2 -- "No" --> C6 --> C5
    D2 -- "Yes" --> C7 --> C8 --> C9
  end

  %% ------------------------------
  %% 4) Sequencing & Data Management
  %% ------------------------------
  subgraph S4[Sequencing & Data Management]
    direction TB
    E1[Illumina run executed]:::action
    E2[Demultiplex (BCL→FASTQ)]:::action
    E3[Run-level QC (yield, Q30, balance)]:::action
    D3{Run QC ok?}:::decision
    E4[Re-pool / re-sequence]:::action
    E5[Transfer FASTQs to server:<br/>/srv/seq-runs/&lt;RunID&gt;/]:::data
    E6[Checksums & integrity verified]:::action
    E7[Project copy/symlink:<br/>/srv/projects/&lt;ProjCode&gt;/raw/]:::data
    E8[Register run + paths in DB/LIMS]:::data
    E9[Basecamp → "Waiting for Bioinformatics"]:::status

    E1 --> E2 --> E3 --> D3
    D3 -- "No" --> E4 --> E1
    D3 -- "Yes" --> E5 --> E6 --> E7 --> E8 --> E9
  end

  %% ------------------------------
  %% 5) Bioinformatics (mb-pipeline)
  %% ------------------------------
  subgraph S5[Bioinformatics (mb-pipeline)]
    direction TB
    F0[Launch pipeline with tagged release<br/>(e.g., vX.Y.Z); capture env (conda/containers)]:::action
    F1[Read QC & trimming (fastp/cutadapt)]:::action
    F2[Denoise/ASV & chimera removal<br/>(DADA2/VSEARCH)]:::action
    F3[Taxonomic assignment against Food DB]:::action
    F4[Control checks & contaminant filtering<br/>(e.g., decontam thresholds)]:::action
    F5[Feature table + taxonomy table built]:::data
    F6[Join with sample metadata; ID QC]:::action
    F7[Exports:<br/>counts.tsv, taxa.tsv, summary.html, run_report.json]:::data
    F8[Load results to relational DB]:::data
    D4{Any QC flags?}:::decision
    F9[Refine params / re-run subset<br/>(track changes)]:::action
    F10[Basecamp → "In Progress" (Analysis stage)]:::status

    F0 --> F1 --> F2 --> F3 --> F4 --> F5 --> F6 --> F7 --> F8 --> D4
    D4 -- "Yes" --> F9 --> F1
    D4 -- "No"  --> F10
  end

  %% ------------------------------
  %% 6) Analysis, Reporting & Collaboration
  %% ------------------------------
  subgraph S6[Analysis, Reporting & Collaboration]
    direction TB
    G1[Exploratory & hypothesis-driven analyses<br/>(meta-analysis across projects as needed)]:::action
    G2[Generate figures & tables]:::data
    G3[Draft technical report; share with collaborator]:::action
    D5{Meets study goals?}:::decision
    G4[Iterate analyses / collect more data]:::action
    G5[Stakeholder review meeting & sign-off]:::action

    G1 --> G2 --> G3 --> D5
    D5 -- "No" --> G4 --> G1
    D5 -- "Yes" --> G5
  end

  %% ------------------------------
  %% 7) Manuscript & Publication
  %% ------------------------------
  subgraph S7[Manuscript & Publication]
    direction TB
    H1[Manuscript outline & writing plan]:::action
    H2[Methods from protocols & pipeline;<br/>results + figures; QA pass]:::action
    H3[Reproducible repo (GitHub) + data snapshot<br/>(archive/DOI)]:::data
    H4[Data deposition (SRA/ENA) + metadata;<br/>IRB/DUA compliance]:::action
    H5[Preprint & journal submission]:::action
    D6{Accepted?}:::decision
    H6[Revise & resubmit / new journal]:::action
    H7[Publish; share; archive; update KB]:::action
    H8[Basecamp → "Done"; close project]:::status

    H1 --> H2 --> H3 --> H4 --> H5 --> D6
    D6 -- "No" --> H6 --> H5
    D6 -- "Yes" --> H7 --> H8
  end

  %% ------------------------------
  %% Cross-phase flow
  %% ------------------------------
  start --> A1
  A4:::handoff --> B1
  B6:::handoff --> C1
  C9:::handoff --> E1
  E9:::handoff --> F0
  F10:::handoff --> G1
  G5:::handoff --> H1
```
