# ASSISTANTFY SHOWCASE

> **An AI-powered B2B SaaS backend designed to automate workflows and booking management for service-based businesses**

*⚠️ **Note:** Assistantfy is currently a proprietary, closed-source SaaS in active development. This repository serves as a technical portfolio to showcase the system's architecture, engineering decisions, and the complex challenges solved during its creation.*

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![DeepSeek API](https://img.shields.io/badge/AI-DeepSeek_API-4D4D4E?style=for-the-badge)

## 📌 The Product Vision
Assistantfy acts as an affordable, 24/7 virtual assistant for local businesses (clinics, salons, etc.). It eliminates manual call handling by leveraging Natural Language Processing (NLP) to answer FAQs and manage end-to-end appointment scheduling, allowing owners to focus on their core services.

---

## 🏗️ System Architecture & Evolution

The project began as a single-tenant MVP using SQLite but is evolving into a highly scalable, multi-tenant architecture backed by PostgreSQL. 

### The Multi-Tenant Database Schema
To handle multiple businesses securely within the same database, I designed a multi-tenant schema where a `Business` (Tenant) acts as the root entity, ensuring strict data isolation.

``` mermaid
erDiagram
    %% CORE TENANT & AUTHENTICATION
    BUSINESS {
        int id PK
        string name
        string timezone
    }

    ADMIN_USER {
        int id PK
        int business_id FK
        string email UK "Unique"
        string hashed_password
        boolean is_active
    }

    BUSINESS_HOURS {
        int id PK
        int business_id FK
        int day_of_week "1=Mon, 7=Sun"
        time start_time
        time end_time
    }

    %% RESOURCES & SKILLS
    WORKER {
        int id PK
        int business_id FK
        string name
    }

    SERVICE {
        int id PK
        int business_id FK
        string name 
        float price 
        int duration_minutes 
    }

    WORKER_SERVICE {
        int worker_id PK,FK "Points to Worker.id"
        int service_id PK,FK "Points to Service.id"
    }

    %% CUSTOMERS
    CLIENT {
        int id PK
        int business_id FK
        string phone_number UK "Unique per Business"
        string name
    }
    
    %% TRANSACTIONS 
    APPOINTMENT {
        int id PK
        int business_id FK
        int client_id FK
        int worker_id FK
        int service_id FK
        datetime start_time 
        datetime end_time
    }

    %% RELATIONSHIPS
    BUSINESS ||--o{ ADMIN_USER : "has accounts (login)"
    BUSINESS ||--o{ BUSINESS_HOURS : "operates during"
    BUSINESS ||--o{ WORKER : "employs"
    BUSINESS ||--o{ SERVICE : "offers"
    BUSINESS ||--o{ CLIENT : "registers"
    BUSINESS ||--o{ APPOINTMENT : "manages"
    
    %% MANY-TO-MANY RESOLUTION
    WORKER ||--o{ WORKER_SERVICE : "can perform"
    SERVICE ||--o{ WORKER_SERVICE : "is performed by"

    %% APPOINTMENT RELATIONSHIPS
    CLIENT ||--o{ APPOINTMENT : "books"
    WORKER ||--o{ APPOINTMENT : "attends"
    SERVICE ||--o{ APPOINTMENT : "includes"
```

## 🧠 Key Engineering Challenges Conquered
Building this project is possibly the best decision I have taken to learn, but it is also becoming a serious product. The following content is a summary of my private progress log: 

## 1. Asynchronous Webhook Processing & Resource Management
Integrating Meta's **WhatsApp Cloud API** required real-time webhook handling. Initially, I used synchronous requests, but to achieve high concurrency, I migrated to asynchronous calls using `httpx.AsyncClient`.

> **The Fix:** I tied the HTTP client and AI service instantiation directly to FastAPI's `@asynccontextmanager` lifespan. This ensures the connection pool is efficiently reused across requests rather than opening/closing connections per message, significantly reducing latency.

## 2. AI Tool Calling & Defensive Programming
The AI (DeepSeek LLM) acts as an autonomous agent that needs to query the database.

* **Defensive Design:** I implemented strict AI Tooling (Function Calling) to allow the LLM to search for available slots and create appointments. Instead of leaking standard HTTP status codes to the LLM upon failure, I apply defensive programming techniques to return natural language constraints back to the prompt, reducing AI hallucinations.
* **Prompt Security:** Core business logic is strictly encapsulated and prompts are secured against injection to protect the SaaS's intellectual property.

## 3. Scalable Dependency Injection
To maintain **Clean Architecture** principles, I heavily rely on FastAPI's Dependency Injection system. Database sessions are injected purely at the router level and passed down to the service layer. This prevents multiple unnecessary database hits during complex AI intent-recognition flows (e.g., verifying a user's phone number before passing context to the LLM).

## 4. Advanced Booking Logic (Skill-Based Routing)

Calculating "empty slots" isn't just about finding free time; it's a capacity problem. By utilizing a many-to-many junction table (WORKER_SERVICE), the algorithm dynamically calculates availability based on:
- Skill-matching: Filtering only the workers qualified to perform the requested service.
- Service metrics: The specific duration of the requested service.
- Concurrency: The operating hours of the business and the overlapping schedules of individual workers to prevent double-booking.


















