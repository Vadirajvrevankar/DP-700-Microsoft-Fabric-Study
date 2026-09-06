# Microsoft Fabric Domains — DP-700 Best Notes

## 1. Domain

### CONCEPT
**Fabric Domain**

### WHAT IT DOES
A Domain groups Fabric **workspaces and data by business area**.

Examples: Sales, Finance, HR, Marketing.

Main purpose:
- Improve data discovery
- Organize workspaces
- Enable delegated governance

⚠️ **Domain does NOT provide access to data.**

### KEYWORDS
`Business area` · `Discovery` · `Governance` · `Workspace organization` · `Delegation`

### TYPICAL SCENARIO
A company wants users to easily find Finance-related workspaces and allow Finance teams to manage some governance settings.

**Use a Finance Domain.**

### ANSWER
> Create a domain to organize related workspaces by business area and enable delegated governance and discovery.

### MEMORY TRICK
**DOMAIN = ORGANIZE + GOVERN**

---

## 2. Domain ≠ Access

### CONCEPT
**Domain Assignment vs Access Control**

### WHAT IT DOES
Separates governance from security.

Domain assignment helps organize and discover data/workspaces. Actual access comes from:
- Workspace roles
- Item permissions

### KEYWORDS
`Domain ≠ Access` · `Workspace role` · `Item permissions` · `Security`

### TYPICAL SCENARIO
A user belongs to the Finance domain but cannot open a Finance workspace.

Why? Domain membership does **not** automatically give workspace access.

### ANSWER
> Domain assignment does not grant access. Use workspace roles and item permissions to control access.

### MEMORY TRICK
> **DOMAIN = WHERE IT BELONGS**  
> **PERMISSIONS = WHO CAN USE IT**

🔥 **DP-700 TRAP:** If the question says **"grant access"**, don't choose Domain.

---

## 3. Domain Roles

### CONCEPT
**Fabric Admin vs Domain Admin vs Domain Contributor**

### WHAT IT DOES
Defines who can perform domain-related operations.

| Role | Main responsibility |
|---|---|
| **Fabric Admin** | Creates/manages domains at tenant level |
| **Domain Admin** | Manages a specific domain |
| **Domain Contributor** | Assigns workspaces to a domain |

### KEYWORDS
`Fabric Admin` · `Domain Admin` · `Domain Contributor`

### TYPICAL SCENARIO
The central Fabric team creates the Sales domain. The Sales domain owner manages it. A workspace administrator needs to assign their workspace to Sales.

### ANSWER
> Fabric Admin creates the domain, Domain Admin manages the domain, and Domain Contributor handles workspace assignment within their permitted scope.

### MEMORY TRICK
**CREATE → MANAGE → ASSIGN**

**Fabric Admin → Domain Admin → Domain Contributor**

---

## 4. Domain Admin

### CONCEPT
**Domain Administrator**

### WHAT IT DOES
Manages a specific domain and its delegated governance responsibilities.

### KEYWORDS
`Manage domain` · `Governance` · `Domain settings` · `Data owner`

### TYPICAL SCENARIO
The Finance department has a knowledgeable data owner who understands classification policies and needs to manage appropriate domain-level governance.

### ANSWER
> Assign Domain Admin responsibility to a trusted person who understands the business data and governance requirements.

### MEMORY TRICK
**Domain Admin = MANAGES**

---

## 5. Domain Contributor

### CONCEPT
**Domain Contributor**

### WHAT IT DOES
Allows workspace administrators to **assign their workspaces to domains**.

It has a narrower scope than Domain Admin.

### KEYWORDS
`Workspace assignment` · `Workspace admin` · `Contributor`

### TYPICAL SCENARIO
A workspace administrator manages `Sales_Analytics` and needs to associate it with the Sales domain.

### ANSWER
> Use a Domain Contributor when the requirement is workspace assignment rather than full domain management.

### MEMORY TRICK
**Contributor = ASSIGN**

---

## 6. Assignment by Capacity

### CONCEPT
**Domain Assignment by Capacity**

### WHAT IT DOES
Assigns workspaces to domains based on their **Fabric capacity**.

### KEYWORDS
`Capacity` · `Department` · `Automatic assignment` · `Scale`

### TYPICAL SCENARIO
Sales capacity → Sales domain, Finance capacity → Finance domain, HR capacity → HR domain.

### ANSWER
> Use By Capacity when capacities already map clearly to business departments. It scales better than per-workspace assignment.

### MEMORY TRICK
**CAPACITY → DEPARTMENT**

🔥 If question says **"capacities map 1:1 to departments" → By Capacity**

---

## 7. Assignment by Name

### CONCEPT
**Domain Assignment by Name**

### WHAT IT DOES
Uses a predictable **workspace naming pattern** to assign workspaces to domains.

### KEYWORDS
`Workspace name` · `Naming convention` · `Pattern` · `Automatic assignment`

### TYPICAL SCENARIO
Workspaces are named `Sales_US`, `Sales_Europe`, and `Sales_Analytics`.

### ANSWER
> Use By Name when workspace names follow a predictable and consistent pattern.

### MEMORY TRICK
**NAME → NAMING PATTERN**

---

## 8. Assignment by Admin

### CONCEPT
**Domain Assignment by Admin**

### WHAT IT DOES
Allows appropriate workspace administrators to control domain assignment for the workspaces they administer.

### KEYWORDS
`Workspace Admin` · `Assignment` · `Delegation`

### TYPICAL SCENARIO
Workspace administrators know which business domain their workspace belongs to, but capacity or naming patterns are not reliable.

### ANSWER
> Use By Admin when workspace administrators should determine the appropriate domain for the workspaces they manage.

### MEMORY TRICK
**ADMIN → WORKSPACE OWNER**

---

## 9. Default Domain

### CONCEPT
**Default Domain**

### WHAT IT DOES
Provides a default domain for **future workspaces that aren't otherwise assigned**.

### KEYWORDS
`Default` · `Future workspaces` · `Automatic` · `Unassigned`

### TYPICAL SCENARIO
A team regularly creates new workspaces and wants them to fall under a particular domain when no other assignment rule applies.

### ANSWER
> Use a Default Domain to automatically handle future workspaces that don't otherwise receive a domain assignment.

### MEMORY TRICK
**DEFAULT = FUTURE FALLBACK**

⚠️ **DP-700 TRAP:** Default domain does not automatically override existing assignments.

---

## 10. Sensitivity Label Delegation

### CONCEPT
**Delegated Sensitivity Label Settings**

### WHAT IT DOES
Allows appropriate domain-level governance over **sensitivity-label settings**.

Sensitivity labels are related to **data classification and protection**.

### KEYWORDS
`Sensitivity label` · `Classification` · `Data owner` · `Governance`

### TYPICAL SCENARIO
Finance has a designated data owner who understands which financial information is confidential.

### ANSWER
> Delegate sensitivity-label settings only to domains with a responsible data owner who understands the organization's classification requirements.

### MEMORY TRICK
**SENSITIVITY LABEL = CLASSIFICATION**

---

## 11. Certification Delegation

### CONCEPT
**Delegated Certification Settings**

### WHAT IT DOES
Allows a domain to manage appropriate certification-related governance according to organizational policy.

Certification helps identify **trusted/approved data assets**.

### KEYWORDS
`Certification` · `Trusted data` · `Data owner` · `Governance`

### TYPICAL SCENARIO
The Sales data owner knows which semantic models or datasets are officially trusted by Sales.

### ANSWER
> Delegate certification settings only when the domain has appropriate ownership and governance knowledge.

### MEMORY TRICK
**CERTIFICATION = TRUST**

---

## 12. Override Workspace Assignment

### CONCEPT
**Allow tenant and domain admins to override workspace assignments**

### WHAT IT DOES
Allows authorized administrators to **override existing workspace-domain assignments**.

### KEYWORDS
`Override` · `Existing assignment` · `Tenant admin` · `Domain admin` · `Preview`

### TYPICAL SCENARIO
A tenant administrator needs to correct existing domain assignments across several workspaces.

### ANSWER
> Enable this setting deliberately when administrators need to change existing workspace-domain assignments.

### MEMORY TRICK
**OVERRIDE = CHANGE EXISTING**

⚠️ Incorrect bulk reassignment can affect teams using delegated domain governance.

---

## 13. Choosing the Assignment Method

### CONCEPT
**Domain Assignment Methods**

### WHAT IT DOES
Lets you choose the assignment approach that matches the organization's structure.

### KEYWORDS
`By capacity` · `By name` · `By admin` · `Default`

### TYPICAL SCENARIO

| Requirement | Choose |
|---|---|
| Capacities map 1:1 to departments | **By Capacity** |
| Predictable workspace names | **By Name** |
| Workspace admins should decide | **By Admin** |
| Future unassigned workspaces | **Default Domain** |

### ANSWER
> Choose the assignment method based on how your organization structures and creates workspaces.

### MEMORY TRICK
**CAPACITY → Department**  
**NAME → Pattern**  
**ADMIN → Owner**  
**DEFAULT → Future**

---

## 14. Domain Admin vs Domain Contributor

### CONCEPT
**Domain Admin vs Domain Contributor**

### WHAT IT DOES
Separates **domain management** from **workspace assignment responsibilities**.

### KEYWORDS
`Domain Admin` · `Domain Contributor` · `Management` · `Assignment`

### TYPICAL SCENARIO
You don't want everyone who can assign a workspace to also have full control over domain governance.

**Domain Admin** → manages domain  
**Domain Contributor** → assigns eligible workspaces

### ANSWER
> Domain admins have broader domain-management responsibilities, while domain contributors have a narrower workspace-assignment role.

### MEMORY TRICK
**Admin = Manage**  
**Contributor = Assign**

---

## 15. Top-Level Domains vs Subdomains

### CONCEPT
**Using Subdomains for Organizational Nuance**

### WHAT IT DOES
Provides detailed organizational structure without creating many independent top-level domains.

### KEYWORDS
`Subdomain` · `Region` · `Sub-team` · `Organizational hierarchy`

### TYPICAL SCENARIO
Instead of creating separate top-level domains for Sales North America, Sales Europe, and Sales Asia, use:

**Sales Domain**
→ North America subdomain  
→ Europe subdomain  
→ Asia subdomain

### ANSWER
> Prefer subdomains when differences are mainly organizational nuances such as regions or sub-teams.

### MEMORY TRICK
**One Department → Many Details = Subdomains**

---

## 16. Federated Governance

### CONCEPT
**Federated Governance with Domains**

### WHAT IT DOES
Allows **central governance + business-level governance**.

### KEYWORDS
`Federated governance` · `Data mesh` · `Domain ownership` · `Delegated governance`

### TYPICAL SCENARIO
The central Fabric team establishes company-wide standards while Finance, Sales, and HR have knowledgeable data owners who manage appropriate domain-level governance.

### ANSWER
> Domains support a federated governance model by allowing business areas to manage appropriate governance responsibilities while maintaining centralized tenant-level control.

### MEMORY TRICK
**CENTRAL CONTROL + LOCAL OWNERSHIP**

---

# DP-700 SCENARIO CHEAT SHEET

| If the question says... | Think... |
|---|---|
| Organize data by business area | **Domain** |
| Improve discovery | **Domain** |
| Delegate governance | **Domain** |
| Give access | **Workspace roles / Item permissions** |
| Capacity maps to department | **By Capacity** |
| Predictable workspace naming | **By Name** |
| Workspace admins should assign | **By Admin** |
| Future unassigned workspaces | **Default Domain** |
| Regions / sub-teams | **Subdomain** |
| Classification | **Sensitivity Label** |
| Trusted/approved data | **Certification** |
| Change existing assignment | **Override setting** |
| Create domains | **Fabric Admin** |
| Manage domain | **Domain Admin** |
| Assign workspaces | **Domain Contributor** |

# FINAL MASTER MEMORY TRICK

> **DOMAIN = ORGANIZE**  
> **SUBDOMAIN = DETAIL**  
> **CAPACITY = DEPARTMENT**  
> **NAME = PATTERN**  
> **ADMIN = OWNER**  
> **DEFAULT = FUTURE**  
> **SENSITIVITY = CLASSIFY**  
> **CERTIFICATION = TRUST**  
> **OVERRIDE = CHANGE**  
> **DOMAIN ≠ ACCESS**

## ⭐ One Exam Sentence

> **“Fabric Domains organize and govern data by business area; they don't grant access.”**
