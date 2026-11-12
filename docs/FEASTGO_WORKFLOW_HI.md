# FeastGo kaise kaam karta hai

Yeh guide FeastGo ke demo ko chalane, order flow test karne, aur app ke alag-alag roles samajhne ke liye hai.

## 1. FeastGo ke baare mein

FeastGo ek food-ordering frontend demo hai. Customer restaurant se food select karke order place karta hai. Restaurant partner order accept karke food prepare karta hai. Delivery partner order pickup karke customer tak pahunchata hai. Admin orders aur platform data manage kar sakta hai.

**Important:** Abhi yeh frontend-only demo hai. API calls `/mock-api` se simulated hoti hain, aur demo data browser ke `localStorage` mein save hota hai. Real backend, database, SMS, email, payment processing, ya multi-user sync connected nahi hai.

## 2. App kholna

- **Live website:** [feastgo-frontend.pages.dev](https://feastgo-frontend.pages.dev)
- **Customer home:** `/`
- **Restaurants:** `/restaurants`
- **Customer login/signup:** `/login`
- **Restaurant partner:** `/seller/login`
- **Delivery partner:** `/delivery/login`
- **Admin:** `/admin/login`

In routes ko live website ke URL ke baad laga kar khola ja sakta hai. Example: `https://feastgo-frontend.pages.dev/seller/login`.

### Local development

Frontend repository ke root folder mein:

```bash
npm ci
npm start
```

Phir `http://localhost:4200` kholo.

## 3. Customer ka order flow

1. Home page se **Explore restaurants** ya **Browse restaurants** kholo.
2. Restaurant select karke menu se item cart mein add karo.
3. Cart check karke **Checkout** par jao.
4. Delivery address choose karo ya naya address add karo.
5. Payment method select karo. Card, UPI, aur Cash on Delivery (COD) options demo ke liye hain; asli payment nahi hoti.
6. Order place karne ke baad **Orders** page ya order tracking page se progress dekho.

Checkout ke liye customer login zaroori hai. Login na hua ho to app login page par bhejegi; login ke baad checkout par wapas aa sakte ho.

### Customer login/signup

- Demo customer ka phone: **9876543210**
- Demo OTP: **123456**
- OTP app mein hi dikhaya jata hai; SMS nahi bheja jata. Code 30 seconds baad expire hota hai. Expire ho to **Resend code** use karo.
- Naya account banane ke liye alag, valid 10-digit number, 3-50 character naam, aur Terms acceptance chahiye. Signup ke waqt bhi demo OTP app par dikhaya jata hai.
- Pehli login ke baad location share karna optional hai; **Skip for now** se aage badh sakte ho.

## 4. Restaurant partner ka flow

Demo login mein form ke required fields mein koi non-empty demo value use kar sakte ho. Real password istemal na karo.

1. `/seller/login` se sign in karke Seller Dashboard kholo.
2. **Menu** section se dishes add, edit, duplicate, availability change, ya remove karo.
3. **Orders** mein naye customer orders dekho.
4. Normal order flow:
   - **Accept**: `placed` -> `confirmed`
   - **Start preparing**: `confirmed` -> `preparing`
   - **Ready for pickup**: `preparing` -> `ready_for_pickup`
5. Order ready hone par delivery assignment ki koshish hoti hai.

Order ko seedha delivery ke liye assign nahi kiya jata jab tak restaurant usay `ready_for_pickup` na kare. Agar demo partner approved, online, aur free ho to order automatically assign hota hai. Partner busy/offline ho ya available na ho to order ready magar unassigned rehta hai; delivery partner baad mein Available Orders se accept kar sakta hai.

## 5. Delivery partner ka flow

Demo login mein required fields mein koi non-empty demo value use kar sakte ho. Default demo partner approved aur online hota hai.

1. `/delivery/login` se sign in karo.
2. Dashboard par online status aur Available Orders dekho.
3. Assigned order ko **Accept order** karo. Agar order available list mein unassigned ho to usay accept karne se woh tumhare naam assign ho jayega.
4. Delivery ko is sequence mein update karo:
   - `assigned` -> **Accept order**
   - `accepted` -> **Reached restaurant**
   - `reached_restaurant` -> **Mark picked up**
   - `picked_up` -> **Start delivery**
   - `out_for_delivery` -> **Reached customer**
   - Customer ke paas pahunch kar delivery OTP/QR section complete karke **Mark delivered**
5. Complete orders **History** mein milte hain.

Demo UI delivery complete karne se pehle kam-se-kam 4 digits mangta hai. Har demo order mein customer OTP/QR available ho yeh zaroori nahi; is mock mein OTP server se verify bhi nahi hota. QR scan camera/browser support aur permission par depend karta hai.

**Orders badge** active assigned deliveries ka count hai; delivered orders history mein count hote hain. Ek partner ke paas ek waqt mein ek active delivery ho sakti hai. Assigned order ko reject karne par woh unassigned ho kar dobara available ho sakta hai.

## 6. Admin ka flow

`/admin/login` par required fields mein koi non-empty demo value use karke sign in karo. Real credentials mat use karo.

Admin dashboard se platform sections jaise users, sellers, delivery partners, restaurants, foods/categories, orders, refunds, coupons, reviews, reports, notifications, aur settings manage kiye ja sakte hain.

**Orders** section mein admin:

- Orders filter/search karke status update kar sakta hai.
- Delivery partner assign ya reassign kar sakta hai.
- Assignment ke liye partner available hona chahiye; busy, offline, unapproved, ya suspended partner ko assign nahi kiya ja sakta.

Normal testing mein seller aur delivery partner ka status flow use karo. Admin status changes ko moderation/support ke liye rakho.

## 7. Order statuses ka matlab

Order status aur delivery status alag cheezein hain:

| Order status | Meaning |
| --- | --- |
| `placed` | Customer ne order create kiya |
| `confirmed` | Restaurant ne order accept kiya |
| `preparing` | Restaurant food bana raha hai |
| `ready_for_pickup` | Food pickup ke liye ready hai |
| `out_for_delivery` | Delivery partner ne food pickup kar liya |
| `delivered` | Order complete ho gaya |
| `rejected` / `cancelled` / `failed` / `returned` | Order reject, cancel, fail, ya return hua |

Delivery progress alag se `pending`, `assigned`, `accepted`, `reached_restaurant`, `picked_up`, `out_for_delivery`, `reached_customer`, aur `delivered` statuses rakhta hai.

## 8. Pura order flow test karne ka tareeqa

1. Agar clean test chahiye to customer ke roop mein login karke **Settings -> Local Testing Data** se pehle purana demo data clear karo.
2. Customer ke roop mein `/login` kholo, `9876543210` se login karo, aur `123456` OTP enter karke restaurant se order place karo.
3. Usi browser mein `/seller/login` kholo aur order ko **Accept -> Start preparing -> Ready for pickup** karo.
4. `/delivery/login` kholo. Demo partner online/free ho to assigned order accept karo; warna dashboard ke Available Orders se order accept karo.
5. Delivery status ko upar diye gaye sequence mein aage badhao aur final step mein 4+ digit demo value enter karke delivered mark karo.
6. `/orders` par customer order wapas dekh kar delivered status verify karo.

Agar order auto-assign nahi hota, check karo ki delivery partner online hai aur koi doosri active delivery usay busy nahi kar rahi.

## 9. Demo data aur reset

- Demo changes browser ke FeastGo storage mein save hote hain, isliye usi browser aur website origin par reload karne ke baad data rehta hai.
- Live website aur `localhost` ka data alag hota hai. Alag browser/device par demo changes share nahi hote.
- Customer login ke baad **Settings -> Local Testing Data -> Clear local testing data** se is browser ke FeastGo demo accounts, orders, cart, favorites, preferences, aur sign-in state reset kar sakte ho. Confirmation ke baad app reload hoti hai.
- Reset sirf current website origin ke FeastGo data ko clear karta hai; doosri websites ka data nahi.

## 10. GitHub se live deploy

Frontend repo: [Rizwan8880/FeastGo-Frontend](https://github.com/Rizwan8880/FeastGo-Frontend)

`main` branch par har push GitHub Actions workflow ko chalata hai. Workflow dependencies install karta hai, Angular production build banata hai (`dist/client/browser`), aur Cloudflare Pages project `feastgo-frontend` par deploy karta hai.

Workflow file: [`.github/workflows/deploy-cloudflare-pages.yml`](../.github/workflows/deploy-cloudflare-pages.yml)

## 11. Demo ki limitations

- Login aur OTP demo-only hain; real identity verification nahi hoti.
- Card, UPI, COD selection, refunds, notifications, chat, aur earnings real services se connected nahi hain.
- Demo data browser mein rehta hai; doosre users/devices ke saath shared nahi hota.
- Real customers ke liye launch karne se pehle backend/API, database, secure authentication, SMS/email provider, payment gateway, aur server-side authorization add karna hoga.
