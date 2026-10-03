# What-next-vision-motor
# 🚗 Vehicle Order & Inventory Management System

A Salesforce Apex-based **Vehicle Order & Inventory Management System** that automates vehicle stock validation, stock deduction, pending order processing, and scheduled batch execution.

## 📌 Project Overview

This project is designed to manage vehicle orders and automatically maintain vehicle inventory in Salesforce.

The system ensures that:

* Orders cannot be placed when a vehicle is out of stock.
* Vehicle stock is automatically reduced when an order is confirmed.
* Pending orders can be processed in bulk using Batch Apex.
* Batch processing can be automated using a Scheduled Apex class.
* The solution follows a Trigger + Handler architecture for better code organization.

---

## 🛠️ Technologies Used

* **Salesforce**
* **Apex**
* **Apex Triggers**
* **Trigger Handler Pattern**
* **Batch Apex**
* **Scheduled Apex**
* **SOQL**
* **Custom Objects & Custom Fields**

---

## 📂 Project Structure

```text
Vehicle Order & Inventory Management System
│
├── VehicleOrderTriggerHandler.cls
├── VehicleOrderTrigger.trigger
├── VehicleOrderBatch.cls
└── VehicleOrderBatchScheduler.cls
```

### 1. VehicleOrderTriggerHandler

Contains the main business logic for vehicle orders.

#### Before Insert / Before Update

Checks the vehicle's available stock.

If stock is `0` or less, the order is prevented using:

```apex
order.addError('This vehicle is out of stock. Order cannot be placed.');
```

#### After Insert / After Update

When an order has the status:

```text
Confirmed
```

the corresponding vehicle stock is reduced by `1`.

---

### 2. VehicleOrderTrigger

The trigger executes the handler during:

* Before Insert
* Before Update
* After Insert
* After Update

```apex
trigger VehicleOrderTrigger on Vehicle_Order__c 
(before insert, before update, after insert, after update) {

    VehicleOrderTriggerHandler.handleTrigger(
        Trigger.new,
        Trigger.oldMap,
        Trigger.isBefore,
        Trigger.isAfter,
        Trigger.isInsert,
        Trigger.isUpdate
    );
}
```

Using a separate handler class keeps the trigger clean and makes the business logic easier to maintain.

---

### 3. VehicleOrderBatch

The Batch Apex class processes all orders having:

```text
Status = Pending
```

The batch:

1. Retrieves pending vehicle orders.
2. Gets the related vehicle records.
3. Checks available stock.
4. Changes eligible orders from `Pending` to `Confirmed`.
5. Decreases the vehicle stock by `1`.

Example query:

```apex
SELECT Id, Status__c, Vehicle__c
FROM Vehicle_Order__c
WHERE Status__c = 'Pending'
```

The batch size used by the scheduler is:

```apex
Database.executeBatch(batchJob, 50);
```

---

### 4. VehicleOrderBatchScheduler

The Scheduled Apex class automatically starts the batch job.

```apex
VehicleOrderBatch batchJob = new VehicleOrderBatch();
Database.executeBatch(batchJob, 50);
```

This allows pending orders to be processed automatically according to the configured Salesforce schedule.

---

## 🔄 System Flow

```text
                Vehicle Order Created
                         │
                         ▼
                VehicleOrderTrigger
                         │
                         ▼
            VehicleOrderTriggerHandler
                         │
              ┌──────────┴──────────┐
              │                     │
         Before Insert/Update    After Insert/Update
              │                     │
              ▼                     ▼
        Check Vehicle Stock     Check Status
              │                     │
        ┌─────┴─────┐               │
        │           │               │
     Stock > 0   Stock <= 0     Confirmed?
        │           │               │
        ▼           ▼          ┌────┴────┐
      Allow       Block         Yes       No
      Order       Order          │         │
                                 ▼         ▼
                            Reduce Stock  No Change


             Scheduled Apex
                   │
                   ▼
          VehicleOrderBatch
                   │
                   ▼
          Find Pending Orders
                   │
                   ▼
          Check Vehicle Stock
                   │
             ┌─────┴─────┐
             │           │
          Stock > 0   Stock = 0
             │           │
             ▼           ▼
       Confirm Order   Remain Pending
       Reduce Stock
```

---

## 🧩 Salesforce Components

### Custom Object

**Vehicle_Order__c**

Important fields:

| Field        | Purpose                |
| ------------ | ---------------------- |
| `Vehicle__c` | References the vehicle |
| `Status__c`  | Stores order status    |

### Vehicle Object

**Vehicle__c**

Important field:

| Field               | Purpose                           |
| ------------------- | --------------------------------- |
| `Stock_Quantity__c` | Stores available vehicle quantity |

---

## ✅ Key Features

### Stock Validation

Prevents users from placing an order when the selected vehicle has no available stock.

### Automatic Stock Deduction

Confirmed orders automatically decrease the corresponding vehicle's stock quantity.

### Bulk Order Processing

Batch Apex processes multiple pending orders efficiently.

### Scheduled Automation

Scheduled Apex can automatically execute the batch process at a configured time.

### Trigger Handler Architecture

Business logic is separated from the trigger, improving maintainability and scalability.

---

## 💡 Example Scenario

Suppose a vehicle has:

```text
Vehicle: Toyota Camry
Stock Quantity: 3
```

When a confirmed order is placed:

```text
Initial Stock  →  3
Order Confirmed
Updated Stock  →  2
```

After three confirmed orders:

```text
Stock → 0
```

If another order is attempted, the system prevents the order because the vehicle is out of stock.

---

## 🚀 Deployment

### Step 1: Create Salesforce Components

Create the required:

* `Vehicle__c` custom object
* `Vehicle_Order__c` custom object
* Required fields
* Relationship between Vehicle Order and Vehicle

### Step 2: Deploy Apex Classes

Deploy:

```text
VehicleOrderTriggerHandler.cls
VehicleOrderBatch.cls
VehicleOrderBatchScheduler.cls
```

### Step 3: Deploy Trigger

Deploy:

```text
VehicleOrderTrigger.trigger
```

### Step 4: Schedule the Batch

The scheduler can be configured using Salesforce:

**Setup → Apex Classes → Schedule Apex**

---

## 🧪 Testing

The following scenarios should be tested:

* Create an order when stock is available.
* Create an order when stock is `0`.
* Confirm an order and verify stock reduction.
* Create multiple pending orders.
* Execute Batch Apex and verify pending orders.
* Verify that orders remain pending when stock is unavailable.
* Schedule the batch and verify automatic execution.

---

## 📈 Future Enhancements

Possible improvements include:

* Preventing duplicate stock deduction when an order is updated.
* Supporting order cancellation and stock restoration.
* Adding Apex test classes with high code coverage.
* Adding error handling for failed updates.
* Implementing Salesforce Flow/LWC for the user interface.
* Adding notifications when vehicle stock reaches a low level.
* Adding reports and dashboards for vehicle inventory and orders.

---

## 👨‍💻 Author

**Yokeshraj**

B.Tech Information Technology

---

## 📄 License

This project is created for **educational and Salesforce development purposes**.
