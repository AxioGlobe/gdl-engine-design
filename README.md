# AxioGlobe GDL Validation Engine

> **Status: Pre-build technical design phase**  
> This repository documents the design of the GDL Validation Engine. No production code exists yet.

## What Is the GDL Validation Engine?

The GDL Validation Engine converts building product manufacturer data — typically delivered as PDF datasheets, CAD files, and performance certificates — into verified parametric BIM objects (GDL format) for placement inside ArchiCAD.

Every building product manufacturer in the world produces technical documentation. None of it is in a format that an architect can place directly into ArchiCAD. The GDL Engine solves this — automatically, at scale, with AI verification at every step.

## The Five-Pass Validation Process

```
PASS 1: DOCUMENT INTELLIGENCE (Claude Sonnet)
  Input:  Manufacturer PDF (datasheet, EPD, fire cert, acoustic cert)
  Output: Structured data extraction
          - Product name and code
          - All dimensions (nominal, tolerance ranges)
          - Thermal performance (U-value, R-value, lambda)
          - Fire classification (BS EN 13501, SANS 10177)
          - Acoustic rating (Rw, Rw+Ctr)
          - Embodied carbon (kg CO2e/kg from EPD)
          - Current price and lead time
          - Manufacturer contact and warranty terms

PASS 2: GEOMETRY EXTRACTION (Gemini 1.5 Flash)
  Input:  CAD drawings or PDF dimensioned drawings
  Output: Parametric geometry definition
          - Base geometry (L x W x H)
          - Parametric ranges (min/max dimensions)
          - Connection points and hotspot positions
          - 2D symbol for plan view
          - 3D geometry for ArchiCAD model view

PASS 3: PERFORMANCE VERIFICATION (Claude Sonnet)
  Input:  Extracted performance data from Pass 1
  Output: Verification against claimed standards
          - Thermal: CIBSE Guide A / SANS 10400 XA compliance
          - Fire: BS EN 13501-1 classification verified
          - Acoustic: ISO 140 / SANS 10053 compliance
          - Carbon: GWP aligned with EN 15804 EPD standard
          - PASS / FAIL / NEEDS_REVIEW status per attribute

PASS 4: GLOBAL CODE COMPLIANCE (Claude Sonnet)
  Input:  Product type, performance data, target jurisdictions
  Output: Jurisdiction-specific compliance status
          - South Africa: SANS 10400 Part XA/S/T/W
          - United Kingdom: Part L / Part B / Approved Documents
          - European Union: EN Eurocodes
          - United Arab Emirates: UAE Fire and Life Safety Code
          - Australia: NCC (National Construction Code)
          - Compliance badge per jurisdiction

PASS 5: PARAMETRIC RANGE TESTING (Gemini)
  Input:  GDL object from Pass 2 + compliance data from Pass 3+4
  Output: Tested parametric object
          - Object renders correctly at all parametric values
          - Property sets update correctly on parameter change
          - BOQ quantities calculate correctly
          - IFC export valid at all parameter values
          - PASS: object published to ArchiCAD library
          - FAIL: object returned to manufacturer with specific feedback

```

## Processing Pipeline

```
Manufacturer                   AxioGlobe GDL Engine              ArchiCAD
    │                                   │                            │
    │── Upload PDF + CAD ──────────────>│                            │
    │                           Pass 1: Claude reads PDF             │
    │                           Pass 2: Gemini extracts geometry     │
    │                           Pass 3: Claude verifies performance  │
    │                           Pass 4: Claude checks 180+ codes     │
    │                           Pass 5: Gemini tests parametrics     │
    │                                   │                            │
    │<── Verification report ───────────│                            │
    │    (PASS / FAIL / CORRECTIONS)    │                            │
    │                                   │                            │
    │                           Publish to GDL Library ────────────>│
    │                                   │            Object available│
    │                                   │            in ArchiCAD     │
    │                                   │            library panel   │
```

## API Endpoints

```
POST /api/v1/gdl/submit
  Description: Submit product for GDL validation
  Auth: manufacturer_api_key
  Body: {
    product_name: string,
    product_code: string,
    manufacturer_id: uuid,
    documents: [
      { type: 'datasheet', file: base64, mime: 'application/pdf' },
      { type: 'cad_drawing', file: base64, mime: 'image/dxf' },
      { type: 'epd', file: base64, mime: 'application/pdf' },
      { type: 'fire_certificate', file: base64, mime: 'application/pdf' }
    ],
    target_jurisdictions: ['ZA', 'GB', 'AE', 'AU', 'EU'],
    product_category: 'facade_system' | 'structural_element' | 'mep_component' | ...
  }
  Response: { job_id: uuid, estimated_processing_time: '45 minutes' }

GET /api/v1/gdl/status/:job_id
  Description: Check processing status
  Response: {
    status: 'processing' | 'review_required' | 'published' | 'rejected',
    pass_results: { pass1: {...}, pass2: {...}, pass3: {...}, pass4: {...}, pass5: {...} },
    gdl_object_url: string | null,
    feedback: string | null
  }

GET /api/v1/gdl/library
  Description: Search published GDL objects
  Query: category, manufacturer, jurisdiction, performance_min
  Response: { products: [{...}], total: number }
```

## Database Schema (Core Tables)

```sql
-- Manufacturers
CREATE TABLE manufacturers (
  id UUID PRIMARY KEY,
  company_name VARCHAR(255),
  registration_number VARCHAR(100),
  country_code CHAR(2),
  api_key_hash VARCHAR(255),
  created_at TIMESTAMP
);

-- GDL Products
CREATE TABLE gdl_products (
  id UUID PRIMARY KEY,
  manufacturer_id UUID REFERENCES manufacturers(id),
  product_name VARCHAR(255),
  product_code VARCHAR(100),
  category VARCHAR(100),
  status VARCHAR(50), -- processing/review/published/rejected
  gdl_object_url VARCHAR(500),
  thermal_u_value DECIMAL(6,3),
  fire_classification VARCHAR(50),
  acoustic_rw INTEGER,
  embodied_carbon DECIMAL(10,3),
  current_price_gbp DECIMAL(10,2),
  lead_time_weeks INTEGER,
  published_at TIMESTAMP
);

-- Jurisdiction Compliance
CREATE TABLE product_compliance (
  id UUID PRIMARY KEY,
  product_id UUID REFERENCES gdl_products(id),
  jurisdiction_code CHAR(2),
  compliance_status VARCHAR(20), -- compliant/non_compliant/not_checked
  applicable_standard VARCHAR(100),
  clause_reference VARCHAR(100),
  verified_at TIMESTAMP
);

-- Specification Events
CREATE TABLE specification_events (
  id UUID PRIMARY KEY,
  product_id UUID REFERENCES gdl_products(id),
  project_type VARCHAR(100),
  region VARCHAR(100),
  specified_at TIMESTAMP,
  quantity DECIMAL(10,2),
  unit VARCHAR(20)
);
```

## Claude and Gemini Integration

The GDL Engine uses two AI models for different tasks based on their respective strengths:

**Claude Sonnet (Anthropic)** — used for:
- Reading and interpreting technical PDF documents
- Generating GDL scripting code from extracted specifications
- Interpreting building code clauses and applying them to specific products
- Writing verification reports in plain language for manufacturers

**Gemini 1.5 Flash (Google)** — used for:
- Visual interpretation of dimensioned drawings
- Geometric extraction from CAD diagrams
- Parametric range testing at scale
- Multi-document correlation across multiple PDFs for one product

## Why This Has Not Been Built Before

The GDL Validation Engine requires three capabilities that became available simultaneously only in 2024-2025:

1. **Large context window models** (Claude Sonnet, Gemini 1.5) capable of reading a 40-page technical datasheet in one pass
2. **Code generation quality** sufficient to produce valid GDL scripting from a natural language specification
3. **Vision capability** sufficient to interpret a dimensioned engineering drawing and extract parametric geometry

All three of these capabilities were below the required threshold before 2024. They are above the threshold now. The GDL Engine is buildable today in a way it was not two years ago.

## Company

**AxioGlobe (PTY) Ltd** — Registration: 2026/437531/07  
Polokwane, Limpopo, South Africa  
axioglobe.co.za · info@axioglobe.co.za

---
*Pre-build technical design. No production code yet.*
