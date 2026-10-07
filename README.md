# AxioGlobe Product Intelligence Engine — GDL Output Architecture

> **Status: architecture / pre-production design**
>
> This repository name is historical. Under the unified AxioGlobe architecture, the former “GDL Validation Engine” is **not a standalone product** and GDL/GSM is **not** the master product database.

## Correct role in the ecosystem

The long-term capability is the **AxioGlobe Product Intelligence Engine**.

Its purpose is to transform manufacturer and supplier source material into canonical, evidence-backed **Product DNA**, then generate and validate downstream artifacts for the environments that need them.

GDL is the first Archicad artifact-output path.

AxioGlobe remains one platform with four external surfaces:

- Axverse
- Axio Supply
- Axio Build
- PEER

Product Intelligence sits underneath those surfaces as shared infrastructure.

## Canonical product truth

The canonical record is:

**Product DNA + provenance + evidence + version history + approval history**

GDL/GSM, Revit families, IFC and web/API representations are derived artifacts.

No downstream format is allowed to silently become the source of truth.

## Authoritative product-intelligence flow

1. Manufacturer / supplier source material
2. Document intelligence and structured extraction
3. Canonical Product DNA / evidence graph
4. Bounded AI enrichment and reasoning
5. Reusable validated parametric generators
6. Artifact assembly for the target environment
7. Deterministic validation and performance testing
8. Safe self-repair where permitted
9. Human exception handling where required
10. Manufacturer approval of product identity and declared data
11. Signed / versioned artifact publication
12. Consumption by Axverse, Project Pulse, AutoBid and other AxioGlobe services
13. Outcome feedback updates trust, confidence and future matching

## Core rules

- Every material fact retains provenance: source, location, extraction method, confidence and approval history.
- Missing or conflicting technical data becomes a visible exception.
- AxioGlobe must not invent missing manufacturer values.
- AI proposes, extracts and interprets inside bounded tasks.
- Deterministic systems verify schema, calculations, permissions and release gates.
- Manufacturer approval confirms commercial product identity and declared data; it is not statutory or professional certification.
- Generated code and uploaded files are treated as untrusted inputs and must be isolated for build/test.
- Revisions are versioned and must not silently overwrite artifacts already used in projects.

## Product DNA model

At minimum, the shared model needs to support:

- Product
- Manufacturer / supplier
- Source document
- Evidence
- Product attribute
- Geometry / parametric definition
- Performance data
- Compliance reference
- BIM artifact
- Validation
- Issue
- Decision
- Action
- Approval
- Version
- Publication
- Project usage
- Outcome

## GDL artifact path

For Archicad, the bounded flow is:

```text
Manufacturer evidence
      ↓
Document intelligence
      ↓
Canonical Product DNA
      ↓
Parametric definition
      ↓
GDL assembly
      ↓
Graphisoft-compatible build
      ↓
Deterministic validation
      ↓
Exception / repair loop
      ↓
Manufacturer approval
      ↓
Versioned Living BIM Object
      ↓
Axverse / project consumption
```

## Validation posture

The current engineering posture is deliberately bounded.

A valid pilot should constrain:

- Archicad version
- operating system
- geography
- one or two low-risk product families
- accepted source-document types
- validation rules
- publication gates

The platform should **not** make a universal “any PDF to perfect GSM automatically” claim.

## Architecture direction

The intended implementation direction is an event-driven modular monolith with isolated workers rather than premature microservices.

Shared platform infrastructure should provide:

- PostgreSQL canonical data
- durable jobs / execution state
- evidence and audit history
- permissions
- versioned contracts
- isolated artifact-generation workers
- deterministic validators
- publication / rollback controls

## Relationship to Axverse

Axverse does not own product truth.

Axverse requests and consumes product intelligence from AxioGlobe Core. When Archicad needs a BIM object, the Product Intelligence Engine produces or retrieves the appropriate validated GDL artifact for that Product DNA version.

## Repository scope

This repository is a **public architecture reference** for Product Intelligence and the GDL artifact path. Production source code, secrets, customer data and proprietary implementation details must remain in controlled private repositories.

---
AxioGlobe (Pty) Ltd · South Africa · https://axioglobe.co.za
