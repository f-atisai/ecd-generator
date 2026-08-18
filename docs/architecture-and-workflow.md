# ECD Generator — Architecture and Workflow

## 1. Purpose

This document provides a high-level view of the architecture and operational workflow of ECD Generator.

ECD Generator is a clinical study-build automation application that uses information from a Medidata Rave Architect Loader Spreadsheet (ALS) to generate a structured Excel workbook for UAT preparation.

This document focuses on:

* where ECD Generator fits within the study-build validation process;
* the user-facing workflow;
* the application's system boundaries;
* the major architectural responsibilities; and
* the design principles supporting reuse across studies.

Production source code, generation rules, metadata mappings, parsing logic, and implementation-specific algorithms are intentionally outside the scope of this public case study.

For a project overview, see the repository [README](../README.md).

---

## 2. Workflow Context

Clinical study builds contain programmed behavior that must be reviewed and validated before deployment.

Depending on the study, this can include:

* CRF data-entry validation;
* visit and form workflow behavior;
* dynamic field behavior;
* data derivations;
* custom-function-driven logic; and
* field-level system validation.

UAT provides a controlled mechanism for confirming that this programmed behavior operates as expected.

ECD Generator supports this process by automating a substantial portion of the **validation-script preparation** activity.

It does not execute UAT and does not modify the Rave study build.

---

## 3. Previous Manual Workflow

Before automation, preparation of the validation workbook was primarily manual.

A Lead Data Manager reviewed the applicable study specifications, identified the programmed behavior requiring validation, and translated that information into the required validation template.

At a high level:

```text
Study Build
     │
     ▼
Study Specifications
     │
     ▼
Manual Review
     │
     ▼
Validation Script Preparation
     │
     ▼
Peer Review
     │
     ▼
UAT Execution
```

This required repeated interpretation and manual preparation for each study.

For larger study builds, preparing the validation workbook could require approximately one working day.

---

## 4. ECD Generator Workflow

ECD Generator introduces an automated preparation step between the study build and validation review.

```text
Rave Study Build
       │
       ▼
Architect Loader Spreadsheet
       │
       ▼
┌───────────────────────────┐
│       ECD Generator       │
│                           │
│   Automated Preparation   │
└─────────────┬─────────────┘
              │
              ▼
     Generated ECD Workbook
              │
              ▼
         Human Review
              │
              ▼
         UAT Execution
```

The application automates repeatable preparation activities while preserving human review and validation judgment.

---

## 5. User Workflow

The application exposes a deliberately simple workflow through Streamlit.

![ECD Generator user interface](../screenshots_and_demo/ecd-generator-ui.png)

*User interface for providing the Rave ALS, protocol number, and ECD version.*

The user:

1. uploads the Architect Loader Spreadsheet;
2. enters the study protocol number;
3. enters the required ECD version;
4. initiates generation; and
5. downloads the resulting Excel workbook.

The generated workbook is then reviewed according to the applicable study validation process before UAT execution.

The application does not require the user to manually configure individual study checks before generation.

---

## 6. System Context

ECD Generator operates as a standalone supporting tool within the broader study-build validation workflow.

```text
┌───────────────────────────┐
│      Medidata Rave        │
│        Study Build        │
└─────────────┬─────────────┘
              │
              │ Study-Build Information
              ▼
┌───────────────────────────┐
│                           │
│       ECD Generator       │
│                           │
│  Processing & Generation  │
│                           │
└─────────────┬─────────────┘
              │
              │ Validation Workbook
              ▼
┌───────────────────────────┐
│       Human Review        │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│       UAT Execution       │
└───────────────────────────┘
```

The application consumes study-build information and produces a validation artifact.

It does not create, update, or deploy study metadata within Medidata Rave.

---

## 7. Inputs and Output

The user provides three inputs:

* a compatible Rave Architect Loader Spreadsheet;
* the study protocol number; and
* the required ECD version.

These inputs are processed by ECD Generator to produce a single Excel validation workbook.

```text
ALS + Study Information
          │
          ▼
    ECD Generator
          │
          ▼
 Generated ECD Workbook
```

The application was designed so that protocol-specific configuration is not required for compatible study builds.

---

## 8. Conceptual Architecture

At a high level, ECD Generator separates the user interface, application processing, and document-generation responsibilities.

```text
┌────────────────────────────┐
│       User Interface       │
│        (Streamlit)         │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│                            │
│   Application Processing   │
│                            │
└──────────────┬─────────────┘
               │
               ▼
┌────────────────────────────┐
│    Workbook Generation     │
└────────────────────────────┘
```

### User Interface

The Streamlit interface handles user interaction, including:

* accepting the study file;
* collecting document-level information;
* initiating generation; and
* returning the resulting workbook.

### Application Processing

The processing layer interprets the supplied study-build information and prepares the validation content required for the output workbook.

The internal processing rules and transformation logic are intentionally not documented publicly.

### Workbook Generation

The generated content is populated into a predefined Excel workbook structure.

`openpyxl` is used for Excel workbook operations.

---

## 9. Validation Workbook

ECD Generator produces a workbook containing six validation categories:

```text
ECD Workbook
│
├── Validation Scripts
├── Matrix Scripts
├── Screen Dynamics
├── Derivations
├── Custom Functions
└── System Checks
```

![Generated ECD workbook](../screenshots_and_demo/generated-workbook.png)

*Example of the generated multi-sheet validation workbook using synthetic study information.*

### Validation Scripts

Validation content associated with edit checks programmed against study CRF data.

### Matrix Scripts

Validation content associated with visit and form workflow behavior.

### Screen Dynamics

Validation content associated with dynamic form behavior.

### Derivations

Validation content associated with study data derivations.

### Custom Functions

Provides visibility into applicable custom-function-driven study logic.

### System Checks

Validation content associated with applicable field-level system behavior.

These categories reflect the validation artifact produced by the application. The internal rules used to identify, interpret, and generate the corresponding content are not part of the public documentation.

---

## 10. Cross-Study Reusability

Cross-study reuse was a core design requirement.

ECD Generator was designed around the common structure of compatible Rave study-build information rather than around individual protocols.

Conceptually:

```text
Study A ─┐
Study B ─┼──► ECD Generator ──► Study-Specific ECD Workbook
Study C ─┘
```

The same application workflow can therefore be used across compatible studies without maintaining a separate implementation for each protocol.

This approach was successfully tested against multiple Rave studies during development.

---

## 11. Human Oversight

ECD Generator automates validation-script preparation, not validation judgment.

The generated workbook remains subject to human review before UAT execution.

Reviewers remain responsible for determining whether:

* the generated validation content appropriately represents the study build;
* validation coverage is sufficient;
* appropriate study-specific test data are used;
* additional validation scenarios are required; and
* UAT results meet the applicable study requirements.

The intended operating model is therefore:

```text
Automated Preparation
         │
         ▼
    Human Review
         │
         ▼
    UAT Execution
```

This preserves human oversight while reducing repetitive preparation effort.

---

## 12. Before and After

| Manual Workflow                              | ECD Generator Workflow                    |
| -------------------------------------------- | ----------------------------------------- |
| Manual review and preparation                | Automated workbook preparation            |
| Repeated document preparation for each study | Common workflow across compatible studies |
| Manual organization of validation content    | Standardized workbook structure           |
| Approximately one working day                | Less than three minutes during testing    |
| Human review required                        | Human review retained                     |

The application does not eliminate the controlled validation process.

Its purpose is to move effort away from repetitive document preparation and toward activities requiring human review and judgment.

---

## 13. Technology

ECD Generator uses a deliberately lightweight technology stack:

* **Python** — application development
* **Pandas** — data processing
* **Streamlit** — user interface
* **openpyxl** — Excel workbook operations
* **Microsoft Excel** — final validation artifact

The application was designed as a focused workflow automation tool rather than a distributed software platform.

---

## 14. Design Principles

### Reusability

The application should support compatible study builds through a common workflow.

### Standardization

Generated validation artifacts should follow a consistent workbook structure.

### Minimal User Input

The user should provide only the information required to initiate generation and identify the resulting artifact.

### Separation of Concerns

User interaction, application processing, and workbook generation should remain conceptually distinct.

### Human-in-the-Loop

Automation should reduce repetitive preparation without replacing review and validation judgment.

### Controlled Disclosure

The public portfolio should demonstrate the application's engineering value without exposing the implementation rules required to reproduce the generation engine.

---

## 15. System Boundaries

ECD Generator has a deliberately defined scope.

### The application does:

* consume compatible Rave study-build information;
* generate structured validation content;
* organize that content into a standardized workbook; and
* provide the generated workbook for review.

### The application does not:

* modify the Rave study build;
* execute UAT;
* approve validation coverage;
* replace peer review;
* make study deployment decisions; or
* replace study-specific validation judgment.

These boundaries preserve the distinction between **workflow automation** and **controlled validation activities**.

---

## 16. Demonstrated Outcome

ECD Generator was completed and successfully tested against multiple Rave studies.

During testing:

* **Manual preparation:** approximately one working day
* **ECD Generator:** less than three minutes

The project demonstrated that a substantial portion of validation-script preparation could be converted from repetitive manual work into a reusable automated workflow.

The application was not subsequently incorporated into the operational study-build process because existing procedures had recently undergone significant changes and adoption would have required another round of process updates.

The technical implementation itself reached a complete and functional state.

---

## 17. Public Documentation Scope

ECD Generator is presented publicly as a technical portfolio case study rather than an implementation specification.

This documentation intentionally describes:

* the problem being addressed;
* the application's role within the validation workflow;
* its user-facing behavior;
* its conceptual architecture;
* its output structure; and
* its demonstrated outcome.

It intentionally excludes:

* production source code;
* parsing and interpretation rules;
* generation algorithms;
* metadata-to-output mappings;
* test-case determination logic;
* implementation-specific business rules;
* proprietary study metadata; and
* confidential organizational assets.

Representative synthetic materials are available in [`samples/`](../samples/). These materials are deliberately limited and are intended to demonstrate the application rather than document its complete processing behavior.
