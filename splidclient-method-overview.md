# SplidClient Public API (Full)

## 🧾 Method Overview

| Class                        | Method/Field                               | Description                                                   |
|-----------------------------|--------------------------------------------|---------------------------------------------------------------|
| `SplidClient#group`         | `getByInviteCode(code)`                    | Fetch group metadata using a public invite code               |
| `SplidClient#group`         | `create(name, members, options?)`          | Create a new group with a name and optional member list       |
| `SplidClient#groupInfo`     | `getOneByGroup(groupId)`                   | Retrieve metadata for a group (e.g., currency info, wallpaper)|
| `SplidClient#person`        | `getAllByGroup(groupId)`                   | Retrieve list of all group members                            |
| `SplidClient#entry`         | `getAllByGroup(groupId)`                   | Retrieve all expenses and payments in a group                 |
| `SplidClient#entry.expense` | `create(entryData)`                        | Create a new expense in a group                               |
| `SplidClient#entry.payment` | `create(paymentData)`                      | Create a payment record between group members                 |
| `SplidClient#SplidClient`   | `getBalance(members, entries, groupInfo?)` | Compute net balance per member based on group entries         |
| `SplidClient#SplidClient`   | `getSuggestedPayments(balanceMap)`         | Determine optimal repayments from a set of member balances    |

---

## 🔍 Method Details

### `SplidClient#group.getByInviteCode(code)`
- `code`: `String` — Invite code in the form `'ABC DEF GHI'`

---

### `SplidClient#group.create(name, members, options?)`
- `name`: `String` — Name of the group  
- `members`: `Array` — Strings or objects `{ name, initials }`  
- `options`: `Object` *(optional)* — Currency, categories, wallpaper, etc.

---

### `SplidClient#groupInfo.getOneByGroup(groupId)`
- `groupId`: `String` — ID of the group

---

### `SplidClient#person.getAllByGroup(groupId)`
- `groupId`: `String` — ID of the group

---

### `SplidClient#entry.getAllByGroup(groupId)`
- `groupId`: `String` — ID of the group

---

### `SplidClient#entry.expense.create(entryData)`
- `entryData`: `Object` — Must include title, primaryPayer, items, profiteers, etc.

---

### `SplidClient#entry.payment.create(paymentData)`
- `paymentData`: `Object` — Must include amount, payer, and profiteer (receiver)

---

### `SplidClient#getBalance(members, entries, groupInfo?)`
- `members`: `Array` — Member objects with `GlobalId`  
- `entries`: `Array` — Expenses + payments  
- `groupInfo`: `Object` *(optional)* — Group-level currency info

---

### `SplidClient#getSuggestedPayments(balanceMap)`
- `balanceMap`: `Object` — Output from `getBalance`
