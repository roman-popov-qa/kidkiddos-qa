# KidKiddos Books — Manual QA Project

Manual testing of the online bookstore [kidkiddos.com](https://kidkiddos.com): product catalog, cart, currency, and checkout.

| | |
|---|---|
| **Tools** | Jira, Xray, Cucumber (Gherkin) |
| **Environment** | macOS Tahoe, Safari 26.4 (21624.1.16.11.4) |
| **Test cases** | 30 (25 Passed, 5 Failed) |
| **Bugs found** | 5 (1 Highest, 4 Medium) |

**Test coverage:** filtering books by language · book formats (Paperback, Hardcover, .pdf, .epub, .mobi) · adding and removing books in the cart · currency change (CAD, USD, EUR) · total price update · checkout form validation · payment fields · boundary values (0.00 CAD, quantity 1,000,000)

All personal data in the test steps is test data.

---

## Selected test cases

Three test cases that show different testing techniques:

| Test | Scenario | Technique | Result | Bug |
|---|---|---|---|---|
| ROMAN-33 | User cannot add 1000000 books to cart | Boundary values | Failed | [ROMAN-38](#roman-38-quantity-field-has-no-max-limit-1000000-books-can-be-added-to-cart) (Highest) |
| ROMAN-24 | Checkout does not accept a numeric First name | Negative testing, form validation | Failed | [ROMAN-34](#roman-34-first-name-field-accepts-numeric-value-1) (Medium) |
| ROMAN-32 | Free eBook (0.00 CAD) can be ordered | Edge case, price | Passed | — |

---

## Test scenarios (Gherkin)

```gherkin
Feature: KidKiddos Books - cart and checkout

  @ROMAN-24 @bug:ROMAN-34
  Scenario: Checkout does not accept a numeric First name
    Given user is on the payment page
    When user enters First name as "1"
    And user fills all other required fields with valid data
    And user clicks "Pay now" button
    Then error message "Enter a valid first name" should be displayed
    And user should stay on the payment page

  @ROMAN-32
  Scenario: Free eBook (0.00 CAD) can be ordered
    Given user is on the home page
    And eBook "I Love My Mom (English Armenian Bilingual Book)" in .pdf format costs "0.00 CAD"
    When user adds the eBook to cart
    And user proceeds to checkout
    And user fills all mandatory fields with valid data
    And user clicks "Complete order" button
    Then order should be created with a confirmation number
    And eBook should be available for download

  @ROMAN-33 @bug:ROMAN-38
  Scenario: User cannot add 1000000 books to cart
    Given user is on the home page
    When user selects "Books by Language"
    And user opens book "I Love My Mom (English Afrikaans Bilingual Book for Kids)" in "Paperback" format
    And user sets quantity to "1000000"
    And user clicks "Add to cart" button
    Then quantity validation error should be displayed
    And user should not be able to proceed to checkout
    And order should not be created
```

---

## Bug reports


### ROMAN-38: Quantity field has no max limit: 1,000,000 books can be added to cart

| Priority | Related test | Reproducibility |
|---|---|---|
| Highest | ROMAN-33 | Always |

**Test data:** Card 4242 4242 4242 4242, Exp 12/28, CVC 123

**Description**
There is no validation for the Quantity field. The system allows the user to enter the value "1000000" and proceed to checkout with a total of $24,139,500.00.

**Steps**
1. Open the product page: I Love My Mom (English Afrikaans Bilingual Book for Kids), Paperback, $22.99
2. Change the quantity to 1000000
3. Click "Add to cart"
4. Open the cart and proceed to checkout
5. Fill in the Delivery fields with valid data:
   Country: Canada, First name: John, Last name: Test, Address: 123 Test St, City: Ottawa, Province: Ontario, Postal code: A1A 1A1, Phone: (555) 555-0100
6. Select the shipping method: Basic Shipping (8-15 business days), FREE
7. Fill in the Payment fields: Card 4242 4242 4242 4242, Exp 12/28, CVC 123, Name on card: John Test
8. Click the "Pay now" button

**Expected Result**
The system should not allow adding a quantity above the maximum limit (e.g. 10 or 100).
On steps 2–3, an error message should be displayed (e.g. "Maximum quantity is 10").
The user should not be able to proceed to checkout with a quantity of 1000000.

**Actual Result**
There is no quantity validation. The system allows adding 1000000 items to the cart and proceeding to checkout.

| Subtotal | Shipping | Estimated taxes | Total |
|---|---|---|---|
| $22,990,000.00 | FREE | $1,149,500.00 | CAD $24,139,500.00 |

After clicking "Pay now", the system shows only a payment error ("Your payment details couldn't be verified. Check your card details and try again."), but no quantity limit error. The user can reach the payment step with a $24M order.

---

### ROMAN-34: First name field accepts numeric value "1"

| Priority | Related test | Reproducibility |
|---|---|---|
| Medium | ROMAN-24 | Always |

**Product:** eBook I Love My Mom (English Armenian Bilingual Book)

**Description**
There is no validation for the First name field. The system allows the user to enter the numeric value "1".

**Steps**
1. Open the product page: eBook I Love My Mom (English Armenian Bilingual Book)
2. Add it to cart and go to checkout
3. Fill in the shipping form:
   Country: Canada, First name: **1**, Last name: Test, Address: 123 Test St, City: Ottawa, Province: Ontario, Postal code: A1A 1A1, Phone: (555) 555-0100
4. Select the shipping method: Basic Shipping
5. Scroll to the Payment section
6. Click "Pay now"

**Expected Result**
Error message "Enter a valid first name" is displayed under the First name field. The user stays on the payment page. Payment is not processed.

**Actual Result**
No validation for the First name field. The system accepts the value "1" and shows only the payment error "Your payment details couldn't be verified. Check your card details and try again."
