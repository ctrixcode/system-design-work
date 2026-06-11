# GoTruks — Logistics & Fleet Management Platform

## Overview
GoTruks is a microservices-based logistics platform designed and built for the Saudi Arabian client UBLCO (ublco.net). The platform successfully digitized the client's completely offline fleet operations into a modern, full-stack SaaS solution. Featuring over 80 operational modules, the platform was built over an intensive 4.5-month period. I architected and led the development of the entire platform from inception to production deployment.

## Architecture

```mermaid
graph TD
    Client["Client / Web UI / Mobile App"]

    subgraph "API Gateway Layer"
        Gateway["API Gateway (Port 5000)<br>• Dynamic Routing<br>• Helmet & CORS<br>• Rate Limiting"]
    end

    subgraph "Core Microservices"
        AuthService["Auth Services (Port 5001)"]
        EssentialModules["Essential Modules (Port 5000 / gRPC 50051)"]
    end

    subgraph "Database Layer (PostgreSQL)"
        GatewayDB[(Gateway DB)]
        AuthDB[(Auth DB)]
        EssentialDB[(Essential Modules DB)]
    end

    subgraph "Asynchronous Workers & Queues"
        SQS["AWS SQS / SNS"]
        NotificationWorker["Notification Worker"]
        PricingWorker["Pricing Expiry Worker (node-cron)"]
        ReportsWorker["Scheduled Reports Worker (node-cron)"]
    end

    %% Client communication
    Client -->|HTTP/REST| Gateway
    
    %% Gateway routing
    Gateway -->|DB Read/Write| GatewayDB
    Gateway -.->|Proxy| AuthService
    Gateway -.->|Proxy| EssentialModules

    %% Inter-service communication
    AuthService <==>|gRPC<br>(user.proto, vehicle.proto, notifications.proto)| EssentialModules

    %% Database connections
    AuthService --> AuthDB
    EssentialModules --> EssentialDB

    %% Async workflows
    EssentialModules -->|Publish Events| SQS
    SQS --> NotificationWorker
    SQS --> PricingWorker
    SQS --> ReportsWorker
```

## Core Domains
The platform encompasses 80+ operational modules grouped into the following primary domains:

- **Bookings & Fleet:** Allocation engines, address resolver, real-time trip tracking, and geofencing.
- **Billing & Ledger:** Automated multi-currency invoices, driver ledgers, settlements, and tax/toll calculators.
- **Pricing Engine:** Complex vendor and client pricing slabs with automated fare calculations.
- **Credit Management:** B2B wallets, dynamic credit limits, and balance transaction processing.
- **Notifications:** Configurable notification templates, user preference matrices, and asynchronous delivery queues.

## Tech Stack

| Technology | Purpose | Why |
| :--- | :--- | :--- |
| **Node.js v18+ & Express v5** | Core Backend Framework | High concurrency support, robust ecosystem, and native asynchronous non-blocking I/O ideal for logistics. |
| **PostgreSQL & Sequelize** | Relational DB & ORM | Reliable ACID transactions, strict schema enforcement, and excellent spatial/JSONB capabilities. |
| **gRPC** | Inter-Service RPC | Protocol buffers provide strongly typed schemas, low latency, and lightweight network payloads compared to REST. |
| **AWS SQS / SNS** | Message Broker & Pub/Sub | Decouples time-intensive processing from API requests, ensuring scalability and fault tolerance. |
| **node-cron** | Task Scheduling | Reliable chron job execution for background expiry evaluations and scheduled reporting workflows. |
| **jsonwebtoken / bcrypt** | Auth & Security | Industry standard stateless authentication and secure, salted password hashing. |
| **firebase-admin** | Push Notifications | Unified API for reliable push notification delivery across mobile and web clients. |
| **exceljs / pdfkit** | Reporting Generation | Performant generation of complex financial statements and bulk operational reports natively in Node. |
| **Pino** | Application Logging | Extremely low overhead, high-performance structured JSON logger suitable for production metrics. |
| **Joi** | Object Schema Validation | Robust runtime validation for incoming API payloads ensuring data integrity before processing. |

## Key Design Decisions

- **gRPC over HTTP for Inter-Service Communication:** Transitioned internal chatter from REST to gRPC, achieving sub-millisecond RPC calls and enforcing strict typing via `.proto` definitions.
- **API Gateway Bypasses Body Parsing on Proxied Routes:** Engineered the gateway to stream raw HTTP requests directly to microservices without parsing the payload into memory, eliminating severe memory spikes and V8 Garbage Collection overhead.
- **Monolithic Modularity:** Structured the system around tightly cohesive, granular modules. This allows for rapid monolithic development now while ensuring a seamless extraction path into completely separate microservices as domain boundaries mature.
- **Background Workers for CPU-Intensive Tasks:** Offloaded PDF report generation, Excel exports, and batched notification processing to SQS and Node-cron workers, maintaining high responsiveness on the main API threads.
- **Full Audit Trails on Financials:** Implemented immutable audit logging on all state changes related to ledgers, credit limits, and invoicing to guarantee absolute compliance and traceability.

## Performance Results

- **80+** active operational modules deployed.
- **Sub-millisecond** latency achieved on internal gRPC communications.
- **4.5 Months** turnaround time from digitizing an offline business to launching a fully functional SaaS.
- Engineered to effortlessly handle **multi-currency** complex invoicing alongside **real-time GPS tracking**.

## My Role
Architected and led the development of the entire platform. Designed the API Gateway, inter-service gRPC communication strategies, core logistics engine, and the comprehensive administrative dashboard. Entirely owned database schema design, system service boundaries, and the cloud deployment strategy.

## What I'd Improve Next

- **Distributed Tracing:** Integrate OpenTelemetry and Jaeger to visualize and monitor request flows across the microservices map.
- **Testing Infrastructure:** Implement a robust Jest test suite to secure business logic and prevent regression.
- **Container Orchestration:** Add a comprehensive `docker-compose` setup to simplify local development environments.
- **Kubernetes Migration:** Migrate current deployments to a K8s cluster incorporating dynamic service discovery to improve auto-scaling capabilities.
