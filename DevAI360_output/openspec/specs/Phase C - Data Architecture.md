# Phase C: Data Architecture (DA)

## 1. Data Architecture Overview

### Data Principles
- **Single Source of Truth:** The PostgreSQL database is the absolute source of truth for all application data.
- **Data Integrity:** Data integrity is enforced at the database level using constraints (NOT NULL, FOREIGN KEY, CHECK) as the final guarantee.
- **Immutable Audit Trail:** All business-critical operations, especially state changes and administrative actions, **MUST** be recorded in an immutable audit log.
- **Soft Delete:** Business-critical entities like Delivery Orders should be soft-deleted to allow for recovery and maintain historical data integrity.

### Technology Stack
- **Database:** PostgreSQL 15+
- **ORM:** Spring Data JPA (Hibernate)
- **Migrations:** Flyway for version-controlled, automated schema migrations.
- **Caching:** Redis (Optional for V1, can be added later for performance optimization of frequently read, semi-static data).

---

## 2. Logical Data Model

### 2.1 Core Entities

| Entity | Description |
|---|---|
| **User** | Represents a user of the system, can be a Production Line User, Warehouse User, or Admin. |
| **DeliveryOrder** | Represents a single material supply request and its entire lifecycle. |
| **AuditLog** | An immutable record of every significant action performed on a DeliveryOrder. |

### 2.2 Entity Relationships

```
User (1) ----< (N) DeliveryOrder (as Requestor)
User (1) ----< (N) DeliveryOrder (as Processor)
DeliveryOrder (1) ----< (N) AuditLog
```

### 2.3 Entity Attributes

**Entity: User**
| Attribute | Type | Nullable | Description |
|---|---|---|---|
| id | UUID | No | Primary key |
| email | String | No | Unique login email |
| password_hash | String | No | Hashed password (bcrypt) |
| role | String | No | User role (`ROLE_PRODUCTION`, `ROLE_WAREHOUSE`, `ROLE_ADMIN`) |
| created_at | TIMESTAMP | No | Creation timestamp |
| updated_at | TIMESTAMP | No | Last update timestamp |

**Entity: DeliveryOrder**
| Attribute | Type | Nullable | Description |
|---|---|---|---|
| id | UUID | No | Primary key |
| material_name | String | No | Name of the requested material |
| quantity | Integer | No | Amount of material requested |
| delivery_location | String | No | Where the material needs to be delivered |
| status | String | No | Current state of the order (`NEW`, `IN_PREPARATION`, etc.) |
| requestor_id | UUID | No | Foreign key to the User who created the order |
| processor_id | UUID | Yes | Foreign key to the User who is processing the order |
| is_deleted | Boolean | No | Soft delete flag, default `false` |
| created_at | TIMESTAMP | No | Creation timestamp |
| updated_at | TIMESTAMP | No | Last update timestamp |

**Entity: AuditLog**
| Attribute | Type | Nullable | Description |
|---|---|---|---|
| id | UUID | No | Primary key |
| order_id | UUID | No | Foreign key to the DeliveryOrder being audited |
| user_id | UUID | No | Foreign key to the User who performed the action |
| action | String | No | Action performed (e.g., `CREATE`, `STATUS_CHANGE`, `DELETE`) |
| reason | String | Yes | Justification for admin edits/deletions |
| details | JSONB | Yes | JSON object containing details of the change |
| timestamp | TIMESTAMP | No | Timestamp of the action |

---

## 3. Physical Data Model

### 3.1 Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Table | snake_case, plural | `users`, `delivery_orders` |
| Column | snake_case | `created_at`, `order_status` |
| Primary Key | `id` | `id` |
| Foreign Key | `{table_name_singular}_id` | `user_id`, `order_id` |
| Index | `idx_{table}_{columns}` | `idx_delivery_orders_status` |
| Unique Constraint | `uq_{table}_{columns}` | `uq_users_email` |

### 3.2 Table Definitions

**Table: `users`**
| Column | Type | Constraints | Index |
|---|---|---|---|
| id | UUID | PK, NOT NULL | PK |
| email | VARCHAR(255) | NOT NULL | `uq_users_email` (UNIQUE) |
| password_hash | VARCHAR(255) | NOT NULL | - |
| role | VARCHAR(50) | NOT NULL | - |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | - |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | - |

**Table: `delivery_orders`**
| Column | Type | Constraints | Index |
|---|---|---|---|
| id | UUID | PK, NOT NULL | PK |
| material_name | VARCHAR(255) | NOT NULL | - |
| quantity | INT | NOT NULL, CHECK (quantity > 0) | - |
| delivery_location | VARCHAR(255) | NOT NULL | - |
| status | VARCHAR(50) | NOT NULL | `idx_delivery_orders_status` |
| requestor_id | UUID | NOT NULL, FK to `users(id)` | `idx_delivery_orders_requestor_id` |
| processor_id | UUID | NULL, FK to `users(id)` | `idx_delivery_orders_processor_id` |
| is_deleted | BOOLEAN | NOT NULL, DEFAULT false | - |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | - |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | - |

**Table: `audit_logs`**
| Column | Type | Constraints | Index |
|---|---|---|---|
| id | UUID | PK, NOT NULL | PK |
| order_id | UUID | NOT NULL, FK to `delivery_orders(id)` | `idx_audit_logs_order_id` |
| user_id | UUID | NOT NULL, FK to `users(id)` | - |
| action | VARCHAR(100) | NOT NULL | - |
| reason | TEXT | NULL | - |
| details | JSONB | NULL | - |
| timestamp | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | - |

---

## 4. Data Migration

### Migration Strategy
- **Tool:** Flyway
- **Location:** SQL migration scripts will be located in `src/main/resources/db/migration`.
- **Naming:** `V{YYYYMMDDHHMM}__{Description}.sql` (e.g., `V202404061000__create_users_table.sql`).
- **Rule:** Migrations are forward-only. Any corrections must be made in a new migration script. No modifications to committed migration files are allowed.

---

## 5. Data Security

### Sensitive Data
- **User Passwords:** **MUST** be hashed using a strong, salted algorithm like bcrypt. Raw passwords must never be stored.
- **PII:** User emails are considered sensitive and should be protected.

### Access Control
- The application will connect to the database via a dedicated service account with limited privileges (CRUD on its own tables).
- Direct database access by developers should be read-only in staging and production environments.
