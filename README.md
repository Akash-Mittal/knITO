# knITO

*A meta-product creator on a modular Knitwear Platform – A creator tool for defining internal product lines.*

---

## Application Overview

`knITO` enables modular design, prototyping, review, and rendering of knitwear product metadata for internal and customer-facing processes.

```mermaid
---
config:
  theme: neo-dark
  look: neo
  layout: elk
---
flowchart TB
 subgraph PROXY["Proxy_service"]
        NGINX["nginx_loadbalancer"]
  end
 subgraph UI_Services["ui_Services"]
        UI["ui"]
        META_UI["meta-creater-ui - Internal Design Tool"]
        KNITO_UI["knito-ui - for customers"]
  end
 subgraph Backend["backend"]
        B["Backend"]
        META_DATA["meta-data-service"]
        PRODUCT["product-service"]
        RENDER["render-service - Blender + WASM"]
        WORKFLOW["workflow-service"]
        KNITTING["knitting-service - Generates Knitting Programs"]
        AUTH["auth-service - SSO + Roles/Policies"]
        NOTIFY["notification-service - Slack/Email/Messages"]
  end
 subgraph Data_Base["Database Layer"]
        DB["DB"]
        MONGO(("MongoDB"))
        MYSQL[("MySQL")]
  end
   NGINX --> UI --> B --> DB
```


---

## 🧩 Services

### 🖥️ UI Services
- **meta-creater-ui**  
  UI for creating products and categories. Designed for internal design teams to manage product metadata.

- **knito-ui**  
  A modern JavaScript-based UI for designing versatile knitwear products using the system's metadata. Built for external customers and brand partners.

---

### 🧠 Business Backend Services
- **meta-data-service**  
  A REST-based service that defines system-wide product metadata. Loads definitions into cache for high-speed access by other backend services.

- **product-service**  
  Handles actual product definitions, tightly integrated with `meta-creater-ui`. Used internally to manage product versions and variations.

- **render-service**  
  Flagship service built on Blender and WebAssembly. Responsible for generating 3D product previews and visual prototypes.

- **workflow-service**  
  Orchestrates business flows between internal and external actors. Enables user collaboration, defines responsibilities, and handles approval workflows.
  
  
> Manages the internal product lifecycle stages:
- DRAFT
- SOFT_PROTOTYPE (Image/Inventory Analysis)
- HARD_PROTOTYPE (Test Knitting)
- REVIEW (Preview, Internal Approval, Customer Demo)
- LIVE

## Versioned Workflow Examples

| Version | Workflow Path                                                                                      |
|---------|----------------------------------------------------------------------------------------------------|
| V1.0    | `DRAFT(V1.0)` → `PROTOTYPE(V1.0)` → `REVIEW(V1.0)` → `LIVE(V1.0)`                                  |
| V1.1    | `DRAFT(V1.0)` → `PROTOTYPE(V1.0)` → `DRAFT(V1.1)` → `REVIEW(V1.1)` → `LIVE(V1.1)`                  |
| V1.2    | `DRAFT(V1.0)` → `PROTOTYPE(V1.0)` → `DRAFT(V1.1)` → `REVIEW(V1.1)` → `DRAFT(V1.2)` → `REVIEW(V1.2)` → `LIVE(V1.2)` |

```mermaid

---
config:
  theme: neo-dark
  look: handDrawn
  layout: fixed
---
flowchart LR
    DRAFT_V1_0["DRAFT_V1_0"] --> PROTOTYPE_V1_0["PROTOTYPE_V1_0"] & PROTOTYPE_V1_0 & PROTOTYPE_V1_0
    PROTOTYPE_V1_0 --> REVIEW_V1_0["REVIEW_V1_0"] & DRAFT__V1_1["DRAFT__V1_1"] & DRAFT__V1_1
    REVIEW_V1_0 --> LIVE_V1_0["LIVE_V1_0"]
    DRAFT__V1_1 --> REVIEW_V1_1["REVIEW_V1_1"] & REVIEW_V1_1
    REVIEW_V1_1 --> LIVE_V1_1["LIVE_V1_1"] & DRAFT_V1_2["DRAFT_V1_2"]
    DRAFT_V1_2 --> REVIEW_V1_2["REVIEW_V1_2"]
    REVIEW_V1_2 --> LIVE_V1_2["LIVE_V1_2"]


```

---

### 🛠️ Technical Backend Services
- **auth-service**  
  Provides authentication (login, SSO) and defines user roles and policies for secure access control.

- **notification-service**  
  Manages notifications across the platform via Email, Slack, or in-app messages. Commonly used by `workflow-service` to update business stakeholders.




- **knitting-service**  
  Generates machine-readable knitting programs from designs created during the `prototype` and `review` stages. Outputs tailored instructions for knitting machines.

---

### 🛠️ Technical Backend Services
- **auth-service**  
  Handles authentication (login, SSO) and defines user roles and policies for secure access control.

- **notification-service**  
  Messaging queue–based notification system used by `workflow-service` to notify business stakeholders via Email, Slack, or in-app messages. Built on a loosely coupled Pub/Sub model.

- **proxy-service**  
  Acts as a gateway (e.g., NGINX or AWS API Gateway). Forwards incoming requests to `auth-service` to validate tokens and fetch authorized roles.

---

### 🗄️ Database Layer
- **MongoDB**  
  Used to store digital assets, metadata documents, and render-specific configurations.

- **MySQL / PostgreSQL**  
  Used for structured application data like users, products, workflows, and audit trails.

## Sample Request/Response 



**Request**
```json
### `product-service``
{
  "product_id": "KW-001",
  "name": "Cable Knit Sweater",
  "category": "Sweater",
  "material": "Wool Blend",
  "gauge": "7 GG",
  "fit": "Regular",
  "color": "Charcoal Grey",
  "graphics"{
  "2d"[],
  "3d"[]
  }
  
}

{
  "category": "Sweater",
  "allowed_gauges": ["5 GG", "7 GG", "12 GG"],
  "materials": ["Cotton", "Wool", "Acrylic"],
  "assets": "/data/path/img1.svg"
}

{
  "colors": ["Charcoal Grey", "Navy Blue", "Olive Green", "Black", "Beige"]
}

### `orchestration-Service``

{
  "factories": [
    {
      "name": "Ludo Gmbh",
      "location": "Berlin",
      "lead_time_days": 14,
      "status": "Available"
    }
  ]
}

### `Notification-Service`

{
  "event": "metadata_created",
  "product_id": "KW-001",
  "recipients": ["factory-manager@knitwear.com"],
  "message": "New metadata for Cable Knit Sweater has been created"
}
```


