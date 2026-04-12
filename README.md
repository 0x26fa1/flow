# Gate Pass Module Flow

This flow covers these modules:

- `Gate Pass Types & Pricing`
- `Gate Pass Management`
- `Vehicle In & Out`

It follows the business rule you described:

1. Customer enters the terminal.
2. A gate pass is issued.
3. Vehicle In & Out logs the vehicle as inside.
4. Payment only happens when the customer decides to exit.

## Business Flow Diagram

```mermaid
flowchart TD
    A[Admin configures gate pass types and prices<br/>Gate Pass Types & Pricing] --> B[Customer arrives at terminal]
    B --> C[Staff selects applicable gate pass type and price]
    C --> D[Gate Pass Management issues gate pass<br/>pass_no generated<br/>trans_status = ACTIVE]
    D --> E[Vehicle In & Out validates gate pass]
    E --> F[Vehicle In & Out creates entry record<br/>timeIn recorded<br/>vehicleStatus = Inside<br/>queueStatus = Waiting]
    F --> G[Customer stays inside terminal]
    G --> H{Customer wants to exit?}
    H -- No --> G
    H -- Yes --> I[Compute payable amount based on gate pass type / pricing]
    I --> J[Customer pays exit fee]
    J --> K[Vehicle In & Out registers exit<br/>timeOut recorded<br/>vehicleStatus = Dispatched]
    K --> L[Vehicle In & Out verifies exit]
    L --> M[Transaction completed]
```

## Module Interaction Diagram

```mermaid
sequenceDiagram
    participant U as Customer / User
    participant P as Gate Pass Types & Pricing
    participant G as Gate Pass Management
    participant V as Vehicle In & Out
    participant X as Payment Step

    Note over P: Setup module for pass types and prices
    U->>G: Request entry / gate pass issuance
    G->>P: Read applicable pass type and price
    G-->>U: Issue gate pass number
    G-->>V: Gate pass becomes available for entry validation

    U->>V: Present gate pass at entry
    V->>G: Validate gate pass status
    G-->>V: Gate pass is valid and active
    V->>V: Record vehicle timeIn
    V->>G: Sync gate pass usage status

    Note over U,V: Vehicle is inside terminal

    U->>V: Request exit
    V->>X: Trigger payment step
    X-->>V: Payment confirmed
    V->>V: Record timeOut and dispatch vehicle
    V-->>U: Exit completed
```

## Current Code Notes

- `Gate Pass Types & Pricing` stores pass type and final price setup data.
- `Gate Pass Management` issues the gate pass and stores `pass_no`, `vehicle`, `driver`, `route`, `issue_datetime`, `expiry_datetime`, and `trans_status`.
- `Vehicle In & Out` validates the gate pass, records `timeIn`, manages inside/dispatched state, and records `timeOut`.
- In the current codebase, payment is not yet implemented inside these three modules.
- The current `Vehicle In & Out` flow changes gate pass status from `ACTIVE` to `USED` once entry is recorded.
- There is no direct foreign-key link yet from `Gate Pass Management` to `Gate Pass Types & Pricing`, so pricing is currently a business reference step rather than a persisted module-to-module relation.
