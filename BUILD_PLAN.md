# Design-on-Demand Marketplace Platform — Complete Build Plan (A to Z)

**Prepared by: Claude — Ready to Send to the Hostinger Website Generator**

---

## 1. Concept and Objective

A marketplace platform that connects customers who need designs with designers, with a financial system that hides the platform's commission from designers, balance and withdrawal wallets, and order intake through both the website and a unified WhatsApp number connected to the website.

**Language:** Fully Arabic with RTL direction (English later).

**Currency:** USD as the primary currency, with the possibility of displaying the equivalent in Yemeni Riyal later.

**Target audience:** Customers and designers in Yemen and the Arab region.

**Principle:** The customer pays the platform, and the platform holds the money in escrow, then transfers the designer's share after the customer accepts the work.

---

## 2. Roles and Permissions

### Customer

**Can:**

* Create an order
* Make payments
* Chat
* Receive the design
* Request revisions
* Leave a rating
* Open a dispute

**Cannot:**

* See other users' data
* See designers' prices

### Designer

**Can:**

* Accept orders
* Deliver work
* View earnings
* Request withdrawals

**Cannot:**

* See the customer's price
* See the platform commission
* See the customer's personal contact information

### Admin

**Can do everything:**

* Manage users
* Manage services
* Manage commissions
* Manage payments
* Manage withdrawals
* Manage disputes
* View reports

**Cannot:**

* Modify a financial transaction record after it has been recorded

---

## 3. Order Journey and Statuses

**Statuses:**

Draft, Awaiting Payment, Paid (Held in Escrow), Awaiting Designer Acceptance, In Progress, Delivered, Revision Requested, Completed, Cancelled, Disputed, Refunded.

### Normal Flow

1. The customer selects a service, enters the details, and uploads files through the website or WhatsApp.
2. The customer pays the price (for example, $5) → the order status changes to **Paid** and the amount is held in escrow.
3. The order is sent to suitable designers → the first designer to accept gets the order.
4. The designer completes the work and delivers the files through the website (or WhatsApp, linked to the order).
5. The customer accepts the work or requests a revision. The number of allowed revisions is determined by the admin for each service.
6. Upon acceptance — or automatically after 48 hours from delivery if there is no response — **$3.50** moves from the pending balance to the available balance.
7. The designer requests a withdrawal to their wallet.

**Execution deadline:** Each service has its own deadline, for example 24 hours. If the designer exceeds the deadline, the order is automatically returned to the distribution system and a delay is recorded against the designer.

---

## 4. Financial System — Most Important Section

### Golden Rule

The price breakdown must be saved inside the order at the moment the order is created and must not be recalculated later.

Changing the commission rate later must **not affect old orders**.

| Service              | Customer Price | Platform Commission (30%) | Designer Earnings |
| -------------------- | -------------: | ------------------------: | ----------------: |
| Logo                 |             $5 |                     $1.50 |             $3.50 |
| Post                 |             $3 |                     $0.90 |             $2.10 |
| Business Card        |             $7 |                     $2.10 |             $4.90 |
| Advertisement Design |            $10 |                     $3.00 |             $7.00 |
| Visual Identity      |            $30 |                     $9.00 |            $21.00 |

Prices and commissions are managed from the admin dashboard and must not be hard-coded.

### Designer Balance Types

* **Pending:** Orders currently in progress or awaiting customer acceptance.
* **Available:** Funds that can be withdrawn.
* **Withdrawn:** Historical withdrawn funds.

### Strict Rules

* Every financial transaction must be recorded in an immutable **Ledger**. Transactions must not be edited or deleted. Corrections must be made through a new reversing transaction.
* The balance must be calculated from the ledger and must never be manually modified as a standalone number.
* Negative balances are not allowed. A designer cannot withdraw more than the available balance.
* The minimum withdrawal amount (suggested: $10) and the maximum daily withdrawal amount are controlled by the admin.
* The designer must never see the customer's price or the platform commission. This must be enforced through database and API permissions, not merely by hiding the fields in the interface.
* Amounts must be rounded to the nearest cent and stored as integer cents — for example, **500 = $5** — to prevent decimal/floating-point errors.

### Refunds

The pending balance is cancelled and a refund transaction is recorded for the customer.

---

## 5. Website Pages

### Public Pages

* Home
* Services and Prices
* Portfolio
* How the Platform Works
* Join as a Designer
* FAQ
* Terms, Privacy Policy, and Refund Policy
* Contact Us
* Login and Registration

### Customer Dashboard

* My Orders with statuses
* New Order:
  1. Service
  2. Details
  3. Files
  4. Payment
* Order Page:
  * Chat
  * Files
  * Revision request
  * Acceptance
* Wallet and invoices
* Profile
* Notifications

### Designer Dashboard

* Available new orders with an **Accept** button
* Orders in progress
* Completed orders
* Earnings:
  * Pending
  * Available
  * Withdrawn
* Withdrawals and withdrawal methods
* Profile and portfolio
* Ratings
* Availability status:
  * Available Now
  * Busy

### Admin Dashboard

* Statistics
* Users
* Designers:
  * Approval
  * Suspension
* Services, prices, and commissions
* Orders
* Payments:
  * Receipt verification
* Withdrawals:
  * Processing
  * Uploading proof
* Disputes
* Financial ledger
* WhatsApp settings
* Reports

---

## 6. Database

### `users`
* ID
* Name
* Phone number (unique)
* Email
* Role
* Status
* Creation date

### `designer_profiles`
* User
* Bio
* Skills
* Availability
* Rating
* Number of completed orders
* Maximum concurrent orders

### `services`
* Name
* Description
* Customer price in cents
* Commission percentage
* Number of revisions
* Delivery deadline
* Active/inactive status

### `orders`
* Customer
* Designer
* Service
* Details
* Source (Website/WhatsApp)
* Status
* Customer price
* Commission
* Designer earnings captured at creation time
* Stage timestamps

### `order_files`
* Order
* Uploader
* Type (Reference/Delivery)
* URL

### `messages`
* Order
* Sender
* Text
* File
* Channel (Website/WhatsApp)
* Timestamp

### `wallets`
* User
* Pending balance
* Available balance (updated only from the ledger)

### `ledger`
* Wallet
* Order
* Transaction type:
  * Payment
  * Hold
  * Release
  * Withdrawal
  * Refund
  * Commission
* Amount
* Balance after transaction
* Timestamp
* Immutable

### `payments`
* Customer
* Order
* Payment method
* Amount
* Transaction number
* Receipt
* Status:
  * Pending
  * Confirmed
  * Rejected
* Confirmed by

### `payout_methods`
* Designer
* Wallet or bank type
* Account name
* Account number

### `withdrawals`
* Designer
* Amount
* Method
* Status:
  * Pending
  * Processing
  * Completed
  * Rejected
* Transfer proof

### Additional Tables
* `reviews`
* `disputes`
* `notifications`
* `settings`
  * Default commission
  * Minimum withdrawal
  * Automatic acceptance period
* `audit_logs`
  * Who performed what and when

---

## 7. Payments and Withdrawals in Yemen

A separate **Payment Adapter** layer should be built so that payment methods can be added or removed without affecting the rest of the system.

Supported methods may include:
* Manual transfer
* Wallet A
* Wallet B
* Bank transfer

### Phase One — Practical and Secure

The platform displays receiving accounts, such as the platform's wallet numbers.
The customer transfers the payment and uploads a screenshot of the receipt and the transaction number.
The admin or supervisor verifies and confirms the payment → the order changes to **Paid**.
Duplicate prevention: the same transaction number cannot be accepted twice.

### Withdrawals

The designer registers a withdrawal method and requests an amount from the available balance.
The amount is immediately deducted from the available balance and the withdrawal appears as **Under Review**.
The admin performs the transfer and uploads proof of payment → the status becomes **Completed**.
If the withdrawal is rejected, the amount is returned through a reversing transaction.

### Later Phase

Direct automatic integration.

**Important warning:** Do not assume that any Yemeni wallet provides a public merchant API. Each provider must be officially contacted to confirm whether it offers an API or Merchant service before promising full automation.

---

## 8. WhatsApp System

Use only the official **WhatsApp Business Platform (Cloud API)** with one unified number.
Avoid unofficial solutions because they may expose the number to blocking or suspension.
Identify the customer using their phone number.
If the customer does not have an account, create an initial account with their consent.

### WhatsApp → Website

The customer sends:
> "I want a logo design."

The bot asks a short series of questions:
* Design type
* Details
* Files

Then it automatically creates an order with the status **Awaiting Payment** and sends the payment link.
Incoming files and messages are linked to the open order and displayed on the order page.

### Website → WhatsApp

Automatic notifications are sent to the customer, such as:
* Payment confirmation
* Designer acceptance
* Delivery
* Revision request

The designer primarily works inside the website.
Messages sent by the customer are delivered to the designer through the order system.

### WhatsApp Restrictions

Messages initiated by the platform outside the 24-hour customer-service window require approved WhatsApp Templates.
There are costs depending on the conversation type.
The customer's **Opt-in** consent must be obtained before sending messages.
The customer's phone number must not be exposed to the designer in order to prevent off-platform transactions.

---

## 9. Automatic Order Distribution

When an order is paid, suitable designers are filtered according to:
* They offer the requested service
* They are currently available
* They have a good rating
* They have good delivery speed
* They have not exceeded their current order limit

A notification is sent to a group of designers, for example **5 designers**.
The first designer to accept gets the order, and the order disappears from the other designers' available-order lists.
If nobody accepts within **15 minutes**, the group is expanded.
If no designer accepts after further expansion, the admin is notified.

---

## 10. Security, Disputes, and Legal Requirements

### Authentication
Login should use a phone number with OTP, with optional email.
Passwords must be securely hashed, and repeated login attempts must be protected against abuse.

### Database-Level Permissions
Use **Row Level Security (RLS)** so that each user can access only their own data.
The designer must not have access to customer-price or commission fields.

### Financial Operations
Financial operations such as:
* Holding funds
* Releasing funds
* Withdrawals

must be executed through **Server Functions**, not directly from the browser.
Every sensitive action must be recorded in `audit_logs`.

### File Uploads
File uploads must have:
* Allowed file types
* Maximum file sizes
* Security scanning

### Disputes
Either party can open a dispute before final acceptance.
The admin reviews:
* Conversation
* Files
* Order history

The admin then decides whether to:
* Complete the order
* Issue a full refund
* Split the amount

### Legal Pages
The platform must include:
* Terms and Conditions
* Privacy Policy
* Refund Policy
* Design ownership clause
* Off-platform transaction prohibition clause

Design ownership transfers to the customer after payment and acceptance.
Regular database backups must be performed.
Consult a local legal professional regarding:
* Commercial licensing
* Financial transactions
* Regulatory requirements

---

## 11. Notes Regarding the Hostinger Generator (Horizons) and Its Limitations

These tools can build interfaces and logic quickly from text descriptions, but sensitive financial systems require additional caution.
The latest capabilities of the tool have not been verified, so its documentation should be reviewed to determine whether it supports:
* A database with permissions, such as Supabase
* Server Functions
* Webhooks, which are required for WhatsApp
* Phone-number/OTP authentication

Build the platform in stages.
Send one prompt at a time and test each stage before moving to the next.
Explicitly instruct the generator that:
* Financial calculations must run on the server
* Prices must be stored in cents
* The ledger must be immutable

If the tool does not support advanced components such as WhatsApp integration and automatic order distribution, use it for the interface and database, then add a separate backend service later.
Keep a file documenting every prompt that was sent and the resulting implementation.

---

# 12. Development Stages and Ready-to-Use Prompts

## Prompt 0 — General Context

> Build an Arabic design marketplace platform with a complete RTL interface that connects customers with designers.
>
> There are three roles: Customer, Designer, and Admin, and each role has its own dashboard.
>
> All amounts are in USD and must be stored in cents.
>
> When an order is created, store the following inside the order:
>
> * Customer price
> * Platform commission
> * Designer earnings
>
> The designer must never see the customer price or platform commission.
>
> Do not build anything yet. Only confirm that you understand the requirements and wait for the next stages.

---

## Stage 1 — Foundation

> Create registration and login using phone number with OTP and email.
>
> Create three user roles with role-based page protection.
>
> Create a professional Arabic homepage and a services/pricing page.
>
> Create a `services` table containing:
>
> * Name
> * Description
> * Price in cents
> * Commission percentage
> * Number of revisions
> * Delivery deadline
> * Active status
>
> Add an admin dashboard for creating and editing services.

---

## Stage 2 — Orders

> Create the order system.
>
> The customer selects a service, enters the details, and uploads files.
>
> The order is created with the status **Awaiting Payment**.
>
> Store a snapshot of the price, commission, and designer earnings inside the order.
>
> Add an order page with the following statuses:
>
> * Paid
> * Awaiting Designer Acceptance
> * In Progress
> * Delivered
> * Revision Requested
> * Completed
> * Cancelled
> * Disputed
> * Refunded
>
> Add an internal conversation system inside the order and allow delivery files to be uploaded.

---

## Stage 3 — Payment, Wallet, and Ledger

> Create the `wallets`, `ledger`, and `payments` tables.
>
> Payments are manual.
>
> The website displays receiving accounts that can be edited by the admin.
>
> The customer uploads a payment receipt and transaction number.
>
> Duplicate transaction numbers must not be accepted.
>
> The admin confirms the payment, changing the order to **Paid** and placing the amount on hold.
>
> The ledger must be immutable and must never be edited or deleted.
>
> Balances must be calculated from the ledger.
>
> When the customer accepts the work, or automatically after 48 hours, the designer's earnings move from pending to available.
>
> Implement all of this on the server.

---

## Stage 4 — Designer Dashboard and Withdrawals

> Create the designer dashboard:
>
> * New available orders with an Accept button
> * First designer to accept gets the order
> * Orders in progress
> * Completed orders
> * Earnings:
>
>   * Pending
>   * Available
>   * Withdrawn
> * Withdrawal methods:
>
>   * Wallet type
>   * Account name
>   * Account number
> * Withdrawal request with a minimum amount controlled by the admin
>
> When a withdrawal is requested, immediately deduct the amount from the available balance.
>
> The admin processes the transfer and uploads proof of payment.
>
> If the withdrawal is rejected, return the amount through a reversing ledger transaction.

---

## Stage 5 — Admin Dashboard and Reports

> Complete the admin dashboard:
>
> * Users
> * Designers
>
>   * Approval
>   * Suspension
> * Orders
> * Payments
> * Withdrawals
> * Disputes
> * Financial ledger
> * `audit_logs`
> * System settings:
>
>   * Default commission
>   * Minimum withdrawal
>   * Automatic acceptance period
> * Profit and commission reports by date range

---

## Stage 6 — Distribution, Ratings, and Notifications

> Add automatic order distribution to suitable designers.
>
> Filter designers according to:
>
> * They offer the requested service
> * They are available
> * Good rating
> * Good delivery speed
> * They have not exceeded their order limit
>
> The first designer to accept gets the order.
>
> Expand the designer group after 15 minutes if nobody accepts.
>
> Add designer ratings after order completion.
>
> Add in-platform notifications.

---

## Stage 7 — WhatsApp

> Connect the website to the official WhatsApp Business Cloud API.
>
> Implement:
>
> * Incoming messages through a Webhook
> * Customer identification using their phone number
> * A bot that collects order information
> * Automatic order creation
> * Payment link delivery
> * Linking incoming files to the order
> * Automatic customer notifications:
>
>   * Payment confirmation
>   * Designer acceptance
>   * Delivery
>
> Use approved WhatsApp templates where required.
>
> The customer's phone number must never be visible to the designer.

---

# 13. Pre-Launch Checklist

* Test a complete order from start to withdrawal using test accounts.
* Make sure the designer cannot access the customer price or commission from any page or API response.
* Attempt to manipulate the system:
  * Withdraw more than the available balance
  * Reuse a transaction number
  * Modify another user's order
* Test:
  * Refunds
  * Order cancellation
  * Disputes
* Reconcile the total ledger balance against the total wallet balances.
* Test the website on mobile devices because most users will access it from mobile.
* Verify RTL support.
* Test page performance on slow internet connections.
* Review:
  * Terms and Conditions
  * Privacy Policy
* Enable:
  * HTTPS
  * Custom domain
  * Regular backups
* Start with a small pilot:
  **10 customers + 5 designers**
  before launching publicly.

---

# 14. Decisions Required

The following decisions are required before development:

1. Platform name, logo, and colors.
2. Final commission percentage — example: **30%**.
3. Initial list of services, including:
   * Prices
   * Delivery deadlines
   * Number of allowed revisions
4. Wallets and accounts that will receive customer payments, and the withdrawal methods available to designers.
5. Minimum withdrawal amount and automatic acceptance period.
6. Whether designers will be accepted through prior approval — recommended — or through open registration.
7. Dedicated WhatsApp Business number for the platform.
