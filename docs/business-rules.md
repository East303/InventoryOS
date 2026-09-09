# Business Rules

Business rules define how InventoryOS must behave to keep inventory data accurate, consistent, and traceable.

## Items

- Every SKU must be unique.
- Item names do not need to be unique.
- Items with historical inventory activity should be deactivated rather than permanently deleted.
- Reorder levels cannot be negative.

## Locations

- Inventory must belong to a valid location.
- Locations may represent warehouses, stores, stockrooms, offices, vehicles, freezers, workshops, or other physical storage areas.
- A location containing inventory cannot be deactivated until its remaining inventory has been transferred or removed.

## Inventory

- Inventory quantities cannot be negative.
- Each item should have only one current inventory balance per location.
- Inventory quantities should only change through a recorded inventory operation.

## Receiving

- Receiving quantities must be greater than zero.
- The item must exist and be active.
- The destination location must exist and be active.
- A successful receipt must increase inventory at the destination location.
- A successful receipt must create a RECEIVE transaction.

## Removing Inventory

- Removal quantities must be greater than zero.
- A user cannot remove more inventory than is currently available at the selected location.
- A removal reason is required.
- A successful removal must decrease inventory at the selected location.
- A successful removal must create a REMOVE transaction.

Initial removal reasons include:

- Used
- Sold
- Damaged
- Lost
- Expired
- Adjustment
- Other

## Transfers

- Transfer quantities must be greater than zero.
- Source and destination locations must be different.
- A user cannot transfer more inventory than is available at the source location.
- A successful transfer decreases inventory at the source location.
- A successful transfer increases inventory at the destination location.
- A transfer must not change the total quantity of the item across all locations.
- A successful transfer must create a TRANSFER transaction.

## Atomic Inventory Operations

Inventory operations that require multiple database changes must succeed completely or fail completely.

For example, during a transfer:

1. Verify sufficient inventory exists.
2. Subtract inventory from the source location.
3. Add inventory to the destination location.
4. Record the transaction.

If any required step fails, none of the changes should be permanently saved.

## Inventory Adjustments

- Adjustments must not silently overwrite inventory quantities.
- An adjustment must preserve a record of the correction.
- The system should record the previous quantity, corrected quantity, and variance.
- An adjustment reason is required.
- A successful adjustment must create an ADJUSTMENT transaction.

## Cycle Counts

A cycle count compares the quantity recorded by InventoryOS with a physical inventory count.

Variance is calculated as:

`Variance = Counted Quantity - Expected Quantity`

For example:

`97 counted - 100 expected = -3 variance`

- Counted quantities cannot be negative.
- Completed cycle counts should be preserved as historical records.
- Confirmed discrepancies may generate inventory adjustment transactions.
- Corrections resulting from cycle counts should remain traceable to the count that caused them.

## Transaction History

Initial transaction types are:

- RECEIVE
- REMOVE
- TRANSFER
- ADJUSTMENT

Historical transactions should normally be immutable.

If an inventory mistake is discovered later, the system should record a correction rather than rewriting the original transaction.

## Data Integrity

InventoryOS should always attempt to answer:

1. What changed?
2. How much changed?
3. Where did it change?
4. Why did it change?
5. When did it change?

When user accounts are introduced, the system should also be able to answer:

6. Who performed the action?

Maintaining accurate and explainable inventory data takes priority over convenience.