---
title: "SAR-2.3_IT_SDLC_Solution_Architecture‎_SAD_v1.0"
description: "Solution Architecture Document (SAD) – NCGR Template"
lang: ar
author: ""
---

<!-- Theme & Styling (CSS hints) -->
<style>
  /* ===== Theme: NCGR – Dark Blue & Teal accents ===== */
  :root{
    --primary:#0b3d91;      /* deep blue */
    --secondary:#00a3a3;    /* teal */
    --accent:#e6b800;       /* amber accent */
    --text:#1f2937;         /* slate-800 */
    --muted:#6b7280;        /* gray-500 */
    --bg:#ffffff;           /* white */
    --table-head-bg:#f3f4f6;/* gray-100 */
    --table-border:#e5e7eb; /* gray-200 */
  }
  html, body { font-family: "Segoe UI", Tahoma, Arial, "Noto Sans Arabic", "Geeza Pro", sans-serif; color: var(--text); background:var(--bg); }
  h1, h2, h3, h4 { color: var(--primary); }
  h1 { border-bottom: 3px solid var(--secondary); padding-bottom: 6px; }
  h2 { border-left: 6px solid var(--secondary); padding-left: 8px; }
  blockquote { border-right: 4px solid var(--secondary); border-left: 4px solid var(--secondary); padding: 0.6em 1em; color: var(--muted); background:#fafafa; }
  table { border-collapse: collapse; width: 100%; font-size: 0.98rem; }
  th, td { border: 1px solid var(--table-border); padding: 8px 10px; vertical-align: top; }
  thead th { background: var(--table-head-bg); }
  caption { caption-side: bottom; color: var(--muted); font-style: italic; padding-top: 4px; }
  .badge { display:inline-block; background:var(--secondary); color:#fff; padding:2px 8px; border-radius:12px; font-size:0.8rem; }
  .logo { width: 140px; height:auto; }
  .restr { color: #8b0000; font-weight:600; }
</style>

<!-- Cover / Header -->
# SAR-2.3_IT_SDLC_Solution_Architecture‎_SAD_v1.0

![Logo Description automatically generated with low confidence](logo-placeholder.png){.logo}

اسم المشروع – مخطط معمارية الحل  
اسم المشروع – مخطط معمارية الحل  
اسم المشروع – مخطط معمارية الحل  

**Solution Architecture SAD**  
**[Project Name]**  

**Enterprise Architecture**  
**وثيقة معمارية الحل**  
**Software Architecture (SAD)**  
**التاريخ: 6 – 6- 2024**

---

## Document Information

**Document number**  
**Document Name**  
**Solution Architecture SAD – [Project name]**  
**Department**  
Enterprise Architecture  
**Description**  
This document should act as a guideline for building Solution Architecture Design (SAD) to design and implement new project or make changes in existing projects as change requests as per defined standards.  
**Document purpose**  
The purpose of this document is to provide a clear overview of the system's architecture, design decisions, and standards to ensure compliance and facilitate communication among stakeholders

---

## Revision History

| **Version** | **Date** | **Editor** | **Description** |
|---|---|---|---|

## Approval History

| **Revision #** | **Name** | **Role** | **Approval Date** | **Signature** |
|---|---|---|---|---|

---

# Contents

- [Figures](#figures)
- [Tables](#tables)
- [1. Document Overview](#document-overview)
  - [1.1 Document Purpose](#document-purpose)
  - [1.2 Documentation Standards‎](#documentation-standards)
  - [1.3 Definitions, Acronyms, and Abbreviations](#definitions-acronyms-and-abbreviations)
  - [1.4 Key Stakeholders](#key-stakeholders)
- [2. Solution overview](#solution-overview)
  - [2.1 Functionalities](#functionalities)
  - [2.2 Vision](#vision)
  - [2.3 Quality Attributes Requirements](#quality-attributes-requirements)
  - [2.4 Out Of Scope](#out-of-scope)
  - [2.5 Affected Systems](#affected-systems)
  - [2.6 Architectural Standards](#architectural-standards)
- [3. Architecture‎ Views](#architecture-views)
  - [3.1 Business view](#business-view)
    - [Context Diagram](#context-diagram)
    - [System Behavior](#system-behavior)
    - [System Interfaces](#system-interfaces)
    - [Conceptual Diagram](#conceptual-diagram)
  - [3.2 Logical View](#logical-view)
    - [System Decomposition Diagram](#system-decomposition-diagram)
    - [System Dependency Structure Matrix (DMS)](#system-dependency-structure-matrix-dms)
    - [<Submodule> View Packet](#submodule-view-packet)
  - [3.3 Component View](#component-view)
    - [Component Diagram](#component-diagram)
    - [Connectors Diagram](#connectors-diagram)
  - [3.4 Deployment View](#deployment-view)
    - [Deployment Diagram](#deployment-diagram)
    - [Servers Requirements](#servers-requirements)
    - [Communication Matrix](#communication-matrix)
  - [3.5 Architecture Decision Matrix](#architecture-decision-matrix)
- [4. Capacity Planning](#capacity-planning)
- [5. Security & Infrastructure Exceptions (If any)](#security--infrastructure-exceptions-if-any)
- [6. Appendix](#appendix)
  - [6.1 Non-Functional Requirements](#non-functional-requirements)
  - [6.2 Document Checklist](#document-checklist)
- [Approval and Authorization](#approval-and-authorization)

---

## Figures

- Figure 1 Use Case Diagram  
- Figure 2 Context Diagram  
- Figure 3 Sequence Diagram  
- Figure 4 Activity Diagram  
- Figure 5 System Interfaces Diagram  
- Figure 6 Conceptual Diagram  
- Figure 7 High level Decomposition  
- Figure 8 Subsystem Context Diagram  
- Figure 9 Subsystem Sequence Diagram  
- Figure 10 Subsystem Activity Diagram  
- Figure 11 Subsystem Interface  
- Figure 12 Module Diagram  
- Figure 13 Component Diagram  
- Figure 14 Connector Diagram  
- Figure 15 Deployment Diagram

## Tables

- Table 1 Definitions, Acronyms, Abbreviations  
- Table 2 Stakeholders Views Matrix  
- Table 3 Concepts Definitions  
- Table 4 System Dependency Structure Matrix  
- Table 5 Subsystem Dependency Structure Matrix  
- Table 6 Communication Matrix

---

# Document Overview

## Document Purpose

The purpose of this Solution Architecture Document is to describe the role of <<system name>>, the system structure and relationships with the other systems. The system structure description will detail the solution elements, the externally-‎visible properties of those elements, and the relationships among them.

## Documentation Standards‎

- TOGAF 9
- Eclipse Software Process Framework (SPF)
- IEEE architecture of software-intensive systems [IEEE 1471-2000]

## Definitions, Acronyms, and Abbreviations

| ID | Abbreviations | Description |
|---:|---|---|
| 1 | SAD | Solution Architecture Document |
| 2 | View |  |
| 3 | Viewpoint |  |
| … | … |  |

*Table 1 Definitions, Acronyms, Abbreviations*

## Key Stakeholders

The purpose of this section is to show the different stakeholders of the document and the views that addresses each stakeholder’s concerns. <The values in below matrix must be updated to match your specific need>

> D=Detailed Information, S=Some Detail, O=Overview, N=Not Required

| Document Stakeholders | **System Context Diagram** | **System Behavior** | **System Interfaces** | **Conceptual Diagram** | **System Decomposition Diagram** | **Modules Diagram** | **Dependency Structure Matrix** | **Component Diagram** | **Connections Diagram** | **Deployment Diagram** | **Servers Requirements** | **Communication Matrix** |
|---|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| Solution Architects | D | D | D | D | D | D | D | D | D | D | S | D |
| Project Manager | D | S | S | D | D | O | N | N | N | O | O | D |
| Application Developers | D | D | D | D | D | D | D | D | D | S | N | D |
| Database Developers | O | O | O | D | O | O | O | O | O | O | S | O |
| Integrators | D | D | D | S | O | N | N | O | O | O | O | D |
| Testers | D | D | D | D | O | O | O | O | N | N | N | O |
| Infrastructure Team |  |  |  |  |  |  |  |  |  |  |  |  |
| Operation Team |  |  |  |  |  |  |  |  |  |  |  |  |
| Security Team |  |  |  |  |  |  |  |  |  |  |  |  |
| … |  |  |  |  |  |  |  |  |  |  |  |  |

*Table 2 Stakeholders Views Matrix*

---

# Solution overview

<The purpose of this section is to provide problem statement that captures the current issues, problems, or pains that affecting the business functions. It is also required to provide an overview of the solution that will used to address the problems identified in the problem statement mentioned earlier>

## Functionalities

<The purpose of functionalities is to provide a high-level description for the system functionalities enriched with use case diagram(s). You may also refer to the BRD for full functionality details>

- **Figure 1 Use Case Diagram**  
<Provide high level description for the use case diagram>

## Vision

<Try to capture the business long-term vision for the solution under discussion>

## Quality Attributes Requirements

<List of all the quality of attributes that will be taken into consideration in this architecture. For example:
- Availability:
  - system must be available 24x7
  - Failures must be detected and notified in 30 seconds
- Operation:
  - Agencies onboarding should not take more than 1 hour
- Performance:
  - The system must be able to handle x concurrent user sessions
  - And requests must be processed within x seconds
- Maintainability:
  - Solution must be able to support new data providers within 10 man days
  - Solution is expected to support the Mobile Application in the future with minor changes
- Security:
  - Changes on contracts values must be audited
  - Contract amount is a sensitive information 
Etc.>

## Out Of Scope

<Provide a list of all the functions that are not considered part of the current scope of the solution>

## Affected Systems

<What are the integrated systems and systems that need to be changed to support the solution under discussion>

| **Application / System Name** |
|---|
| <Application1 > |
| <Application 2> |

## Architectural Standards

The following is the list of standards followed through architecting the solution and must be also followed through the implementation:

- NCGR Security standards
- NCGR EA standards
- NCGR DevOps standards
- NCGR Reference Architecture
- NCGR API Standards
- NCGR API Maturity Model
- NCGR SAD Checklist

---

# Architecture‎ Views

## Business view

This view adheres to [Business Viewpoint](https://confluence.mof.gov.sa/display/SA/Business+Viewpoint)  
<The purpose of the business view is to provide a description about the <u>solution role</u>, and <u>to which value stream it belongs</u>, and its <u>main business capabilities</u>. Note: System is presented in this view diagrams as black box>

<Define the proposed solution business capabilities, provide the solution business capabilities diagram (if possible) where business capabilities are presented as black boxes>

### Context Diagram

<The purpose of this diagram is to show the system boundaries and what is in and what is out. In this diagram, the system must be presented as black box>

- **Figure 2 Context Diagram**  
<Provide a description for the system context and provide more details about the above diagram>

### System Behavior

<The purpose of this diagram is to show the business services for the system according to architect understanding for the business requirements. The business services must be visually elaborated using System Activity Diagram and/or System Sequence Diagram. System will be presented as black box without exposing internal system structure>

- **Figure 3 Sequence Diagram**  
- **Figure 4 Activity Diagram**  
<Provide a description about the business services and details about above diagrams>

### System Interfaces

<The purpose of this section is to show the system interfaces according to the services identified in the previous section. The provided interfaces define the systems under discussion responsibilities while the required interfaces present the external systems responsibilities>

- **Figure 5 System Interfaces Diagram**  
<Describe the provided interfaces and required interfaces shown in the diagram>

### Conceptual Diagram

<The purpose of this section is to identify the main business concepts, define them and identify the relationships among them as per the business domain, business stakeholders and subject matter experts>

- **Figure 6 Conceptual Diagram**

| Concept | Definition | Linked with |
|---|---|---|
| Concept1 | Provide a definition for this concept as per the business stakeholders | * Uses Concept3<br>* Provides Concept4<br>* Includes Concept2<br>* Supports Concept5 |
| Concept2 | .. | * Included in Concept1<br>* Parent of Concept6<br>* Provides Concept7 |
| … |  |  |
|  |  |  |

*Table 3 Concepts Definitions*

---

## Logical View

This view adheres to [Logical Viewpoint](https://confluence.mof.gov.sa/display/SA/Logical+Viewpoint)  
<Describe here the design approach followed and architectural style used while architecting the solution>

### System Decomposition Diagram

<The purpose of this section is to describe how the system is divided, according to the used design approach and the architectural style, into layers, sub-systems, bounded contexts, subdomains, etc.>

- **Figure 7 High level Decomposition**  
<Describe at a high-level the above diagram, describe how the architectural style influenced the above decomposition and explain how the subsystems are distributed into the above layers and define the layers and subsystems roles and the main responsibilities>

### System Dependency Structure Matrix (DMS)

<The purpose of this matrix is to show which dependency is allowed between the layers and subsystems and which is prohibited>

|  |  | _Caller_ |  |  |  |  |
|---|---|---|---|---|---|---|
|  |  | _Subsystem 1_ | _Subsystem 2_ | _Subsystem 3_ | _Subsystem 4_ | _Subsystem 5_ |
| _Callee_ | _Subsystem 1_ |  | Prohibited | Prohibited | Prohibited | Prohibited |
|  | _Subsystem 2_ | Allowed |  | Prohibited | Prohibited | Prohibited |
|  | _Subsystem 3_ | Prohibited | Allowed |  | Prohibited | Prohibited |
|  | _Subsystem 4_ | Prohibited | Prohibited | Allowed |  | Prohibited |
|  | _Subsystem 5_ | Prohibited | Prohibited | Prohibited | Allowed |  |

*Table 4 System Dependency Structure Matrix*

### <Submodule> View Packet

<For each layer, subsystem, and module identified in the previous section, a view packet must be provided and detailed here>

#### Subsystem Context Diagram

<The purpose of this diagram is to show the subsystem boundaries and what is in and what is out. In this diagram, the subsystem must be presented as black box>

- **Figure 8 Subsystem Context Diagram**  
<Provide a description for the subsystem context and provide more details about the above diagram>

#### Subsystem Behavior

<The purpose of this diagram is to show the system behavior and responsibilities according to architect understanding for the business requirements. The system behavior must be visually elaborated using System Activity Diagram and/or System Sequence Diagram. System will be presented as black box without exposing internal system structure>

- **Figure 9 Subsystem Sequence Diagram**  
- **Figure 10 Subsystem Activity Diagram**  
<Provide a description about the subsystem behavior and details about above diagrams>

#### Subsystem Interfaces

<The purpose of this section is to show the subsystem interfaces according to the subsystem behavior and responsibilities identified in the previous section>

- **Figure 11 Subsystem Interface**  
<Describe the provided interfaces and required interfaces shown in the diagram>

#### Subsystem Structure Diagram

<The purpose of this section is to show the subsystems internal structures by listing its modules and the relationships between the subsystem modules>

- **Figure 12 Module Diagram**  
<Describe at a high-level the above diagram, describe how the architectural style influenced the above decomposition and explain the modules roles and the main responsibilities>

#### Subsystem Dependency Structure Matrix (DMS)

<The purpose of this matrix is to show which dependency is allowed between the above modules and which is prohibited>

|  |  | _Caller_ |  |  |  |  |  |
|---|---|---|---|---|---|---|---|
|  |  | Module 1 | Module 2 | Module 3 | Module 4 | Module 5 | Module 6 |
| _Callee_ | Module 1 |  |  |  |  |  |  |
|  | Module 2 |  |  |  |  |  |  |
|  | Module 3 |  |  |  |  |  |  |
|  | Module 4 |  |  |  |  |  |  |
|  | Module 5 |  |  |  |  |  |  |
|  | Module 6 |  |  |  |  |  |  |

*Table 5 Subsystem Dependency Structure Matrix*

---

## Component View

This view adheres to [Component Viewpoint](https://confluence.mof.gov.sa/display/SA/Component+Viewpoint)  
<<The purpose of this view is to show how the different layers, subsystems, modules are physically packaged according to the architectural style. Components are independently deployable by residing inside an execution environment (such as web servers, application servers, containers, etc.; for example: Tiers, Microservices, etc.>>

### Component Diagram

<The purpose of this diagram is to show the list of components and for each component, it shows the layers, subsystems and modules packaged in that component>

- **Figure 13 Component Diagram**  
<Describe the different components shown in the above diagram and the purpose of each component and rationale behind such packaging>

### Connectors Diagram

<The purpose of this diagram is to show how the different components, listed in the previous section, are connected with each other. This diagram also shows the relationship between the components and external systems>

- **Figure 14 Connector Diagram**

---

## Deployment View

This view adheres to [Deployment Viewpoint](https://confluence.mof.gov.sa/display/SA/Deployment+Viewpoint)  
<The purpose of this diagram is to show the deployment topology for the solution and the communication matrix between the servers>

### Deployment Diagram

<The purpose of this diagram is to show the deployment topology that allocates the components identified in the previous view with the servers. Any server with a capacity that meets the requirements can be used>  
Please specify any specific design considerations such as (Assumptions, Constrains, Risks and Strategies)  
Please use attached NCGR (Visio Master Template) for design purposes. (Visio Master Template) with modeling reference for design purposes.

- **Figure 15 Deployment Diagram**

### Servers Requirements

<<Servers Requirements for production environment.>>

| **Server #** | **Environment** | **Role** | **OS** | **OS Ver/Edition** | **App\\DB technology & Version Details** |
|---|---|---|---|---|---|
| Server 1 | Production | Web | Windows  | 2016 / Standard |  |
| Server 2 | Production | Web | Windows  | 2016 / Datacenter |  |
| Server 3 | Production | DB Node 1 | RHEL | 7.6 |  |
| Server 4 | Production | DB Node 2 | RHEL | 7.6 |  |
| … |  |  |  |  |  |

<Other environments are required as per NCGR DevOps standards>  
<High Availability requirements>  
<Disaster Recovery requirements>  
<If the application requirement a new domain name, mention it here>  
*Note: The above shows only the production requirements. For other environments, DevOps standards must be followed*

### Communication Matrix

The following table shows the communication requirements between the servers, protocols used and the most critical services  
*Note: The exact services and services names will be detailed at the development stage*

| **Source** | **Destination** | **Main Services** | **Protocol** |
|---|---|---|---|
|  |  | * \n* \n*  |  |
|  |  | * \n* \n* |  |
|  |  | * \n* \n* |  |

*Table 6 Communication Matrix*

---

## Architecture Decision Matrix

<Provide separate table for each decision record>

| Title |  |
|---|---|
| Status | Proposed, Accepted |
| Context |  |
| Alternatives |  |
| Decision |  |
| Consequences |  |
| Compliance |  |
| Notes |  |

---

# Capacity Planning

Please check [Quality Attributes Requirements](#quality-attributes-requirements) section for more details

# Security & Infrastructure Exceptions (If any)

<any exception for NCGR standards must be explicity mentioned here with a justification>

# Appendix

## Non-Functional Requirements

## Document Checklist

| Section | Compliance | Yes, No, Partial | Justification (If Not) |
|---|---|---|---|
| **Document Overview** | Did you provide Document Overview Section? |  |  |
|  | Did you explain the purpose of this document? |  |  |
|  | Did you list the standards used in the documentation? |  |  |
|  | Did you provide a full list of definitions for the document terms, acronyms, and abbreviations? |  |  |
|  | Did you identify the different stakeholders and the identified the views that addresses each stakeholder concerns? |  |  |
| Solution Overview | Did you provide a comprehensive solution overview? |  |  |
|  | Did you explain the main solution functions? |  |  |
|  | Did you provide detailed list of the quality attributes? |  |  |
|  | Did you highlight what is exactly out of the scope? |  |  |
|  | Did you list the affected systems? |  |  |
|  | Did you list the standards followed in the architecture? |  |  |
| Business View | Did you provide Business View? |  |  |
|  | Did you explain the solution context? |  |  |
|  | Did you identify the system behaviour? |  |  |
|  | Did you list the system interfaces? |  |  |
|  | Did you provide the conceptual diagram and concepts definitions? |  |  |
| Logical View | Did you provide Logical View? |  |  |
|  | Did you show the system decomposition? |  |  |
|  | Did you show the layers and subsystems details? |  |  |
|  | Did you explain the subsystems structures? |  |  |
|  | Did you explain the subsystems behaviours and interfaces? |  |  |
| Component View | Did you provide Component View? |  |  |
|  | Did you list the components and explain the components contents? |  |  |
|  | Did you show the connections between the components? |  |  |
| Deployment View | Did you provide deployment view? |  |  |
|  | Did you explain the deployment topology? |  |  |
|  | Did you list the servers’ requirements? |  |  |
|  | Did you list the communication matrix between the servers? |  |  |

---

# Approval and Authorization

**الموافقة والاعتماد**

| **1** | **معماري الهيكلية المؤسسية** | **2** | **مدير إدارة الهيكلية المؤسسية** |
|---|---|---|---|
| الاسم:  
التاريخ:  
التوقيع: |  | الاسم:  
التاريخ:  
التوقيع: |  |

<p class="restr">Restricted - Internal Use Only</p>
