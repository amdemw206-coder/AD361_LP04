
## Step 1 
- Five conditions that affect whether a reservation is accepted.
    - (i) Reservation between 1 - 31 nights 
    - (ii) Reservation between 1 - 12 guests 
    - (iii) Additional fees for more than 5 guests
    - (iv) Every night in the requested date range must be available
    - (v) You must pay for your reservation 
- Five actions that may occur based on those conditions.
    - (i) Reservation is created
    - (ii) Failure in payment ('Cancelled' reservation state)
    - (iii) 'ValueError' for invalid reservation information 
    - (iv) 'UnavailableError' for unavailable reservation dates
    - (v) Final payment (base fee + additional guest fee)

## Step 2 
| Conditions and Actions                           | Rule 1 | Rule 2 | Rule 3 | Rule 4 | Rule 5 | Rule 6 |
| ------------------------------------------------ | :----: | :----: | :----: | :----: | :----: | :----: |
| Reservation between 1 - 31 nights                |   T    |   T    |   T    |   T    |   T    |   F    |
| Reservation between 1 - 12 guests                |   T    |   T    |   F    |   T    |   T    |   F    |
| Additional fees for more than 5 guests           |   T    |   F    |   T    |   T    |   T    |   F    |
| Paid for your reservation                        |   T    |   T    |   T    |   F    |   T    |   F    |
| Every night requested range available            |   T    |   T    |   T    |   T    |   F    |   F    |
| **Reservation is created**                       |   X    |   X    |        |   -    |        |        |
| **Failure in payment**                           |        |        |   -    |   X    |   -    |   X    |
| **'ValueError' Occurs**                          |        |        |   X    |        |        |   X    |
| **'UnavailableError' Occurs**                    |        |        |        |        |   X    |   X    |
| **Final payment (base fee + additional fees)**   |   X    |   X    |        |        |        |        |

- **Rule 1:** Create a valid reservation, 5+ guests to account for additional fees.
- **Rule 2:** Create a valid reservation, <= 5 guests for base fee.
- **Rule 3:** Create a invalid reservation, < 1 or > 12 guests to raise 'ValueError'.
- **Rule 4:** Create a invalid reservation, user hasn't paid for reservation. 
- **Rule 5:** Create a invalid reservation, 1 or more nights in the date range aren't available.
- **Rule 6:** Create a invalid reservation, every condition is false.


## Step 3 

### TC01: Valid Reservation With Extra Guest Fee
- **Check-in:** October 1
- **Checkout:** October 8
- **Number of guests:** 6
- **Reservation Paid?** T 
- **Availability:** Every requested night is available.
- **Expected Result:** The reservation is created, final payment is confirmed.
- **Expected Price:** `$1475`
    - 7 Nights Base Fee: `$1400`
    - One time cleaning fee: `$50`
    - Extra Guest Fee: `$25`

### TC02: Valid Reservation w/o Extra Guest Fee
- **Check-in:** October 1
- **Checkout:** October 8
- **Number of guests:** 4
- **Reservation Paid?** T 
- **Availability:** Every requested night is available.
- **Expected Result:** The reservation is created, final payment is confirmed.
- **Expected Price:** `$1450`
    - 7 Nights Base Fee: `$1400`
    - One time cleaning fee: `$50`

### TC03: Invalid Reservation, Exceeds Guest Limit  
- **Check-in:** October 1
- **Checkout:** October 8
- **Number of guests:** 15
- **Reservation Paid?** T 
- **Availability:** Every requested night is available.
- **Expected Result:** The reservation is not created, invalid reservation information
- **Expected Error:** `ValueError`

### TC04: Invalid Reservation, User Hasn't Paid 
- **Check-in:** July 4
- **Checkout:** July 7
- **Number of guests:** 3
- **Reservation Paid?** F 
- **Availability:** Every requested night is available.
- **Expected Result:** The reservation is not created, final payment not confirmed.
- **Expected Error:** Reservation state is set to `Cancelled` 


### TC05: Invalid Reservation, Date range not available
- **Check-in:** September 11
- **Checkout:** September 15
- **Number of guests:** 7
- **Reservation Paid?** T 
- **Availability:** Requested night "September 13th" isn't available.
- **Expected Result:** Reservation is not created  
- **Expected Error:** `Unavailable Error`

### TC06: Invalid Reservation, Every Condition is Invalid
- **Check-in:** October 1
- **Checkout:** October 17
- **Number of guests:** 0
- **Reservation Paid?** F 
- **Availability:** Requested night "October 2" is Unavailable.
- **Expected Result:** Failure in payment, invalid reservation information, and date range errors.
- **Expected Errors:** 
    - `Unavailable Error`
    - `Value Error`
    - Reservation state is set to `Cancelled` 

## Step 4 
- Identify five possible reservation states.
    - (i) `Pending` state
    - (ii) `Confirmed` state 
    - (iii) `Cancelled`state
    - (iv) `Checked In` state
    - (v) `Completed` state 
- Identify five events that may cause the reservation to change states.
    - Creating a new valid reservation 
    - Successful payment of reservation  
    - Failed payment of reservation 
    - A confirmed reservation on it's check-in date
    - A confirmed reservation on it's checkout date

## Step 5 
| Current State | Event                  | Expected State     | Expected Result                                  |
| ------------- | ---------------------  | --------------     | ------------------------------------------------ |
| Pending       | Payment Succeeds       | Confirmed          | Reservation is confirmed                         |
| Pending       | Payment Fails          | Cancelled          | Reservation is cancelled                         |
| Confirmed     | @ Check-in Date        | Checked In         | Reservation is 'Checked In' during check-in date |
| Checked In    | @ Checkout Date        | Completed          | Reservaion is 'Completed' after checkout date    |
| Pending       | Cancelling Reservation | Cancelled          | Reservation is cancelled                         |
| Confirmed     | Cancelling Reservation | Cancelled          | Reservation is cancelled                         |
| Cancelled     | Validating Reservation | Cancelled          | `InvalidStateError`                              |
| Completed     | Changing checkout date | Completed          | `InvalidStateError`                              |
| Pending       | Trying to checkout     | Pending            | `InvalidStateError`                              |

## Step 6:
| Test ID | Starting State | Event                           | Expected State     | Expected Result                   | Transition Type |
| ------- | -------------- | ------------------------------- | ------------------ | --------------------------------- | --------------- |
| ST01    | Pending        | Payment succeeds                | Confirmed          | The reservation is confirmed.     | Valid           |
| ST02    | Pending        | Payment Fails                   | Cancelled          | The reservation is cancelled.     | Valid           |
| ST03    | Confirmed      | System Recognizes Check-in Date | Checked In         | The Reservation is  Checked In    | Valid           |
| ST04    | Checked In     | System Recognizes Checkout Date | Completed          | The Reservtaion is  Completed     | Valid           |
| ST05    | Pending        | Cancelling Reservation          | Cancelled          | The reservation is cancelled.     | Valid           |
| ST06    | Confirmed      | Cancelling Reservation          | Cancelled          | The reservation is cancelled.     | Valid           |
| ST07    | Cancelled      | Validating Reservation          | Cancelled          | `InvalidStateError` Occurs        | Invalid         |
| ST08    | Completed      | Changing checkout date          | Completed          | `InvalidStateError` Occurs        | Invalid         |
| ST09    | Pending        | Trying to Checkout              | Pending            | `InvalidStateError` Occurs        | Invalid         |
| ST10    | Cancelled      | Trying to Checkout              | Cancelled          | `InvalidStateError` Occurs        | Invalid         |
| ST11    | Pending        | User Payment succeeds           | Confirmed          | The Reservation is Confirmed.     | Valid           | 
| ST12    | Confirmed      | System Recognizes Check-in Date | Checked In         | The Reservation is Checked In     | Valid           |
| ST13    | Checked In     | System Recognizes Checkout Date | Completed          | The Reservation is Completed      | Valid           |