# Database Design

> This document describes the initial conceptual database model for InventoryOS.
> The schema is expected to evolve as development progresses and database concepts are learned.

## Database Goals

The InventoryOS database should prioritize:

- Data integrity
- Traceability
- Reliable inventory balances
- Clear relationships between data
- Safe inventory transactions
- Scalability without unnecessary complexity

PostgreSQL is the planned relational database.

## Core Entities

### Items

Represents a type of physical inventory.

Potential fields:

- id
- sku
- name
- description
- reorder_level
- is_active
- created_at
- updated_at

Important rules:

- `id` uniquely identifies the database record.
- `sku` must be unique.
- Items with historical activity should normally be deactivated rather than deleted.

---

### Locations

Represents a physical place where inventory can exist.

Potential fields:

- id
- name
- description
- is_active
- created_at
- updated_at

Examples of locations include warehouses, stores, stockrooms, freezers, vehicles, offices, and workshops.

---

### Inventory Balances

Represents the current quantity of an item at a particular location.

Potential fields:

- id
- item_id
- location_id
- quantity
- updated_at

The combination of `item_id` and `location_id` should be unique.

Example:

Item:

`USB-C Cable`

Location:

`Main Stockroom`

Quantity:

`125`

This represents 125 USB-C cables currently stored in the Main Stockroom.

Inventory quantities should never be negative.

---

### Inventory Transactions

Represents historical changes to inventory.

Potential fields:

- id
- item_id
- transaction_type
- quantity
- from_location_id
- to_location_id
- reason
- notes
- created_at

Initial transaction types:

- RECEIVE
- REMOVE
- TRANSFER
- ADJUSTMENT

Transactions provide an audit trail explaining how inventory reached its current state.

Historical transaction records should normally not be edited or deleted.

## Inventory Balance vs Transaction History

InventoryOS will maintain two related concepts.

### Current State

Inventory balances answer:

> How much inventory exists right now?

Example:

`USB-C Cable | Main Stockroom | 125`

### Historical State

Transactions answer:

> How did the inventory reach that quantity?

Example:

- RECEIVE +100
- RECEIVE +50
- REMOVE -10
- TRANSFER -15

Current quantity:

`125`

The current balance provides fast access to inventory quantities while transaction history preserves traceability.

## Cycle Counts

Cycle counting will require additional records.

### Cycle Counts

Represents a counting event.

Potential fields:

- id
- location_id
- status
- started_at
- completed_at
- created_at

Possible statuses:

- OPEN
- IN_PROGRESS
- COMPLETED
- CANCELLED

### Cycle Count Items

Represents individual items counted during a cycle count.

Potential fields:

- id
- cycle_count_id
- item_id
- expected_quantity
- counted_quantity
- variance

Variance is calculated as:

`counted_quantity - expected_quantity`

Example:

Expected: 100

Counted: 97

Variance: -3

Confirmed discrepancies may generate inventory adjustment transactions.

## Relationships

Conceptually:

Items have many Inventory Balances.

Locations have many Inventory Balances.

Each Inventory Balance belongs to one Item and one Location.

Items have many Inventory Transactions.

Locations may participate in many Inventory Transactions.

Locations have many Cycle Counts.

Cycle Counts have many Cycle Count Items.

Items may appear in many Cycle Count Items.

A simplified relationship model:

Items
↓
Inventory Balances
↑
Locations

Items
↓
Inventory Transactions
↑
Locations

Locations
↓
Cycle Counts
↓
Cycle Count Items
↑
Items

## Database Constraints

The database should eventually enforce important rules rather than relying entirely on application code.

Examples:

- SKU must be unique.
- Inventory quantity cannot be negative.
- Item/location inventory combinations must be unique.
- Foreign keys must reference valid records.
- Required fields cannot be null.

## Database Transactions

Inventory operations involving multiple database changes should use database transactions.

For example, transferring inventory requires:

1. Verify sufficient source inventory.
2. Decrease the source balance.
3. Increase the destination balance.
4. Create the transaction history record.

These operations should behave atomically:

Either all changes succeed or none are committed.

## Concurrency

InventoryOS should eventually account for multiple users modifying inventory at the same time.

Example:

System quantity: 10

User A attempts to remove 8.

User B attempts to remove 5 at nearly the same time.

The system must prevent both operations from succeeding if doing so would create negative inventory.

The exact concurrency strategy will be designed after learning PostgreSQL transactions, locking, and isolation concepts.

## Indexing

Indexes may eventually be useful for frequently searched fields such as:

- SKU
- Item name
- Transaction date
- Location
- Transaction type

Indexes should be added based on actual query requirements rather than automatically indexing every field.

## Future Database Entities

Later versions may introduce:

- Users
- Roles
- Organizations
- Vendors
- Purchase Orders
- Orders
- Lots
- Serial Numbers
- Pallets
- Bins
- Audit Events
- Notifications

These are intentionally outside the initial database scope.

## Design Status

This is a conceptual model, not the final PostgreSQL schema.

The actual schema will be created incrementally while learning:

- SQL
- Relational database design
- Primary and foreign keys
- Constraints
- Normalization
- Indexes
- Transactions
- Concurrency
- Database migrations

Understanding why the database is designed a particular way is more important than completing the schema quickly.