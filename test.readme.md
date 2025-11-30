# Property Management Accounting Setup (ERPNext)

This document outlines the essential accounts required for a property management system implemented in ERPNext. Each account includes its type, where it is used in the system, and why it is needed.

---

## 📘 Accounts Overview

| **Account Name**                 | **Account Type**         | **Role in the System**                                                        | **Why It's Needed**                                                                                 |
| -------------------------------- | ------------------------ | ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Rent Income**                  | Income                   | Credit side of the recurring Sales Invoice.                                   | To track pure rent revenue and separate it from other income streams such as sales.                 |
| **Service Charges Income**       | Income                   | Credit side for non-rent charges (e.g., utility or maintenance service fees). | To accurately measure ancillary revenue streams.                                                    |
| **Security Deposits Received**   | Liability                | Credit side of the security deposit Journal Entry.                            | Deposits must remain a liability until the lease ends since they are owed back to the tenant.       |
| **Temporary Deposit Control**    | Asset (or Current Asset) | Debit side of the deposit Journal Entry.                                      | Works as a clearing account to balance the liability until cash is actually received into the bank. |
| **Property Maintenance Expense** | Expense                  | Debit side of Purchase Invoices (repairs, cleaning, maintenance).             | Tracks operational costs per property; important for profitability reports.                         |
| **Bank / Cash Account**          | Asset                    | Debit side of the Payment Entry (standard).                                   | Where rent payments and deposits actually land.                                                     |

---

## 📌 Usage Notes

- These accounts help structure property management workflows such as:

  - Tenant billing (rent + service charges)
  - Deposit handling with proper liability control
  - Expense tracking for each property
  - Accurately reflecting cash flow in Bank/Cash accounts

- This setup is fully compatible with standard ERPNext accounting flows.

---

If you want, I can also prepare:
✔ A chart of accounts tree
✔ Journal Entry examples
✔ Sales Invoice sample with rent + service charges
✔ Deposit refund workflow

Just tell me!
