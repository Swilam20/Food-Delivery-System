# Food Delivery App Functional Analysis  
**Case Study: Talabat**
---

## 📊 Part A — Analyst View
---

### 1. Authentication & Security Management

**Core Functions**
- User authentication (login)
- Third-party login 
- OTP verification (SMS or email)
- Password reset

---

### 2. Registration Management

**Functions**
- Create new account
- Register via email, mobile number or third-party provider

---

### 3. Account & Profile Management

**User-facing Functions**
- Update personal information (name, phone, email, photo)
- Manage delivery addresses (multiple saved addresses + labels home/work/..)
- Add debit/credit cards 
- Manage referral invitations (invite friends)
- Language and region settings
- Account upgrade (Pro/premium)
- Delete account 

---

### 4. Merchant Management (Restaurants & Stores)

**User-facing Functions**
- Browse merchants and categories
- Search for merchants
- Filter merchants (cuisine, discount, rating)
- Sort merchants (top rated, free delivery)
- Mark merchants as favorites

**Merchant Profile Page Displays**
- Ratings & reviews
- Delivery fees
- Opening hours
- Promotions
- Distance 

**Admin Functions**
- Add/edit merchant information

---

### 5. Product & Menu Management

**User-facing Functions**
- View menu items
- View product details (description)
- Product customization (add-ons)
- Select quantity
- Product suggestions 
- Recommended combos
- Out-of-stock indication

**Admin**
- Add/edit menu items

---

### 6. Cart Management

**Functions**
- Add/remove items
- Modify quantity
- Edit product customizations
- Apply promo or coupon codes
- Display fees, service fees, VAT
- Display ETA before checkout
- Delivery fee calculation

---

### 7. Order Management

**Functions**
- Place order
- Choose delivery location
- Change payment method
- Live Order Tracking
- Cancel order (rules dependent)
- Call  rider/driver
- Add delivery instructions/notes


---

### 8. Payment Management

**Functions**
- Support multiple payment modes:
  - Cash on delivery
  - Credit/debit card
  - Wallet balance
- Refund processing 
- Invoice/receipt generation

---

### 9. Loyalty Program

**Functions**
- Discount & promo codes
- Collect points per transaction
- Points redemption
- Partner integrations for point exchange

---

### 10. History & Reordering

**Functions**
- View past orders 
- Reorder from history
- Rate merchants

---

### 11. Search & Discovery

**Functions**
- Keyword search (merchant or food)
- Category discovery
- Location-based suggestions
- Trending merchants

---

### 12. Notification & Communication Management

**Functions**
- Push notifications (order updates, promos)
- Customer feedback 

---
---

## 🛠️ Part B — Technical View 
---

### 1. Analyze Place oreder Funcation
---

#### 1. Flowchart Diagram

```mermaid
flowchart TD
    %% =========================
    %% Start & Validation
    %% =========================
    Start([Start])
    Validate[Validate product availability]
    Available{Is order available?}
    PriceUpdated{Has price been updated?}

    NotifyUpdate[Notify customer with updates]
    ToCart[Redirect to Cart]
    End([End])

    Start --> Validate --> Available
    Available -- No --> NotifyUpdate --> ToCart --> End

    Available -- Yes --> PriceUpdated
    PriceUpdated -- Yes --> NotifyUpdate 


    %% =========================
    %% Order Creation & Payment Choice
    %% =========================
    CreateOrder[Create order: Pending Status]
    PaymentMethod{Payment method?}

    PriceUpdated -- No --> CreateOrder --> PaymentMethod


    %% =========================
    %% Cash Payment Flow (Independent)
    %% =========================
    subgraph CashFlow [Cash Payment]
        Cash_Send[Send order to restaurant]
        Cash_Accept{Restaurant accepts order?}

        Cash_Dispatch[Assign delivery]
        Cash_Success[Notify customer: Order placed successfully]
        Cash_Track[Redirect to Order Tracking]
        Cash_End([End])

        Cash_Reject[Notify customer: Order rejected]
        Cash_Restaurant[Redirect to Restaurant screen]

        Cash_Send --> Cash_Accept
        Cash_Accept -- Yes --> Cash_Dispatch --> Cash_Success --> Cash_Track --> Cash_End
        Cash_Accept -- No --> Cash_Reject --> Cash_Restaurant --> Cash_End
    end


    %% =========================
    %% Card Payment Flow (Independent)
    %% =========================
    subgraph CardFlow [Card Payment]
        Card_Authorize[Authorize payment]
        Card_Authorized{Authorization successful?}

        Card_Fail[Notify customer: Payment failed]
        Card_Checkout[Redirect to Checkout]
        Card_End([End])

        Card_Send[Send order to restaurant]
        Card_Accept{Restaurant accepts order?}

        Card_Capture[Capture authorized amount]
        Card_Dispatch[Assign delivery]
        Card_Success[Notify customer: Order placed successfully]
        Card_Track[Redirect to Order Tracking]

        Card_Release[Release authorization]
        Card_Restaurant[Redirect to Restaurant screen]

        Card_Authorize --> Card_Authorized
        Card_Authorized -- No --> Card_Fail --> Card_Checkout --> Card_End

        Card_Authorized -- Yes --> Card_Send --> Card_Accept
        Card_Accept -- Yes --> Card_Capture --> Card_Dispatch --> Card_Success --> Card_Track --> Card_End
        Card_Accept -- No --> Card_Release --> Card_Restaurant --> Card_End
    end


    %% =========================
    %% Branching (No subgraph cross-links)
    %% =========================
    PaymentMethod -- Cash --> Cash_Send
    PaymentMethod -- Card --> Card_Authorize


```

#### 2. Sequence Diagram

```mermaid
sequenceDiagram
    autonumber

    participant U as Customer
    participant A as App
    participant B as Backend
    participant R as Restaurant
    participant P as Payment Gateway
    participant D as Delivery Service

    %% =========================
    %% Start & Validation
    %% =========================
    U->>A: Click "Place Order"
    A->>B: PlaceOrder request

    B->>B: Validate product availability
    alt Order not available
        B->>A: Notify updates / unavailable items
        A->>U: Redirect to Cart
    else Order available
        B->>B: Check price updates
        alt Price updated
            B->>A: Notify customer with updated price
            A->>U: Redirect to Cart
        else Price unchanged
            %% =========================
            %% Order Creation
            %% =========================
            B->>B: Create order (Status: PENDING)

            %% =========================
            %% Payment Method Decision
            %% =========================
            alt Payment = Cash
                %% ===== Cash Flow =====
                B->>R: Send order (pending acceptance)
                R-->>B: Accept / Reject

                alt Restaurant accepts
                    B->>D: Assign delivery
                    B->>A: Order placed successfully
                    A->>U: Redirect to Order Tracking
                else Restaurant rejects
                    B->>A: Order rejected
                    A->>U: Redirect to Restaurant screen
                end

            else Payment = Card
                %% ===== Card Authorization =====
                B->>P: Authorize payment
                P-->>B: Authorization result

                alt Authorization failed
                    B->>A: Payment failed
                    A->>U: Redirect to Checkout
                else Authorization successful
                    B->>R: Send order (pending acceptance)
                    R-->>B: Accept / Reject

                    alt Restaurant accepts
                        B->>P: Capture authorized amount
                        P-->>B: Capture confirmation
                        B->>D: Assign delivery
                        B->>A: Order placed successfully
                        A->>U: Redirect to Order Tracking
                    else Restaurant rejects
                        B->>P: Release authorization
                        P-->>B: Authorization released
                        B->>A: Order rejected
                        A->>U: Redirect to Restaurant screen
                    end
                end
            end
        end
    end
```

#### 3. Pseudocode


```Pseudocode
PlaceOrder(Customer, Cart, PaymentMethod):

    for each item in Cart.items:
        if NOT CheckAvailability(item):
            Notify(Customer, item.name + " is not available")
            return CartPage

    server_total = CalculatePrice(Cart)
    client_total = Cart.total_amount

    if client_total != server_total:
        Notify(Customer, "Some item prices have been updated. Please review your cart.")
        return CartPage

    Order = CreateOrder(Customer, Cart, PaymentMethod, total_amount=server_total, status="PENDING")

    auth_ref = null

    if Order.payment_method == "CARD":
        auth_ref = AuthorizePayment(Customer, Order.total_amount)
        if auth_ref == null:
            UpdateOrderStatus(Order.id, "FAILED_PAYMENT")
            Notify(Customer, "Payment authorization failed")
            return CheckoutPage

    restaurantDecision = SendOrderToRestaurant(Order)   # "ACCEPTED" or "REJECTED"

    if restaurantDecision == "REJECTED":
        if Order.payment_method == "CARD":
            ReleaseAuthorization(auth_ref)

        UpdateOrderStatus(Order.id, "REJECTED")
        Notify(Customer, "Restaurant is busy right now. Please try again later.")
        return RestaurantPage

    if Order.payment_method == "CARD":
        captureOk = CaptureAuthorizedAmount(auth_ref, Order.total_amount)
        if NOT captureOk:
            UpdateOrderStatus(Order.id, "FAILED_PAYMENT")
            Notify(Customer, "Payment capture failed. Please try another payment method.")
            return CheckoutPage

    UpdateOrderStatus(Order.id, "CONFIRMED")

    AssignDriver(Order)
    Notify(Customer, "Order placed successfully!")
    return OrderTrackingPage
```

#### 3. Entity Relationship Diagram

```mermaid
erDiagram

    CUSTOMER {
        int cust_id PK
        string cust_name
        string cust_address
        string cust_mobile_number
    }

    PAYMENTMETHOD {
        int payment_id PK
        int cust_id FK
        string type
        string token
        string last4
        string expiry
    }

    CART {
        int cart_id PK
        int cust_id FK
    }

    RESTAURANT {
        int rest_id PK
        string rest_address
        string rest_status
    }

    MENU {
        int menu_id PK
        int rest_id FK
    }

    MENUITEM {
        int menuitem_id PK
        int menu_id FK
        string name
        float price
        string item_status
    }

    ORDERS {
        int o_id PK
        int cust_id FK
        int rest_id FK
        string o_payment_method
        string o_status
        float o_total_amount
    }

    CARTMENUITEM {
        int cart_id FK
        int menuitem_id FK
        int quantity
    }

    ORDERMENUITEM {
        int o_id FK
        int menuitem_id FK
        int quantity
        float unit_price_at_order
    }

    %% =========================
    %% Relationships
    %% =========================

    CUSTOMER ||--o{ PAYMENTMETHOD : owns
    CUSTOMER ||--|| CART : has
    CUSTOMER ||--o{ ORDERS : places

    RESTAURANT ||--|| MENU : has
    MENU ||--o{ MENUITEM : contains

    CART ||--o{ CARTMENUITEM : includes
    MENUITEM ||--o{ CARTMENUITEM : selected_as

    ORDERS ||--o{ ORDERMENUITEM : contains
    MENUITEM ||--o{ ORDERMENUITEM : ordered_as

    RESTAURANT ||--o{ ORDERS : receives

```
