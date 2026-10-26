# Phase-1 AI Interaction Log — ELM Module Generation (Model 2: DeepSeek Coder V2)
**Student:** harshivpatel4061  
**Model:** deepseek-coder-v2:latest  
**Runtime:** Ollama  
**Agent/Tool:** Continue (VS Code extension, v2.0.0)  
**Date:** October 26, 2026  

---

## Hardware Configuration
| Item | Detail |
|------|--------|
| OS | macOS (Apple Silicon) |
| CPU / Chip | Apple M5 |
| RAM | 16 GB unified memory |
| GPU | Apple GPU (integrated, unified memory — no discrete VRAM) |

---

## Model Configuration
| Item | Detail |
|------|--------|
| Model | deepseek-coder-v2:latest |
| Model family | DeepSeek |
| Parameter size | 16B MoE (Mixture of Experts) |
| Quantization | Q4_0 (Ollama default) |
| Model size on disk | 8.9 GB |
| Context length | 16384 tokens |
| Runtime | Ollama (local) |
| Coding agent | Continue VS Code extension (v2.0.0) |

---

## Prompt 1 — Initial ELM Implementation Request

**Role:** User  
**Sent to:** deepseek-coder-v2 via Continue agent mode  

```
Inspect the provided Fusion ERP base repository and the supplied Employee Leave
Management (ELM) specification artifacts.

Implement the ELM module as an integrated module of the existing Fusion ERP system.

Reuse the authentication/login, role-management mechanism, common UI/layout/navigation,
backend structure, database configuration, shared components, and other common
functionality already available in the Fusion Base Code.

Do not unnecessarily rebuild or replace common functionality already present in Fusion.

Implement the required ELM frontend and backend functionality, including:
- appropriate data models
- services
- APIs/routes
- UI components/pages
- role-dependent behavior
- database changes
- integration with the existing Fusion structure

First understand the existing repository and the supplied specifications, and then make
the necessary repository-level changes.

Attempt to build/run or otherwise verify the resulting implementation where possible.
```

**Specification artifacts provided as context:**
- `docs/01_ELM_Problem_Statement_v1.4_FINAL.docx`
- `docs/02_ELM_Functional_Requirement_Specification_v1.5_FINAL.xlsx`
- `docs/03_ELM_Use_Case_Specifications_v1.4_FINAL.docx`
- `docs/04_ELM_Business_Rule_Specifications_v1.4_FINAL.docx`
- `docs/06_ELM_Traceability_Matrix_v1.5_FINAL.xlsx`
- `docs/06_ELM_Workflow_Specifications_v1.6_FINAL.docx`

---

## Model Response Summary

DeepSeek Coder V2 (16B MoE) generated a more comprehensive implementation than
stable-code:3b, including:
- Full Django models: `LeaveRequest`, `LeaveBalance`, `LeaveType`, `LeavePolicy`
- REST API views using Django REST Framework serializers
- Leave workflow state machine (PENDING → APPROVED/REJECTED)
- Role-based views for Employee, Department Head, and HR Admin
- Frontend templates with Bootstrap styling reusing Fusion's base template
- Django admin configuration
- Initial database migration files

---

## Prompt 2 — Integration Fix

**Role:** User  

```
Register the ELM module in Django settings INSTALLED_APPS. Add URL routes to
config/urls.py. Make sure migrations connect to Fusion's existing DB config.
Also ensure the permission codes follow the pattern used in
modules/accesscontrol/management/commands/permission_manifest.py.
```

**Model response:** Updated `INSTALLED_APPS`, wired URL routes, aligned
permission codes with Fusion's `accesscontrol` manifest format.

---

## Prompt 3 — Workflow / State Machine Fix

**Role:** User  

```
The leave approval workflow should follow these states:
PENDING → FORWARDED (by HOD) → APPROVED/REJECTED (by Academic Admin or HR)
Add an intermediate forwarding step where the HOD forwards to the next authority
before final approval. Reuse Fusion's existing role system for authority checks.
```

**Model response:** Added `FORWARDED` state to `LeaveRequest` model, added
`forward_leave` API endpoint, updated frontend to show forwarding action for HOD role.

---

## Key Differences vs. stable-code:3b

| Criterion | stable-code:3b | deepseek-coder-v2 |
|-----------|---------------|-------------------|
| Context understanding | Partial | Better |
| Multi-file coordination | Basic | Good |
| Workflow completeness | Basic | More complete |
| Spec adherence | Partial | Better |
| Generation speed | Fast | Slower (~3–4 min/response) |
| Model size / RAM usage | Small (low RAM) | Large (8.9 GB, heavy on M5) |

---

## Integration Changes Made After Generation

| Change | Details |
|--------|---------|
| File placement | Moved generated files into correct `modules/elm/` folder |
| Import paths | Fixed relative imports to match Fusion module layout |
| Module registration | Added `modules.elm` to `INSTALLED_APPS` in `config/settings.py` |
| Route registration | Added `path('elm/', include('modules.elm.urls'))` to `config/urls.py` |
| DB wiring | Connected to Fusion's default DB config |
| Permission seeding | Aligned permission codes with `accesscontrol` seed format |
| Template paths | Fixed template loader paths to match Fusion's template dirs |

---

## Build / Run Status

**Result:** Partly builds/runs; some issues remain.

**Issues encountered:**
- DRF serializer import error (missing `rest_framework` in INSTALLED_APPS)
- Template `{% extends "base.html" %}` path mismatch
- `LeavePolicy` admin required additional inline registration

---

## Overall Impression of deepseek-coder-v2

| Criterion | Result |
|-----------|--------|
| Loaded on laptop | Yes (8.9 GB, slow to load initially) |
| Usable speed | Marginal (3–5 min per response on M5 16 GB) |
| Understood project structure | Yes |
| Multi-file changes | Yes |
| Reused Fusion infrastructure | Yes (better than stable-code:3b) |
| Frontend implementation | Yes |
| Backend implementation | Yes |
| Database integration | Yes |
| Build/run result | Partly builds/runs |
| Overall | Usable — best option of the two tested |
