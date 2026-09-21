# Foodics OAuth Scopes Specification — SareeTap Platform

**Target System:** SareeTap Multi-Tenant Platform (`sareetap.com`)  
**Provider:** Foodics API v5 (`api.foodics.com`)  
**Reference:** [Foodics Official Scopes Matrix](https://apidocs.foodics.com/core/scopes.html)  
**Status:** Authoritative Application Permission Specification  

---

## 1. Complete Scope String (Copy & Paste)

Use this exact space-separated string when registering the SareeTap application in the **Foodics Developer Console / Partner Request Form** and constructing the OAuth authorization URL:

```text
general.read tables.write orders.list orders.get orders.write customers.list customers.get customers.write customers.loyalty.read customers.loyalty.write customers.accounts.read coupons.read operations.read inventory.transactions.read
```

> **Optional Bidirectional Menu Editing:**  
> If the restaurant wishes to create, edit, or delete Foodics menu items, categories, and modifiers directly from SareeTap (rather than Foodics remaining the single source of truth), append `menu.write` to the string.

---

## 2. Scope Matrix & Operational Capabilities

| # | Foodics Scope | Entity Affected | Permissions & SareeTap Operational Powers |
| :---: | :--- | :--- | :--- |
| **1** | **`general.read`** | **Tables, Sections, Products, Modifiers, Categories, Taxes, Branches, Charges, Discounts, Payment Methods** | **Core Foundation & Full Catalog Sync:**<br>• Reads all menu products, categories, descriptions, images, and bilingual names (`en`/`ar`).<br>• Reads active VAT rates, tax groups, and service charges.<br>• Reads restaurant tables, floor sections, and capacity.<br>• Reads branches, payment tender types, and store settings. |
| **2** | **`tables.write`** | **Tables & Sections** | **Floor Plan Management:**<br>• Allows restaurant managers to create, edit, rename, and arrange dining tables and floor sections in Foodics directly from the SareeTap dashboard. |
| **3** | **`orders.write`** | **Orders** | **Kitchen & Register Injection:**<br>• Automatically injects guest QR table orders directly into Foodics kitchen screens (KDS), thermal prep station printers, and cashier registers (`POST /v5/orders`).<br>• Updates orders and applies payments/discounts. |
| **4** | **`orders.list`** | **Orders** | **Kitchen Tracking & Sales Reporting:**<br>• Tracks live kitchen preparation status (`PREPARING`, `READY`, `SERVED`) to show real-time progress bars to diners on their phones.<br>• Powers sales, revenue, AOV, and product mix reporting. |
| **5** | **`orders.get`** | **Orders** | **Verification & Idempotency:**<br>• Fetches individual order details to prevent duplicate ticket injection and verify receipt figures. |
| **6** | **`customers.list`** | **Customers** | **Diner Recognition:**<br>• Automatically identifies returning diners by mobile number upon scanning the table QR code. |
| **7** | **`customers.get`** | **Customers** | **Customer CRM & Preferences:**<br>• Retrieves dining history, total spend, and past order preferences. |
| **8** | **`customers.write`** | **Customers** | **Auto-Enrollment:**<br>• Automatically registers new diners into the restaurant's Foodics CRM from mobile checkouts. |
| **9** | **`customers.loyalty.read`** | **Loyalty** | **Table-side Points Balance:**<br>• Diners view their live Foodics loyalty points balance on their mobile phone while dining. |
| **10** | **`customers.loyalty.write`** | **Loyalty** | **Point Redemption & Earning:**<br>• Allows diners to earn points on table orders or redeem reward discounts at checkout without staff assistance. |
| **11** | **`customers.accounts.read`** | **House Accounts** | **Corporate & VIP Billing:**<br>• Allows VIP diners and corporate accounts to charge their table orders directly to their house account balance in Foodics. |
| **12** | **`coupons.read`** | **Coupons** | **Promo Codes:**<br>• Validates and applies Foodics promotional discount vouchers directly on the SareeTap checkout screen. |
| **13** | **`operations.read`** | **Shifts, Tills, Business Days** | **Shift Guardrails & Register Reconciliation:**<br>• Checks if the branch and kitchen are currently open before accepting orders (eliminating "ghost orders" sent to closed kitchens).<br>• Powers daily shift and till reconciliation reports. |
| **14** | **`inventory.transactions.read`** | **Inventory Levels & Counts** | **Live 86ing & Food Cost Analytics:**<br>• Reads real-time stock levels to automatically disable dishes (86ing) when ingredients run out before an order is placed.<br>• Tracks ingredient consumption and wastage. |

---

## 3. Frequently Asked Architecture Questions

### Why is there no "kitchen" scope?
In Foodics, the Kitchen Display System (KDS) and kitchen thermal printers are **not separate API entities**. They are driven entirely by **`orders.write`** (which dispatches the order to prep stations) and **`orders.list` / `orders.get`** (which monitors the kitchen preparation lifecycle). Hardware devices are read via **`general.read`** (`/v5/devices`).

### Why is there no "reports" scope?
Foodics does not expose pre-computed report endpoints. Instead, all financial, product, and operational analytics are calculated programmatically using **`orders.list`** (revenue, tax, payment mix, top products) and **`operations.read`** (shifts, tills, cash drawers).

### How are tables read without a `tables.read` scope?
In the official Foodics permission structure, reading tables and floor sections is bundled into **`general.read`** (`GET /v5/tables` and `GET /v5/sections`). **`tables.write`** is only required if SareeTap creates or edits tables in Foodics.

---

## 4. OAuth 2.0 URL Example Construction

```typescript
const foodicsAuthUrl = new URL("https://console.foodics.com/oauth/authorize");
foodicsAuthUrl.searchParams.set("client_id", process.env.FOODICS_CLIENT_ID!);
foodicsAuthUrl.searchParams.set("redirect_uri", "https://sareetap.com/api/restaurant/integrations/foodics/oauth/callback");
foodicsAuthUrl.searchParams.set("response_type", "code");
foodicsAuthUrl.searchParams.set("state", oauthCsrfState);
foodicsAuthUrl.searchParams.set(
  "scope",
  [
    "general.read",
    "tables.write",
    "orders.list",
    "orders.get",
    "orders.write",
    "customers.list",
    "customers.get",
    "customers.write",
    "customers.loyalty.read",
    "customers.loyalty.write",
    "customers.accounts.read",
    "coupons.read",
    "operations.read",
    "inventory.transactions.read",
  ].join(" ")
);
```
