# How FeastGo Works

This guide explains how to run the FeastGo demo, test its order flow, and use its different roles.

## 1. About FeastGo

FeastGo is a food-ordering frontend demo. Customers choose food from a restaurant and place an order. The restaurant partner accepts the order and prepares the food. A delivery partner picks it up and takes it to the customer. Admins can manage orders and platform data.

**Important:** This is currently a frontend-only demo. API calls are simulated through `/mock-api`, and demo data is stored in the browser's `localStorage`. There is no connected production backend, database, SMS, email, payment processing, or multi-user sync.

## 2. Open the app

- **Live website:** [feastgo-frontend.pages.dev](https://feastgo-frontend.pages.dev)
- **Customer home:** `/`
- **Restaurants:** `/restaurants`
- **Customer login/signup:** `/login`
- **Restaurant partner:** `/seller/login`
- **Delivery partner:** `/delivery/login`
- **Admin:** `/admin/login`

Open a route by adding it to the live website URL. For example: `https://feastgo-frontend.pages.dev/seller/login`.

### Local development

From the frontend repository root:

```bash
npm ci
npm start
```

Then open `http://localhost:4200`.

## 3. Customer order flow

1. Select **Explore restaurants** or **Browse restaurants** from the home page.
2. Choose a restaurant and add menu items to the cart.
3. Review the cart and select **Checkout**.
4. Choose a delivery address or add a new one.
5. Select a payment method. Card, UPI, and Cash on Delivery (COD) are demo options; no real payment is made.
6. After placing the order, follow its progress from the **Orders** page or order-tracking page.

Customers must sign in to check out. If not signed in, the app redirects to the login page; after signing in, they can return to checkout.

### Customer login/signup

- Demo customer phone number: **9876543210**
- Demo OTP: **123456**
- The OTP is shown in the app; no SMS is sent. It expires after 30 seconds. Select **Resend code** if it expires.
- To create an account, use a different valid 10-digit phone number, a name of 3-50 characters, and accept the Terms. The demo OTP is shown in the app during signup as well.
- Sharing location after the first login is optional; select **Skip for now** to continue.

## 4. Restaurant partner flow

For the demo login, enter any non-empty values in the required fields. Do not use a real password.

1. Sign in at `/seller/login` to open the Seller Dashboard.
2. In **Menu**, add, edit, duplicate, change availability, or remove dishes.
3. Find new customer orders under **Orders**.
4. Use the normal order flow:
   - **Accept**: `placed` -> `confirmed`
   - **Start preparing**: `confirmed` -> `preparing`
   - **Ready for pickup**: `preparing` -> `ready_for_pickup`
5. Delivery assignment is attempted after the order is ready.

An order is not assigned for delivery until the restaurant marks it `ready_for_pickup`. If a demo partner is approved, online, and available, the order is assigned automatically. If no partner is available, the order remains ready but unassigned; a delivery partner can accept it later from Available Orders.

## 5. Delivery partner flow

For the demo login, enter any non-empty values in the required fields. The default demo partner is approved and online.

1. Sign in at `/delivery/login`.
2. Check online status and Available Orders on the dashboard.
3. Accept an assigned order using **Accept order**. If an unassigned order appears in the available list, accepting it assigns the order to you.
4. Update delivery status in this sequence:
   - `assigned` -> **Accept order**
   - `accepted` -> **Reached restaurant**
   - `reached_restaurant` -> **Mark picked up**
   - `picked_up` -> **Start delivery**
   - `out_for_delivery` -> **Reached customer**
   - After reaching the customer, complete the delivery OTP/QR section and select **Mark delivered**.
5. Completed orders appear in **History**.

The demo UI requires at least four digits before completing a delivery. A customer OTP/QR is not necessarily available for every demo order, and the mock does not verify the OTP on a server. QR scanning depends on camera/browser support and permission.

The **Orders** badge counts active assigned deliveries; delivered orders are shown in History. A partner can have one active delivery at a time. Rejecting an assigned order can unassign it and make it available again.

## 6. Admin flow

At `/admin/login`, enter any non-empty demo values in the required fields. Do not use real credentials.

The Admin Dashboard can manage platform sections such as users, sellers, delivery partners, restaurants, foods/categories, orders, refunds, coupons, reviews, reports, notifications, and settings.

In the **Orders** section, admins can:

- Filter or search orders and update their status.
- Assign or reassign a delivery partner.
- Assign only an available partner; busy, offline, unapproved, or suspended partners cannot be assigned.

For normal testing, use the seller and delivery partner status flows. Reserve admin status changes for moderation or support.

## 7. Order status meanings

Order status and delivery status are separate:

| Order status | Meaning |
| --- | --- |
| `placed` | The customer created the order |
| `confirmed` | The restaurant accepted the order |
| `preparing` | The restaurant is preparing the food |
| `ready_for_pickup` | The food is ready for pickup |
| `out_for_delivery` | The delivery partner picked up the food |
| `delivered` | The order is complete |
| `rejected` / `cancelled` / `failed` / `returned` | The order was rejected, cancelled, failed, or returned |

Delivery progress has its own statuses: `pending`, `assigned`, `accepted`, `reached_restaurant`, `picked_up`, `out_for_delivery`, `reached_customer`, and `delivered`.

## 8. Test the complete order flow

1. To start with clean test data, sign in as a customer and select **Settings -> Local Testing Data** to clear previous demo data.
2. Open `/login` as a customer, sign in with `9876543210`, enter OTP `123456`, and place an order from a restaurant.
3. In the same browser, open `/seller/login` and move the order through **Accept -> Start preparing -> Ready for pickup**.
4. Open `/delivery/login`. If the demo partner is online and available, accept the assigned order; otherwise, accept it from Available Orders.
5. Advance the delivery status in the sequence above. At the final step, enter any four-or-more-digit demo value and mark the order delivered.
6. Open `/orders` as a customer and verify that the order is marked delivered.

If automatic assignment does not happen, check that the delivery partner is online and does not already have another active delivery.

## 9. Demo data and reset

- Demo changes are stored in FeastGo's browser storage, so they persist after reloads on the same browser and website origin.
- Data on the live website and `localhost` are separate. Demo changes are not shared across browsers or devices.
- After customer login, use **Settings -> Local Testing Data -> Clear local testing data** to reset FeastGo demo accounts, orders, cart, favorites, preferences, and sign-in state for the current browser. The app reloads after confirmation.
- Reset only clears FeastGo data for the current website origin; it does not clear data for other websites.

## 10. Deploy from GitHub

Frontend repository: [Rizwan8880/FeastGo-Frontend](https://github.com/Rizwan8880/FeastGo-Frontend)

Every push to the `main` branch runs a GitHub Actions workflow. It installs dependencies, creates an Angular production build (`dist/client/browser`), and deploys to the Cloudflare Pages project `feastgo-frontend`.

Workflow file: [`.github/workflows/deploy-cloudflare-pages.yml`](../.github/workflows/deploy-cloudflare-pages.yml)

## 11. Demo limitations

- Login and OTP are for demo purposes; there is no real identity verification.
- Card, UPI, COD selection, refunds, notifications, chat, and earnings are not connected to real services.
- Demo data stays in the browser and is not shared with other users or devices.
- Before launching for real customers, add a backend/API, database, secure authentication, SMS/email provider, payment gateway, and server-side authorization.
