# Wexel Workflow Documentation

> **Last Updated:** November 2025  
> **Version:** 1.0.0

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [System Architecture](#system-architecture)
4. [Database Schema](#database-schema)
5. [Authentication & Authorization](#authentication--authorization)
6. [Core Business Flows](#core-business-flows)
7. [Module Structure](#module-structure)
8. [Frontend Architecture](#frontend-architecture)
9. [API Documentation](#api-documentation)
10. [Deployment](#deployment)
11. [Getting Started](#getting-started)

---

## Project Overview

**Wexel** is a comprehensive telehealth platform that connects **patients** with **physiotherapists** for virtual consultations and treatments. The platform manages the complete lifecycle of medical appointments, from scheduling to video consultations.

### Key Features

- **Multi-role System**: Patients, Physiotherapists (Doctors), and Admins
- **Appointment Management**: Schedule, track, and manage appointments
- **Video Consultations**: Integrated Zoom SDK for video calls
- **Payment Processing**: Stripe integration with package and coupon system
- **Medical Screening**: Dynamic questionnaire system for patient assessment
- **Credit System**: Package-based credit allocation for appointments
- **Multi-language Support**: English and German (internationalization ready)
- **Availability Management**: Physio weekly and date-specific scheduling

---

## Technology Stack

### Backend (NestJS)

```
Core Framework
├─ NestJS 11.x          # Progressive Node.js framework
├─ Fastify              # High-performance web server
├─ TypeScript 5.x       # Type-safe development
└─ Node.js 18+          # Runtime environment

Database & ORM
├─ PostgreSQL           # Primary database
├─ Prisma 6.x           # Type-safe ORM
└─ Prisma Migrations    # Schema versioning

Authentication & Security
├─ Passport JWT         # JWT strategy
├─ JWKS (jwks-rsa)      # Key verification
├─ Keycloak             # Identity provider (3 realms)
├─ Helmet               # Security headers
└─ CSRF Protection      # Cross-site request forgery

Payment & Integration
├─ Stripe 17.x          # Payment processing
├─ Zoom Video SDK       # Video consultations
└─ Nodemailer           # Email notifications

Utilities
├─ AutoMapper           # Object-to-object mapping
├─ Class Validator      # DTO validation
├─ Class Transformer    # Data transformation
├─ date-fns            # Date manipulation
└─ Winston              # Logging
```

### Frontend (Next.js + Nx Monorepo)

```
Core Framework
├─ Next.js 15.x         # React framework
├─ React 19.x           # UI library
├─ TypeScript 5.x       # Type safety
└─ Nx Workspace         # Monorepo tooling

UI & Styling
├─ Tailwind CSS 3.x     # Utility-first CSS
├─ Radix UI             # Headless component primitives
├─ Lucide React         # Icon library
├─ Framer Motion        # Animation library
└─ shadcn/ui patterns   # Component patterns

State Management
├─ Zustand              # Lightweight state management
├─ TanStack Query 5.x   # Server state management
└─ React Hook Form      # Form state management

Authentication
├─ NextAuth 5.0 beta    # Authentication for Next.js
└─ Keycloak Integration # SSO provider

Utilities
├─ Axios                # HTTP client
├─ Zod                  # Schema validation
├─ date-fns             # Date utilities
├─ next-intl            # Internationalization
└─ React Hot Toast      # Notifications
```

### DevOps & Infrastructure

```
Cloud & Deployment
├─ Azure Pipelines      # CI/CD
├─ Kubernetes           # Container orchestration
├─ Docker               # Containerization
└─ Azure Container Registry

Package Management
└─ pnpm                 # Fast, disk space efficient
```

---

## System Architecture

### High-Level Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        PP[Patient Portal<br/>Next.js]
        DP[Doctor Portal<br/>Next.js]
    end
    
    subgraph "API Gateway"
        NEST[NestJS Backend<br/>Fastify]
    end
    
    subgraph "Authentication"
        KC_P[Keycloak Patient Realm]
        KC_D[Keycloak Physio Realm]
        KC_A[Keycloak Admin Realm]
    end
    
    subgraph "Database"
        PG[(PostgreSQL<br/>Prisma ORM)]
    end
    
    subgraph "External Services"
        STRIPE[Stripe<br/>Payments]
        ZOOM[Zoom Video SDK<br/>Consultations]
        EMAIL[Email Service<br/>Nodemailer]
    end
    
    PP -->|HTTPS/REST| NEST
    DP -->|HTTPS/REST| NEST
    
    NEST -->|JWT Validation| KC_P
    NEST -->|JWT Validation| KC_D
    NEST -->|JWT Validation| KC_A
    
    NEST -->|Prisma Client| PG
    
    NEST -->|Payment Intent| STRIPE
    STRIPE -->|Webhook| NEST
    
    NEST -->|Generate Token| ZOOM
    NEST -->|Send Email| EMAIL
    
    style PP fill:#3b82f6
    style DP fill:#8b5cf6
    style NEST fill:#10b981
    style PG fill:#f59e0b
```

### Application Layer Architecture

```mermaid
graph LR
    subgraph "Frontend Apps"
        A[Patient Portal]
        B[Doctor Portal]
    end
    
    subgraph "Shared Libraries"
        C[API Client]
        D[UI Components]
        E[Hooks]
        F[Utils]
        G[Types]
        H[Theme]
    end
    
    subgraph "Backend Modules"
        I[User Module]
        J[Appointment Module]
        K[Payment Module]
        L[Credit Module]
        M[Coupon Module]
        N[Video Call Module]
    end
    
    A --> C
    A --> D
    A --> E
    B --> C
    B --> D
    B --> E
    
    C --> I
    C --> J
    C --> K
    C --> L
    C --> M
    C --> N
    
    style A fill:#3b82f6
    style B fill:#8b5cf6
    style C fill:#10b981
```

---

## Database Schema

### Entity Relationship Overview

```mermaid
erDiagram
    User ||--o{ RolesOnUser : "has"
    User ||--o| Physio : "is"
    User ||--o| Patient : "is"
    User ||--o{ Payment : "makes"
    User ||--o{ Credit : "owns"
    User ||--o{ Answer : "provides"
    User ||--o{ Appointment : "physio"
    User ||--o{ Appointment : "patient"
    User ||--o{ CouponUsage : "uses"
    
    Role ||--o{ RolesOnUser : "assigned to"
    
    Physio ||--o{ PhysioAvailability : "has"
    
    Package ||--o{ Payment : "purchased"
    Package ||--o{ CouponsOnPackage : "applicable"
    
    Payment ||--o{ Credit : "generates"
    Payment ||--o{ CouponUsage : "includes"
    
    Coupon ||--o{ CouponUsage : "used in"
    Coupon ||--o{ CouponsOnUser : "assigned"
    Coupon ||--o{ CouponsOnPackage : "applies to"
    
    Appointment ||--o{ AnswersOnAppointment : "has"
    
    Questionnaire ||--o{ Question : "contains"
    
    Question ||--o{ Answer : "answered by"
    Question ||--o{ Question : "depends on"
    
    Answer ||--o{ AnswersOnAppointment : "linked to"
    
    User {
        uuid id PK
        string firstName
        string lastName
        string email UK
        enum status
        datetime birthDay
        string mobile
        string timeZone
        enum gender
    }
    
    Physio {
        uuid id PK
        uuid userId FK
        string specialty
        int totalYearsOfExperience
    }
    
    Patient {
        uuid id PK
        uuid userId FK
        float weight
        float height
        string occupation
    }
    
    Appointment {
        uuid id PK
        enum status
        uuid physioUserId FK
        uuid patientUserId FK
        datetime appointmentTime
    }
    
    Payment {
        uuid id PK
        enum paymentType
        enum status
        float amount
        uuid userId FK
        uuid packageId FK
    }
    
    Package {
        uuid id PK
        json name
        int credits
        float price
        float discount
    }
    
    Credit {
        uuid id PK
        uuid userId FK
        int total
        int used
    }
    
    Coupon {
        uuid id PK
        string code UK
        enum type
        enum discountType
        float discount
        boolean isActive
    }
```

**Relationships:**
- User has multiple RolesOnUser (many-to-many with Role)
- User can be one Physio (one-to-one)
- User can be one Patient (one-to-one)
- User makes multiple Payments
- User owns multiple Credits
- User provides multiple Answers
- User participates in Appointments as Physio or Patient
- User uses Coupons

**Key Tables:**

**User Table:**
- id (uuid, PK)
- firstName, lastName (string)
- email (string, unique)
- status (enum: IN_REVIEW, ACTIVE, INACTIVE, BLOCKED)
- profilePictureUrl, birthDay, address, city, country
- mobile, timeZone, zipCode
- languages (array), gender (enum)

**Physio Table:**
- id (uuid, PK)
- userId (uuid, FK to User)
- specialty, description
- totalYearsOfExperience (int)

**Patient Table:**
- id (uuid, PK)
- userId (uuid, FK to User)
- weight, height (float)
- activities (array)
- occupation (string)
- medicalDocuments (array)

**Role Table:**
- id (uuid, PK)
- name (string, unique)

**Appointment Table:**
- id (uuid, PK)
- status (enum: PENDING, SUCCESS, CANCELED)
- physioUserId, patientUserId (uuid, FK to User)
- note (string)
- appointmentTime (datetime)

**Payment Table:**
- id (uuid, PK)
- paymentType (enum: CARD, COUPON)
- status (enum: PENDING, SUCCESS, FAILED)
- amount (float)
- ipgPaymentId (string, unique)
- userId, packageId (uuid, FK)

**Package Table:**
- id (uuid, PK)
- name, description (json for i18n)
- feature (boolean)
- credits (int)
- price, discount (float)
- discountType (enum: FLAT, PERCENTAGE)

**Credit Table:**
- id (uuid, PK)
- userId, paymentId (uuid, FK)
- total, used (int)

**Coupon Table:**
- id (uuid, PK)
- code (string, unique)
- type (enum: PUBLIC_PACKAGE, PUBLIC_NON_PACKAGE, USER_GROUP_PACKAGE, USER_GROUP_NON_PACKAGE)
- discountType (enum: FLAT, PERCENTAGE)
- discount (float)
- maxUsePerUser, maxUses (int)
- isActive (boolean)
- expiresAt (datetime)

### Key Database Models

#### User Model (Central Entity)
- **Purpose**: Represents all users in the system
- **Status Types**: `IN_REVIEW`, `ACTIVE`, `INACTIVE`, `BLOCKED`
- **Relationships**: Can be a Physio, Patient, or Admin through `RolesOnUser`

#### Physio & Patient Models
- **Pattern**: Single Table Inheritance
- Each user has a single role extension (Physio XOR Patient)
- One-to-one relationship with User table

#### Appointment Model
- **Status Flow**: `PENDING` → `SUCCESS` or `CANCELED`
- Links patient and physio users
- Contains medical screening answers via `AnswersOnAppointment`

#### Payment & Credit System
- **Payment Types**: `CARD`, `COUPON`
- **Payment Status**: `PENDING`, `SUCCESS`, `FAILED`
- Credits are purchased through packages
- Each credit = 1 appointment booking

#### Coupon System
- **Types**: 
  - `PUBLIC_PACKAGE`: Available to all for packages
  - `PUBLIC_NON_PACKAGE`: General public discount
  - `USER_GROUP_PACKAGE`: Assigned to specific users for packages
  - `USER_GROUP_NON_PACKAGE`: Assigned to specific users
- **Discount Types**: `FLAT`, `PERCENTAGE`

#### PhysioAvailability
- **Recurrence Types**: 
  - `WEEKLY`: Recurring weekly slots (uses `dayOfWeek`)
  - `SPECIFIC_DATE`: One-time availability (uses `specificDate`)
- Stores time slots in `HH:mm` format

---

## Authentication & Authorization

### Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Keycloak
    participant Backend
    participant Database
    
    User->>Frontend: Login Request
    Frontend->>Keycloak: Authenticate (email/password)
    Keycloak->>Keycloak: Validate Credentials
    Keycloak-->>Frontend: JWT Token + Refresh Token
    
    Frontend->>Frontend: Store tokens
    
    Frontend->>Backend: API Request + JWT
    Backend->>Backend: Extract JWT from Header
    Backend->>Keycloak: Validate JWT (JWKS)
    Keycloak-->>Backend: Public Key
    Backend->>Backend: Verify Signature & Decode
    Backend->>Backend: Extract roles from token
    
    alt Valid Token & Authorized
        Backend->>Database: Execute Query
        Database-->>Backend: Return Data
        Backend-->>Frontend: Success Response
    else Invalid Token
        Backend-->>Frontend: 401 Unauthorized
    else Missing Permissions
        Backend-->>Frontend: 403 Forbidden
    end
    
    Frontend-->>User: Display Result
```

### Multi-Realm Strategy

The system uses **3 separate Keycloak realms** for role isolation:

1. **Patient Realm** (`PATIENT_JWK_URL`)
   - Patient users
   - Self-registration enabled
   - Limited administrative capabilities

2. **Physio Realm** (`PHYSIO_JWK_URL`)
   - Physiotherapist users
   - Admin-approved registration
   - Professional credentials

3. **Admin Realm** (`ADMIN_JWK_URL`)
   - Administrative users
   - Full system access
   - User management capabilities

### JWT Strategy Implementation

**File**: `backend-app/src/modules/auth/jwt.strategy.ts`

```typescript
// Key Features:
// 1. Dynamic JWKS URL selection based on token issuer
// 2. RS256 algorithm for asymmetric encryption
// 3. Public key caching for performance
// 4. Role extraction from Keycloak token payload
```

### Authorization Guards

```mermaid
graph TD
    A[Incoming Request] --> B{JWT Valid?}
    B -->|No| C[401 Unauthorized]
    B -->|Yes| D{Has Required Role?}
    D -->|No| E[403 Forbidden]
    D -->|Yes| F[Execute Controller]
    F --> G[Return Response]
    
    style C fill:#ef4444
    style E fill:#f97316
    style G fill:#10b981
```

**Guards Used:**
1. `JwtAuthGuard`: Validates JWT token
2. `RolesGuard`: Checks user roles against route requirements

**Decorator Example:**
```typescript
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(ROLES.PATIENT)
@Post('user/:userId/appointments')
```

**Available Roles:**
- `PATIENT`: Book appointments, view own data
- `PHYSIO`: View assigned appointments, manage availability
- `ADMIN`: Full system access, user management

---

## Core Business Flows

### 1. Patient Onboarding Flow

```mermaid
sequenceDiagram
    participant P as Patient
    participant FE as Frontend
    participant KC as Keycloak
    participant BE as Backend
    participant DB as Database
    
    P->>FE: Register Account
    FE->>KC: Create User (Patient Realm)
    KC-->>FE: User Created
    
    P->>FE: Login
    FE->>KC: Authenticate
    KC-->>FE: JWT Token
    
    FE->>BE: Create User Profile
    BE->>DB: Insert User Record
    DB-->>BE: User Created
    
    BE->>DB: Assign PATIENT Role
    DB-->>BE: Role Assigned
    
    FE->>BE: Complete Patient Profile
    BE->>DB: Insert Patient Record
    DB-->>BE: Patient Created
    
    BE-->>FE: Profile Complete
    FE-->>P: Welcome Dashboard
```

### 2. Package Purchase & Credit Allocation Flow

```mermaid
sequenceDiagram
    participant P as Patient
    participant FE as Frontend
    participant BE as Backend
    participant Stripe as Stripe
    participant DB as Database
    
    P->>FE: Select Package
    FE->>BE: Create Payment Intent
    
    opt With Coupon
        BE->>DB: Validate Coupon
        DB-->>BE: Coupon Valid
        BE->>BE: Calculate Discount
    end
    
    BE->>BE: Calculate Final Price
    
    alt Price = 0 (100% Discount)
        BE->>DB: Create FREE Payment Record
        BE->>DB: Allocate Credits
        BE->>DB: Log Coupon Usage
        BE-->>FE: Success (No Stripe)
    else Price > 0
        BE->>Stripe: Create Payment Intent
        Stripe-->>BE: Client Secret
        BE-->>FE: Payment Intent
        
        FE->>FE: Show Stripe Checkout
        P->>FE: Enter Card Details
        FE->>Stripe: Confirm Payment
        Stripe-->>FE: Payment Success
        
        Stripe->>BE: Webhook (payment_intent.succeeded)
        BE->>DB: Update Payment Status = SUCCESS
        BE->>DB: Allocate Credits (package.credits)
        BE->>DB: Log Coupon Usage (if applicable)
        BE-->>Stripe: Webhook Acknowledged
    end
    
    FE-->>P: Credits Available
```

**Credit Allocation Logic:**
- Each package has a `credits` value
- Payment success triggers credit creation
- `Credit.total` = package credits
- `Credit.used` starts at 0
- Available credits = `total - used`

### 3. Appointment Booking Flow

```mermaid
sequenceDiagram
    participant P as Patient
    participant FE as Frontend
    participant BE as Backend
    participant DB as Database
    participant Email as Email Service
    
    P->>FE: Browse Physiotherapists
    FE->>BE: GET /physios
    BE->>DB: Fetch Physios
    DB-->>BE: Physio List
    BE-->>FE: Physio Profiles
    
    P->>FE: Select Physio
    FE->>BE: GET /physio-availability/:physioId
    BE->>DB: Fetch Availability Slots
    DB-->>BE: Available Time Slots
    BE-->>FE: Time Slots
    
    P->>FE: Select Time + Fill Questionnaire
    FE->>BE: POST /user/:userId/appointments
    
    BE->>DB: Check User Credits
    DB-->>BE: Credits Available
    
    alt Credits Available
        BE->>DB: Create Appointment (PENDING)
        BE->>DB: Decrease Credit (total - 1)
        BE->>DB: Save Questionnaire Answers
        DB-->>BE: Appointment Created
        
        BE->>Email: Notify Physio
        BE->>Email: Confirm to Patient
        
        BE-->>FE: Success
        FE-->>P: Appointment Confirmed
    else No Credits
        BE-->>FE: 400 Insufficient Credits
        FE-->>P: Purchase Package Prompt
    end
```

**Key Business Rules:**
1. **Credit Check**: Must have available credits before booking
2. **Availability Validation**: Time slot must be within physio's availability
3. **Questionnaire Required**: Medical screening must be completed
4. **Status Flow**: 
   - Created as `PENDING`
   - Updated to `SUCCESS` after video call completion
   - Can be `CANCELED` by patient

### 4. Video Consultation Flow

```mermaid
sequenceDiagram
    participant P as Patient
    participant D as Physio
    participant FE_P as Patient Frontend
    participant FE_D as Physio Frontend
    participant BE as Backend
    participant Zoom as Zoom SDK
    
    Note over P,D: 5 minutes before appointment
    
    P->>FE_P: Click "Join Appointment"
    FE_P->>BE: POST /video-call/generate-zoom-token
    BE->>BE: Generate JWT for Zoom
    BE-->>FE_P: Zoom Token + Session Details
    
    FE_P->>Zoom: Initialize Zoom SDK
    Zoom-->>FE_P: Video Room Ready
    FE_P-->>P: Show Video Interface
    
    D->>FE_D: Click "Join Appointment"
    FE_D->>BE: POST /video-call/generate-zoom-token
    BE->>BE: Generate JWT for Zoom
    BE-->>FE_D: Zoom Token + Session Details
    
    FE_D->>Zoom: Initialize Zoom SDK
    Zoom-->>FE_D: Video Room Ready
    FE_D-->>D: Show Video Interface
    
    Note over P,D: Video consultation session
    
    D->>FE_D: Complete Appointment
    FE_D->>BE: PATCH /appointments/:id (status=SUCCESS)
    BE->>BE: Update Appointment Status
    BE-->>FE_D: Success
    FE_D-->>D: Appointment Completed
```

**Zoom Integration Details:**
- Uses Zoom Video SDK (not Zoom Meetings)
- Token generated using `jsrsasign` library
- Session name = Appointment ID
- Role-based permissions (host vs participant)

### 5. Physio Availability Management Flow

```mermaid
graph TD
    A[Physio Login] --> B{Set Availability Type}
    
    B -->|Weekly Recurring| C[Select Day of Week]
    C --> D[Set Start/End Time]
    D --> E[Save Weekly Slot]
    E --> F[Repeats Every Week]
    
    B -->|Specific Date| G[Select Calendar Date]
    G --> H[Set Start/End Time]
    H --> I[Save Date-Specific Slot]
    I --> J[One-time Availability]
    
    F --> K[Patients Can Book]
    J --> K
    
    K --> L{Appointment Booked}
    L --> M[Slot Blocked for Duration]
    
    style E fill:#10b981
    style I fill:#3b82f6
    style M fill:#f59e0b
```

**Availability Rules:**
1. **Unique Constraint**: Cannot have overlapping slots for same day
2. **Time Format**: Stored as `HH:mm` (24-hour format)
3. **Active Flag**: Can disable without deleting (`isActive`)
4. **Booking Validation**: Checks both weekly and specific date availability

---

## Module Structure

### Backend Module Architecture

Each module follows NestJS best practices with consistent structure:

```
module-name/
├── dto/
│   ├── request/          # Input DTOs
│   │   ├── create-*.dto.ts
│   │   ├── update-*.dto.ts
│   │   └── query-*.dto.ts
│   └── response/         # Output DTOs
│       └── *-response.dto.ts
├── entities/             # Prisma entity classes
│   └── *.entity.ts
├── repository/           # Data access layer
│   └── *.repository.ts
├── service/              # Business logic
│   ├── *.service.ts
│   └── *.service.spec.ts
├── utils/                # Module-specific utilities
│   ├── constants.ts
│   └── *.profile.ts      # AutoMapper profiles
├── *.controller.ts       # API endpoints
├── *.controller.spec.ts  # Controller tests
└── *.module.ts           # Module definition
```

### Key Modules Overview

```mermaid
graph TD
    subgraph "Core Modules"
        A[User Module]
        B[Auth Module]
        C[Roles Module]
    end
    
    subgraph "Business Modules"
        D[Appointment Module]
        E[Payment Module]
        F[Credit Module]
        G[Coupon Module]
        H[Package Module]
    end
    
    subgraph "Medical Modules"
        I[Patient Module]
        J[Physio Module]
        K[Question Module]
        L[Questionnaire Module]
        M[Answer Module]
    end
    
    subgraph "Integration Modules"
        N[Video Call Module]
        O[Physio Availability Module]
    end
    
    A --> I
    A --> J
    B --> A
    C --> A
    
    D --> A
    D --> M
    E --> F
    E --> G
    G --> H
    
    K --> L
    M --> K
    D --> M
    
    J --> O
    N --> D
    
    style A fill:#3b82f6
    style D fill:#10b981
    style E fill:#f59e0b
```

### Module Descriptions

#### 1. **User Module**
- **Purpose**: Core user management
- **Responsibilities**:
  - CRUD operations for users
  - Profile management
  - User status updates
- **Key Files**:
  - `users.service.ts`: Business logic
  - `users.repository.ts`: Prisma queries
  - `users.controller.ts`: REST endpoints

#### 2. **Auth Module**
- **Purpose**: Authentication and authorization
- **Responsibilities**:
  - JWT validation via Keycloak
  - Role-based access control
  - Token verification
- **Key Files**:
  - `jwt.strategy.ts`: Passport JWT strategy
  - `jwt-auth.guard.ts`: Authentication guard
  - `roles.guard.ts`: Authorization guard

#### 3. **Appointment Module**
- **Purpose**: Appointment lifecycle management
- **Responsibilities**:
  - Create appointments
  - Update appointment status
  - Link questionnaire answers
  - Query appointments with filters
- **Key Endpoints**:
  - `POST /user/:userId/appointments`
  - `GET /appointments`
  - `PATCH /user/:userId/appointments/:id`

#### 4. **Payment Module**
- **Purpose**: Payment processing and Stripe integration
- **Responsibilities**:
  - Create Stripe payment intents
  - Handle Stripe webhooks
  - Process payment success/failure
  - Handle 100% coupon (free checkout)
- **Key Endpoints**:
  - `POST /payments/stripe/intent`
  - `POST /payments/stripe/notify` (webhook)
  - `GET /users/:userId/payments`

#### 5. **Credit Module**
- **Purpose**: Credit management for appointments
- **Responsibilities**:
  - Allocate credits after payment
  - Track credit usage
  - Calculate available credits
  - Decrease credits on booking
- **Business Logic**:
  ```typescript
  Available Credits = SUM(credit.total) - SUM(credit.used)
  ```

#### 6. **Coupon Module**
- **Purpose**: Discount and promotion management
- **Responsibilities**:
  - Validate coupon codes
  - Apply discounts (flat/percentage)
  - Track coupon usage
  - Enforce usage limits
- **Validation Rules**:
  - Check expiration
  - Verify user eligibility
  - Check max uses per user
  - Check global max uses

#### 7. **Package Module**
- **Purpose**: Credit package definitions
- **Responsibilities**:
  - List available packages
  - Calculate pricing with discounts
  - Support internationalization (JSON fields)
- **Data Structure**:
  ```json
  {
    "name": {"en": "Basic Plan", "de": "Basisplan"},
    "credits": 5,
    "price": 99.99,
    "discount": 10,
    "discountType": "PERCENTAGE"
  }
  ```

#### 8. **Physio Module**
- **Purpose**: Physiotherapist profile management
- **Responsibilities**:
  - Create physio profiles
  - List physios for patient selection
  - Update professional details
- **Profile Fields**:
  - Specialty
  - Years of experience
  - Description/Bio

#### 9. **Patient Module**
- **Purpose**: Patient profile management
- **Responsibilities**:
  - Create patient profiles
  - Store medical information
  - Manage medical documents
- **Profile Fields**:
  - Physical metrics (height, weight)
  - Activities
  - Occupation
  - Medical documents (URLs)

#### 10. **Question/Questionnaire/Answer Modules**
- **Purpose**: Dynamic medical screening system
- **Responsibilities**:
  - Define questionnaires
  - Manage conditional questions
  - Store patient answers
  - Link answers to appointments
- **Features**:
  - Conditional logic (question depends on previous answer)
  - Multiple question types (TEXT, RADIO, MULTIPLE_CHOICE)
  - Gender-specific questions
  - Multi-language support

#### 11. **Video Call Module**
- **Purpose**: Zoom video integration
- **Responsibilities**:
  - Generate Zoom SDK tokens
  - Manage session credentials
- **Token Generation**:
  - Uses appointment ID as session name
  - Role-based token (host vs participant)
  - Time-limited tokens

#### 12. **Physio Availability Module**
- **Purpose**: Scheduling and availability management
- **Responsibilities**:
  - Set weekly recurring availability
  - Set date-specific availability
  - Validate time slots
  - Check booking conflicts
- **Recurrence Types**:
  - `WEEKLY`: Repeats every week on specified day
  - `SPECIFIC_DATE`: One-time availability

---

## Frontend Architecture

### Nx Monorepo Structure

```
frontend-web-app-2/
├── apps/
│   ├── patient-portal/      # Patient-facing application
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── [locale]/    # Internationalized routes
│   │   │   │   ├── api/          # API routes (NextAuth)
│   │   │   │   └── layout.tsx
│   │   │   ├── components/       # App-specific components
│   │   │   ├── i18n/             # i18n configuration
│   │   │   └── middleware.ts     # Auth & locale middleware
│   │   ├── languages/            # Translation files
│   │   └── Dockerfile
│   └── doctor-portal/       # Physio-facing application
│       ├── src/
│       │   ├── app/
│       │   ├── components/
│       │   └── middleware.ts
│       ├── languages/
│       └── Dockerfile
├── libs/                    # Shared libraries
│   ├── api/                 # API client functions
│   ├── components/          # Shared UI components
│   ├── hooks/               # Shared React hooks
│   ├── types/               # TypeScript types
│   ├── utils/               # Utility functions
│   ├── constants/           # Shared constants
│   ├── auth/                # Auth utilities
│   ├── axios/               # HTTP client config
│   ├── react-query/         # TanStack Query config
│   └── theme/               # Tailwind theme
└── package.json
```

### Shared Libraries Strategy

**Benefits of Nx Monorepo:**
1. **Code Reusability**: Share components, hooks, and utils
2. **Type Safety**: Shared types between apps
3. **Consistency**: Same UI components across portals
4. **DRY Principle**: Single source of truth for API clients
5. **Efficient Builds**: Only rebuild affected apps

### Frontend Data Flow

```mermaid
graph TD
    A[User Interaction] --> B[React Component]
    B --> C{Data Needed?}
    
    C -->|No| D[Local State<br/>useState/Zustand]
    
    C -->|Yes| E[TanStack Query Hook]
    E --> F{Cache Hit?}
    
    F -->|Yes| G[Return Cached Data]
    F -->|No| H[API Client Call]
    
    H --> I[Axios Interceptor]
    I --> J{Has Token?}
    
    J -->|Yes| K[Add Bearer Token]
    J -->|No| L[Redirect to Login]
    
    K --> M[Send Request to Backend]
    M --> N{Response Status}
    
    N -->|200-299| O[Update Cache]
    N -->|401| P[Refresh Token/Logout]
    N -->|400/500| Q[Show Error Toast]
    
    O --> R[Update UI]
    G --> R
    
    D --> R
    
    style R fill:#10b981
    style P fill:#ef4444
    style Q fill:#f97316
```

### Key Frontend Patterns

#### 1. **Server State Management (TanStack Query)**

```typescript
// Example: Fetch appointments
const { data, isLoading, error } = useQuery({
  queryKey: ['appointments', userId],
  queryFn: () => appointmentApi.getAppointments(userId),
  staleTime: 5 * 60 * 1000, // 5 minutes
});
```

**Benefits:**
- Automatic caching
- Background refetching
- Optimistic updates
- Request deduplication

#### 2. **Client State Management (Zustand)**

```typescript
// Example: Breadcrumb store
interface BreadcrumbState {
  items: BreadcrumbItem[];
  setBreadcrumbs: (items: BreadcrumbItem[]) => void;
}

const useBreadcrumbStore = create<BreadcrumbState>((set) => ({
  items: [],
  setBreadcrumbs: (items) => set({ items }),
}));
```

**Use Cases:**
- UI state (modals, dialogs)
- Navigation state (breadcrumbs)
- Temporary form state

#### 3. **Form Management (React Hook Form + Zod)**

```typescript
const schema = z.object({
  firstName: z.string().min(1, 'Required'),
  email: z.string().email('Invalid email'),
});

const form = useForm({
  resolver: zodResolver(schema),
  defaultValues: { firstName: '', email: '' },
});
```

**Benefits:**
- Type-safe validation
- Performance (uncontrolled components)
- Easy error handling

#### 4. **Authentication (NextAuth)**

```typescript
// Keycloak provider configuration
providers: [
  KeycloakProvider({
    clientId: process.env.KEYCLOAK_CLIENT_ID,
    clientSecret: process.env.KEYCLOAK_CLIENT_SECRET,
    issuer: process.env.KEYCLOAK_ISSUER,
  }),
]
```

**Features:**
- JWT session handling
- Automatic token refresh
- Protected routes via middleware

#### 5. **Internationalization (next-intl)**

```typescript
// Usage in components
const t = useTranslations('AppointmentPage');

<h1>{t('title')}</h1>
```

**Supported Locales:**
- `en`: English
- `de`: German

### Component Architecture

```mermaid
graph TD
    A[Page Component] --> B[Layout Component]
    B --> C[Feature Component]
    C --> D[UI Components<br/>libs/components]
    
    C --> E[Hooks<br/>libs/hooks]
    E --> F[API Client<br/>libs/api]
    
    C --> G[Local State<br/>useState/useForm]
    C --> H[Global State<br/>Zustand]
    C --> I[Server State<br/>TanStack Query]
    
    D --> J[Radix UI Primitives]
    D --> K[Custom Styled Components]
    
    style A fill:#3b82f6
    style D fill:#10b981
    style F fill:#f59e0b
```

### Styling Strategy

**Tailwind CSS + shadcn/ui Pattern:**

1. **Utility-First Approach**
   ```tsx
   <div className="flex items-center gap-4 rounded-lg border p-4">
   ```

2. **Component Variants (CVA)**
   ```typescript
   const buttonVariants = cva(
     "inline-flex items-center justify-center rounded-md",
     {
       variants: {
         variant: {
           default: "bg-primary text-white",
           outline: "border border-gray-300",
         },
         size: {
           sm: "h-9 px-3",
           lg: "h-11 px-8",
         },
       },
     }
   );
   ```

3. **Design System**
   - Consistent spacing (4px grid)
   - Color palette (primary, secondary, accent)
   - Typography scale
   - Component library (Button, Card, Dialog, etc.)

---

## API Documentation

### API Versioning

```
Base URL: https://api.wexel.com/api/v1
```

**Versioning Strategy:**
- URI-based versioning (`/api/v1`, `/api/v2`)
- Configured in `main.ts`:
  ```typescript
  app.enableVersioning({
    type: VersioningType.URI,
    prefix: 'api/v',
  });
  ```

### Swagger Documentation

**Available in non-production environments:**

```
URL: http://localhost:3000/api/docs
```

**Features:**
- Interactive API testing
- Request/response schemas
- Authentication (Bearer token)
- Try it out functionality

### API Response Format

**Standardized Response Structure:**

```typescript
// Success Response
{
  "statusCode": 200,
  "message": "Success message",
  "data": { /* response payload */ }
}

// Paginated Response
{
  "statusCode": 200,
  "message": "Success message",
  "data": {
    "items": [ /* array of items */ ],
    "meta": {
      "page": 1,
      "limit": 10,
      "total": 100,
      "totalPages": 10
    }
  }
}

// Error Response
{
  "statusCode": 400,
  "message": "Error message",
  "error": "Bad Request"
}
```

**Implemented via `ResponseInterceptor`**

### Key API Endpoints

#### Authentication
- N/A - Handled by Keycloak

#### Users
- `GET /api/v1/users/:id` - Get user profile
- `PATCH /api/v1/users/:id` - Update user profile
- `GET /api/v1/users` - List users (Admin only)

#### Appointments
- `POST /api/v1/user/:userId/appointments` - Create appointment (Patient)
- `GET /api/v1/user/:userId/appointments` - List user appointments
- `GET /api/v1/user/:userId/appointments/:id` - Get appointment details
- `PATCH /api/v1/user/:userId/appointments/:id` - Update appointment status
- `GET /api/v1/appointments` - List all appointments (Admin/Physio)

#### Payments
- `POST /api/v1/payments/stripe/intent` - Create payment intent
- `POST /api/v1/payments/stripe/notify` - Stripe webhook (no auth)
- `GET /api/v1/users/:userId/payments` - List user payments
- `GET /api/v1/users/:userId/payments/:id` - Get payment details

#### Credits
- `GET /api/v1/users/:userId/credits/total` - Get available credits

#### Packages
- `GET /api/v1/packages` - List available packages
- `GET /api/v1/packages/:id` - Get package details

#### Coupons
- `POST /api/v1/coupons/validate` - Validate coupon code
- `GET /api/v1/coupons` - List coupons (Admin only)

#### Physios
- `GET /api/v1/physios` - List physiotherapists
- `GET /api/v1/physios/:id` - Get physio profile

#### Physio Availability
- `POST /api/v1/physio-availability` - Set availability (Physio)
- `GET /api/v1/physio-availability/:physioId` - Get physio availability
- `DELETE /api/v1/physio-availability/:id` - Remove availability (Physio)

#### Video Call
- `POST /api/v1/video-call/generate-zoom-token` - Generate Zoom token

#### Questionnaires
- `GET /api/v1/questionnaires` - List questionnaires
- `GET /api/v1/questionnaires/:id` - Get questionnaire with questions

#### Answers
- `POST /api/v1/answers` - Submit questionnaire answers
- `GET /api/v1/users/:userId/answers` - Get user answers

---

## Deployment

### Infrastructure Overview

```mermaid
graph TB
    subgraph "Azure DevOps"
        A[Git Repository]
        B[Azure Pipelines]
    end
    
    subgraph "Build Process"
        C[Build Backend Docker Image]
        D[Build Patient Portal Image]
        E[Build Doctor Portal Image]
    end
    
    subgraph "Container Registry"
        F[Azure Container Registry]
    end
    
    subgraph "Kubernetes Cluster"
        G[Backend Deployment]
        H[Patient Portal Deployment]
        I[Doctor Portal Deployment]
        J[PostgreSQL StatefulSet]
    end
    
    subgraph "Services"
        K[LoadBalancer Service]
        L[ClusterIP Services]
    end
    
    A --> B
    B --> C
    B --> D
    B --> E
    
    C --> F
    D --> F
    E --> F
    
    F --> G
    F --> H
    F --> I
    
    G --> L
    H --> L
    I --> L
    
    L --> K
    
    J --> L
    
    style F fill:#3b82f6
    style K fill:#10b981
```

### Environments

The project has **3 environments**:

1. **Development** (`-dev` pipelines)
   - Auto-deploy on `develop` branch
   - Development Keycloak realms
   - Debug logging enabled

2. **QA/Staging** (`-qa` pipelines)
   - Manual trigger from `qa` branch
   - Staging Keycloak realms
   - Production-like configuration

3. **Production** (`-prod` pipelines)
   - Manual trigger from `main` branch
   - Production Keycloak realms
   - Optimized builds, minimal logging

### Pipeline Configuration Files

**Backend:**
- `azure-pipelines-dev.yml`
- `azure-pipelines-qa.yml`
- `azure-pipelines-prod.yml`

**Frontend:**
- `azure-pipelines-dev.yml`
- `azure-pipelines-qa.yml`
- `azure-pipelines-prod.yml`

### Deployment Process

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Git as Git Repository
    participant Pipeline as Azure Pipeline
    participant Docker as Docker Build
    participant ACR as Container Registry
    participant K8s as Kubernetes
    
    Dev->>Git: Push to branch
    Git->>Pipeline: Trigger pipeline
    
    Pipeline->>Pipeline: Install dependencies
    Pipeline->>Pipeline: Run linter
    Pipeline->>Pipeline: Run tests
    
    Pipeline->>Docker: Build Docker image
    Docker->>ACR: Push image
    
    Pipeline->>K8s: Update deployment
    K8s->>ACR: Pull new image
    K8s->>K8s: Rolling update
    K8s->>K8s: Health checks
    
    K8s-->>Pipeline: Deployment success
    Pipeline-->>Dev: Notification
```

### Kubernetes Manifests

**Backend Deployment** (`backend-app/manifests/deployment.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: backend
        image: ${ACR_NAME}/backend-app:${TAG}
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: backend-secrets
              key: database-url
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
```

**Backend Service** (`backend-app/manifests/service.yaml`):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: backend-app
```

### Database Migrations

**Migration Strategy:**

1. **Development**:
   ```bash
   cd backend-app
   pnpm prisma migrate dev --name migration_name
   ```

2. **Production**:
   ```bash
   pnpm prisma migrate deploy
   ```
   - Runs automatically in deployment pipeline
   - Never rolls back automatically
   - Manual intervention if migration fails

**Seeding:**

```bash
pnpm prisma db seed
```

Seeds:
- Roles (PATIENT, PHYSIO, ADMIN)
- Packages (default credit packages)
- Questionnaires (medical screening questions)

---

## Getting Started

### Prerequisites

**Required:**
- Node.js 18+ (LTS recommended)
- pnpm 8+ (`npm install -g pnpm`)
- PostgreSQL 14+
- Docker (optional, for containerized development)

**Accounts Needed:**
- Keycloak instance (or local setup)
- Stripe account (test mode)
- Zoom Video SDK credentials
- Email service credentials (SMTP)

### Backend Setup

#### 1. Clone Repository

```bash
git clone <repository-url>
cd wexel
```

#### 2. Install Backend Dependencies

```bash
cd backend-app
pnpm install
```

#### 3. Environment Configuration

Create `.env` file from `sample.env`:

```bash
cp sample.env .env
```

**Key Environment Variables:**

```env
# Application
NODE_ENV=development
PORT=3000
HOST=0.0.0.0

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/wexel_db

# Keycloak Authentication
PATIENT_JWK_URL=https://keycloak.example.com/realms/patient
PHYSIO_JWK_URL=https://keycloak.example.com/realms/physio
ADMIN_JWK_URL=https://keycloak.example.com/realms/admin

# Stripe
STRIPE_SECRET_KEY=sk_test_xxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxx

# Zoom Video SDK
ZOOM_SDK_KEY=xxxxx
ZOOM_SDK_SECRET=xxxxx

# Email
EMAIL_HOST=smtp.example.com
EMAIL_PORT=587
EMAIL_USER=noreply@example.com
EMAIL_PASSWORD=xxxxx
EMAIL_FROM=Wexel <noreply@example.com>
```

#### 4. Database Setup

```bash
# Run migrations
pnpm prisma migrate dev

# Seed initial data
pnpm prisma db seed

# (Optional) Open Prisma Studio
pnpm prisma studio
```

#### 5. Start Backend

```bash
# Development mode (hot reload)
pnpm run start:dev

# Production mode
pnpm run build
pnpm run start:prod
```

**Backend will run on:** `http://localhost:3000`  
**API Docs:** `http://localhost:3000/api/docs`

### Frontend Setup

#### 1. Install Frontend Dependencies

```bash
cd frontend-web-app-2
pnpm install
```

#### 2. Environment Configuration

**Patient Portal** (`apps/patient-portal/env.sample` → `.env.local`):

```env
# NextAuth
NEXTAUTH_URL=http://localhost:3001
NEXTAUTH_SECRET=your-secret-key

# Keycloak
KEYCLOAK_CLIENT_ID=patient-portal
KEYCLOAK_CLIENT_SECRET=xxxxx
KEYCLOAK_ISSUER=https://keycloak.example.com/realms/patient

# Backend API
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxxxx

# Sanity CMS (for blog content)
NEXT_PUBLIC_SANITY_PROJECT_ID=xxxxx
NEXT_PUBLIC_SANITY_DATASET=production
```

**Doctor Portal** (similar configuration with physio realm)

#### 3. Start Frontend Applications

```bash
# Patient Portal (port 3001)
pnpm patient-portal:dev

# Doctor Portal (port 3002)
pnpm doctor-portal:dev
```

**Access Applications:**
- Patient Portal: `http://localhost:3001`
- Doctor Portal: `http://localhost:3002`

### Development Workflow

#### 1. Code Structure Guidelines

**Backend:**
- Follow NestJS module structure
- Use DTOs for all inputs/outputs
- Implement repository pattern for data access
- Use AutoMapper for entity-to-DTO conversion
- Write unit tests for services
- Document complex business logic

**Frontend:**
- Use functional components with hooks
- Keep components small and focused
- Use TanStack Query for server state
- Use Zustand for client state
- Follow TypeScript strict mode
- Use Tailwind for styling

#### 2. Git Workflow

```bash
# Create feature branch
git checkout -b feature/feature-name

# Make changes and commit
git add .
git commit -m "feat: add feature description"

# Push to remote
git push origin feature/feature-name

# Create Pull Request
# Request code review
# Merge after approval
```

**Commit Message Convention:**
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes
- `refactor:` Code refactoring
- `test:` Test changes
- `chore:` Build/dependency changes

#### 3. Testing

**Backend:**
```bash
# Unit tests
pnpm test

# E2E tests
pnpm test:e2e

# Test coverage
pnpm test:cov
```

**Frontend:**
```bash
# Run tests
pnpm nx test patient-portal
pnpm nx test doctor-portal

# Run linter
pnpm lint
pnpm lint:fix
```

#### 4. Database Changes

**Creating a migration:**

```bash
cd backend-app

# Create migration
pnpm prisma migrate dev --name descriptive_name

# Example: add new field
pnpm prisma migrate dev --name add_user_phone_verified_field
```

**Updating seed data:**

Edit `prisma/seeds/seed.ts` and run:

```bash
pnpm prisma db seed
```

### Troubleshooting

#### Backend Issues

**Port already in use:**
```bash
# Find process
lsof -i :3000

# Kill process
kill -9 <PID>
```

**Database connection failed:**
- Check PostgreSQL is running
- Verify DATABASE_URL in .env
- Check database exists

**Prisma client out of sync:**
```bash
pnpm prisma generate
```

#### Frontend Issues

**Module not found:**
```bash
# Clear nx cache
pnpm nx reset

# Reinstall dependencies
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

**Type errors:**
```bash
# Regenerate types
pnpm nx reset
```

**Build errors:**
```bash
# Clear Next.js cache
rm -rf apps/patient-portal/.next
rm -rf apps/doctor-portal/.next
```

### Useful Commands

**Backend:**
```bash
# Generate new module
nest generate module module-name
nest generate controller module-name
nest generate service module-name

# Format code
pnpm format

# Lint
pnpm lint
```

**Frontend:**
```bash
# Generate component library
nx g @nx/react:lib my-lib

# Generate component
nx g @nx/react:component MyComponent --project=patient-portal

# Build for production
pnpm patient-portal:build
pnpm doctor-portal:build
```

---

## 📖 Additional Resources

### Documentation Links

- **NestJS**: https://docs.nestjs.com
- **Next.js**: https://nextjs.org/docs
- **Prisma**: https://www.prisma.io/docs
- **TanStack Query**: https://tanstack.com/query/latest
- **Radix UI**: https://www.radix-ui.com
- **Tailwind CSS**: https://tailwindcss.com/docs
- **NextAuth**: https://next-auth.js.org
- **Stripe**: https://stripe.com/docs
- **Zoom Video SDK**: https://developers.zoom.us/docs/video-sdk

### Team Contacts

- **Project Lead**: thaksharadhananjaya@gmail.com
- **Architecture Questions**: [Architecture team]
- **DevOps Support**: [DevOps team]

### Code Review Checklist

- [ ] Code follows project structure conventions
- [ ] All DTOs have proper validation
- [ ] Business logic is in services, not controllers
- [ ] Database queries use repository pattern
- [ ] Proper error handling implemented
- [ ] API endpoints have Swagger documentation
- [ ] Frontend components are typed
- [ ] Tests added/updated
- [ ] No hardcoded values (use env vars)
- [ ] Security best practices followed
- [ ] Performance considerations addressed

---

## Security Considerations

### Authentication Security
- JWT tokens expire after 15 minutes (configurable in Keycloak)
- Refresh tokens used for session extension
- HTTPS required in production
- CORS configured for specific origins

### Data Protection
- Passwords never stored (handled by Keycloak)
- Sensitive data encrypted at rest
- PII (Personally Identifiable Information) access logged
- GDPR compliance considerations

### API Security
- Rate limiting on authentication endpoints
- Request validation using class-validator
- SQL injection prevention via Prisma
- XSS protection via Helmet
- CSRF protection enabled

### Payment Security
- Stripe handles all card data (PCI DSS compliant)
- Webhook signature verification required
- Payment amounts validated server-side
- Idempotency keys for payment operations

---

## Glossary

- **Physio**: Physiotherapist (healthcare provider)
- **Patient**: Service recipient (client)
- **Appointment**: Scheduled video consultation session
- **Credit**: Unit of currency for booking appointments
- **Package**: Bundle of credits for purchase
- **Coupon**: Discount code for packages/appointments
- **Questionnaire**: Medical screening form
- **Availability**: Physio's available time slots
- **Session**: Video call meeting
- **Realm**: Keycloak authentication domain
- **IPG**: Internet Payment Gateway (Stripe)
- **DTO**: Data Transfer Object
- **ORM**: Object-Relational Mapping (Prisma)

---

## Quick Reference

### Environment Variables Checklist

**Backend (.env):**
```
- DATABASE_URL
- PATIENT_JWK_URL
- PHYSIO_JWK_URL
- ADMIN_JWK_URL
- STRIPE_SECRET_KEY
- STRIPE_WEBHOOK_SECRET
- ZOOM_SDK_KEY
- ZOOM_SDK_SECRET
- EMAIL_HOST, EMAIL_PORT, EMAIL_USER, EMAIL_PASSWORD
```

**Frontend (.env.local):**
```
- NEXTAUTH_URL
- NEXTAUTH_SECRET
- KEYCLOAK_CLIENT_ID
- KEYCLOAK_CLIENT_SECRET
- KEYCLOAK_ISSUER
- NEXT_PUBLIC_API_URL
- NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY
```

### Port Reference

| Service | Port | URL |
|---------|------|-----|
| Backend API | 3000 | http://localhost:3000 |
| API Docs | 3000 | http://localhost:3000/api/docs |
| Patient Portal | 3001 | http://localhost:3001 |
| Doctor Portal | 3002 | http://localhost:3002 |
| PostgreSQL | 5432 | localhost:5432 |

### Common Commands

```bash
# Backend
cd backend-app
pnpm start:dev              # Start backend
pnpm prisma studio          # Open database GUI
pnpm prisma migrate dev     # Run migrations

# Frontend
cd frontend-web-app-2
pnpm patient-portal:dev     # Start patient portal
pnpm doctor-portal:dev      # Start doctor portal
pnpm lint:fix               # Fix linting issues

# Database
psql -U postgres -d wexel_db    # Connect to DB
```

---

**Welcome to the Wexel team!**

If you have questions not covered in this document, please reach out to the team or update this document for future developers.

Happy coding!


