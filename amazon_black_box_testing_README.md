# Lab: Black Box Testing
## Student: Victoria Salomon

## Step 1: Select a Website

- **Website:** Amazon.com
- **URL:** https://www.amazon.com/
- **Features:** Product Search Bar and Shopping Cart
- **Method:** Black-box testing (without looking at source code)

## Step 2: Equivalence Partitioning — Amazon Search Bar

**Goal:** Check how Amazon responds to different search inputs.

### Equivalence Partitions

| Partition | Input Category | Valid / Invalid | Test Input | Expected Result |
| --- | --- | --- | --- | --- |
| P1 | Search with matching products | Valid | `USB-C cable` | Shows matching products |
| P2 | Search with no exact match | Valid | `xyz987nonexistent123` | Shows a no-results message (initial expectation) |
| P3 | Empty or spaces-only search | Invalid for a meaningful search | Empty input (`""`) | Returns to or stays on the homepage |

### Test Cases and Results

| Test ID | Partition | Input | Expected Result | Actual Result | Status |
| --- | --- | --- | --- | --- | --- |
| EP-01 | P1 | `USB-C cable` | Shows matching products | Amazon showed USB-C cables | Pass |
| EP-02 | P2 | `xyz987nonexistent123` | Shows a no-results message | Amazon showed one item: Xyzal allergy medicine | Did not match expectation |
| EP-03 | P3 | Empty input (`""`) | Returns to or stays on the homepage | Amazon showed the homepage | Pass |
| EP-04 | P3 | Three spaces (`"   "`) | Treats spaces like an empty search | Amazon showed the homepage | Pass |

### Observations

- **EP-02:** I expected no results, but Amazon showed Xyzal. It may have matched part of the search text. This does not necessarily mean the website has a bug.
- **EP-03 and EP-04:** Empty and spaces-only searches both showed the homepage. I counted these as invalid *search terms* because they have no meaningful keyword.
- **Coverage:** All 3 identified partitions were tested (3/3 = 100% of these partitions).

## Step 3: Decision Table Testing — Amazon Add to Cart

**Goal:** Check how Amazon handles product availability and quantity limits.

### Conditions

| ID | Condition | Values |
| --- | --- | --- |
| C1 | Is the product available to purchase? | True / False |
| C2 | Is the requested quantity allowed? | True / False |

A product can be available but still have a purchase limit. For example, a blouse may allow only one item per customer.

### Actions

| ID | Action |
| --- | --- |
| A1 | Add the requested quantity to the cart |
| A2 | Prevent adding the full requested quantity to the cart |

### Decision Table

| Conditions / Actions | Rule 1 | Rule 2 | Rule 3 |
| --- | --- | --- | --- |
| **C1: Product available?** | T | T | F |
| **C2: Requested quantity allowed?** | T | F | – |
| **A1: Add requested quantity to cart** | X | | |
| **A2: Prevent adding full requested quantity to cart** | | X | X |

**Legend:** T = True; F = False; – = Does not matter; X = Expected action.

### Test Cases

| Test ID | Conditions / Scenario | Expected Result | Actual Result | Status |
| --- | --- | --- | --- | --- |
| DT-01 | Product available; quantity allowed | Adds requested quantity to cart | Amazon added the product to the cart using the default quantity of 1 | PASS |
| DT-02 | Product available; requested quantity exceeds purchase limit | Amazon prevents selecting a quantity above the allowed limit | Some products had quantity selectors limited to 5, while others allowed up to 10 | PASS |
| DT-03 | Product unavailable | Does not allow adding the product | A "Currently Unavailable" product has no active Add to Cart button | PASS |

### Observations

- Amazon lets users click **Add to Cart** without first changing quantity; the default is usually 1.
- A blouse appeared to have a one-item limit, with no option to add more.
- I tested a product with a maximum quantity of 5. Amazon did not allow me to select more than 5 items
- The amount of limited items to add to cart varied depending on item, for other tested items allowed up to 10 items.
## Step 4: State Transition Testing

### Selected Feature

- **Website:** Amazon (https://www.amazon.com)
- **Feature:** Shopping Cart

The purpose of this test is to check how the shopping cart changes when a user adds products, removes products, or starts checkout.

### States

| State ID | State | Description |
|---|---|---|
| S1 | EMPTY | The cart has no products. |
| S2 | HAS ITEMS | The cart contains one or more products. |
| S3 | CHECKOUT | The user has entered the checkout process. |

### Events

| Event ID | Event | Description |
|---|---|---|
| E1 | Add Product | Add a product to the cart. |
| E2 | Remove Last Item | Remove the last remaining product from the cart. |
| E3 | Proceed to Checkout | Attempt to begin checkout. |
| E4 | Return to Cart | Return to the shopping cart from checkout. |

### State Transition Table

| Current State / Event | Add Product | Remove Last Item | Proceed to Checkout | Return to Cart |
|---|---|---|---|---|
| **EMPTY** | HAS ITEMS | — | — | — |
| **HAS ITEMS** | HAS ITEMS | EMPTY | CHECKOUT | — |
| **CHECKOUT** | — | — | — | HAS ITEMS |

**Legend:** `—` means the transition is not available or is outside the scope of this simplified cart model. For the invalid cases tested below, the expected state remains EMPTY.

### Valid and Invalid Transitions

| Transition ID | Start State | Event | Expected End State | Type |
|---|---|---|---|---|
| T1 | EMPTY | Add Product | HAS ITEMS | Valid |
| T2 | HAS ITEMS | Add Product | HAS ITEMS | Valid |
| T3 | HAS ITEMS | Remove Last Item | EMPTY | Valid |
| T4 | HAS ITEMS | Proceed to Checkout | CHECKOUT | Valid |
| T5 | CHECKOUT | Return to Cart | HAS ITEMS | Valid |
| T6 | EMPTY | Proceed to Checkout | EMPTY (checkout unavailable) | Invalid |
| T7 | EMPTY | Remove Last Item | EMPTY (nothing to remove) | Invalid |

### Test Cases

| | TC1 | TC2 | TC3 | TC4 | TC5 | TC6 | TC7 |
|---|---|---|---|---|---|---|---|
| **Start State** | EMPTY | HAS ITEMS | HAS ITEMS | HAS ITEMS | CHECKOUT | EMPTY | EMPTY |
| **Trigger** | Add Product | Add Product | Remove Last Item | Proceed to Checkout | Return to Cart | Proceed to Checkout | Remove Last Item |
| **Expected Action** | Add product to cart | Add another product | Remove final product | Start checkout | Show cart with items | Do not start checkout | No item can be removed |
| **Expected End State** | HAS ITEMS | HAS ITEMS | EMPTY | CHECKOUT | HAS ITEMS | EMPTY | EMPTY |
| **Transition Type** | Valid | Valid | Valid | Valid | Valid | Invalid | Invalid |

### Test Execution and Results

| Test ID | Test Scenario | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| TC1 | Add a product to an empty cart. | The cart contains the product. | An item was added to the cart | PASS |
| TC2 | Add another product while the cart already has an item. | Both products appear in the cart. | Added another item, shows 2 items in cart | PASS |
| TC3 | Remove the last remaining product from the cart. | The cart becomes empty. | Cart shows empty | PASS |
| TC4 | Select Proceed to Checkout with an item in the cart. | Checkout begins. | Proceeds to checkout | PASS|
| TC5 | Return to the cart after entering checkout. | The cart still contains the product. | The item still shows in cart | PASS |
| TC6 | Check whether checkout can start from an empty cart. | Checkout is unavailable; the cart stays empty. | No proceed to checkout option | PASS |
| TC7 | Check whether a product can be removed from an empty cart. | No product removal action is available; the cart stays empty. | No option to remove cart available. | PASS |

### Test Notes

The test cases cover five valid transitions and two invalid transitions. Adding a product when the cart already contains items is included because the cart remains in the HAS ITEMS state. The invalid tests check whether unavailable actions are blocked or missing

**Testing Note:** All tests were performed using a signed-in Amazon account to access the shopping cart and checkout features.

**Status key:** Pass = expected result observed; Fail = expected result not observed; 

