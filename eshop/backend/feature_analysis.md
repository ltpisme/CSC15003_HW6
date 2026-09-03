# Feature Analysis Report

## Repository Architecture

The repository contains four applications:

| Area | Technology | Confirmed Files | Role |
| --- | --- | --- | --- |
| Backend API | Node.js, Express, SQLite | `backend/server.js`, `backend/database.js` | Defines HTTP endpoints, JWT authentication middleware, in-memory cart, SQLite queries, seed data |
| Web frontend | React, Vite, Tailwind | `frontend-web/src/pages/*.jsx`, `frontend-web/src/context/*.jsx` | Customer product browsing, cart, checkout, profile/order history |
| Admin frontend | React, Vite, Tailwind | `frontend-admin/src/App.jsx` | Admin login, dashboard, product/category/coupon/order management, CSV preview/import |
| Mobile app | React Native, Expo | `frontend-mobile/App.js` | Mobile product browsing, cart, checkout, profile/order history |
| Database layer | SQLite | `backend/database.sqlite`, schema in `backend/database.js` | Tables: `users`, `products`, `categories`, `orders`, `coupons`, `coupon_usage` |

Authentication is implemented by `authenticateToken()` in `backend/server.js:100-110`. It verifies `Authorization: Bearer <token>` and places the JWT payload on `req.user`. There is no backend role-check middleware; admin APIs only require a valid token. The admin UI performs a client-side role check after login in `frontend-admin/src/App.jsx:61-74`.

External dependencies are limited to local libraries: Express, CORS, body-parser, jsonwebtoken, sqlite3, axios, React, React Native/Expo. No payment provider, email provider, file upload storage, or CSV parsing library is used.

---

## FR-05 Product List and Search

### 1. Overview

Business purpose: display products to customers and allow search by product name.

Actors:

| Actor | Entry Point | Permission |
| --- | --- | --- |
| Web customer/guest | `frontend-web/src/pages/Home.jsx` | Public |
| Mobile customer/guest | `frontend-mobile/App.js` home view | Public |
| Backend caller | `GET /api/products` | Public |

Scope confirmed in code:

- Web home page fetches products on mount and on search submit.
- Mobile home view fetches products on mount and on search button press.
- Backend supports optional `search` query parameter.
- Product cards render image, name, price, detail link/action, and add-to-cart action.

Need verification:

- No dedicated backend service/repository layer exists.
- No pagination, sorting, category filtering, empty-state message, or search sanitization is implemented.
- Requirements mention safe keyword display and single `<h1>`, but web implementation renders the search term with `dangerouslySetInnerHTML` and can render a second `<h1>` result count.

Components:

| Layer | Component | Responsibility |
| --- | --- | --- |
| Web frontend | `Home()` in `frontend-web/src/pages/Home.jsx:6-116` | Search form, product grid, product fetch, error HTML rendering, add-to-cart button |
| Web state | `CartProvider` in `frontend-web/src/context/CartContext.jsx` | Stores cart in React state; `addToCart()` appends product line |
| Mobile frontend | `fetchProducts()`, `handleSearch()`, `renderHome()` in `frontend-mobile/App.js:84-111`, `480-535` | Fetches product list/search results and renders mobile product list |
| Backend API | `app.get("/api/products")` in `backend/server.js:141-157` | Executes full list query or SQL `LIKE` search |
| Database | `products` table in `backend/database.js:63-71` | Stores product catalog fields |

### 2. End-to-End Workflows

#### Workflow 1: Initial Product List Load

Purpose: show all products when home screen opens.

Trigger: page/app mount.

Actor: guest or logged-in customer.

Flow:

```
User opens Home
    ↓
Web: useEffect() calls fetchProducts() with empty query
Mobile: useEffect() calls fetchProducts()
    ↓
GET /api/products?search=
    ↓
Backend sees req.query.search as falsy
    ↓
SQLite: SELECT * FROM products
    ↓
Response: JSON array of product records
    ↓
Frontend stores products and renders grid/list
```

Normal flow:

- Web `Home.fetchProducts()` calls `axios.get("http://localhost:3000/api/products?search=")` in `frontend-web/src/pages/Home.jsx:12-23`.
- Mobile `fetchProducts()` calls `fetch(`${API_URL}/products?search=${query}`)` and parses response text as JSON in `frontend-mobile/App.js:84-102`.
- Backend returns `res.json(rows)` from `SELECT * FROM products` in `backend/server.js:152-155`.

Alternative flow:

- If products table is empty, backend returns `[]`. Web and mobile render no products. No explicit empty-state text is implemented.

Error flow:

- Full-list backend query does not check `err`; if SQLite fails, `rows` may be undefined and still passed to `res.json(rows)`. Need verification through fault injection.
- Mobile catches network errors and stores the message in `errorHtml`; web only stores string response errors.

#### Workflow 2: Product Search by Name

Purpose: filter product list by product name substring.

Trigger: user enters text and submits search.

Actor: guest or logged-in customer.

Flow:

```
User enters search keyword
    ↓
Web handleSearch() prevents submit and calls fetchProducts(search)
Mobile handleSearch() calls fetchProducts(search)
    ↓
GET /api/products?search=<raw keyword>
    ↓
Backend builds SQL string:
SELECT * FROM products WHERE name LIKE '%<keyword>%'
    ↓
SQLite executes query
    ↓
Response JSON array, or HTML database error string
    ↓
Frontend displays result list, or renders error HTML
```

Normal flow:

- Search is substring matching on `products.name`.
- Matching is delegated to SQLite `LIKE`; case behavior depends on SQLite collation and text values.

Alternative flow:

- Empty search string behaves like full product list because backend checks `if (searchQuery)`.
- Keyword with no matches returns `[]`; no explicit empty-state message is rendered.

Error/exception flow:

- Search query is interpolated directly into SQL in `backend/server.js:143-145`. Inputs containing quote characters or SQL syntax can cause database errors or injection behavior.
- Backend returns an HTML string on search database error in `backend/server.js:146-149`.
- Web renders backend error HTML with `dangerouslySetInnerHTML` in `frontend-web/src/pages/Home.jsx:68-72`.
- Web renders the user search term with `dangerouslySetInnerHTML` in `frontend-web/src/pages/Home.jsx:61-65`.
- Mobile renders search text as a React Native `<Text>` value, not HTML, in `frontend-mobile/App.js:502-505`.

#### Workflow 3: Product Card Add To Cart

Purpose: add a listed product to the local cart.

Trigger: user presses "Thêm vào giỏ".

Actor: web customer.

Flow:

```
User clicks Add to Cart on product card
    ↓
Home passes product and quantity 1 to addToCart()
    ↓
CartContext appends a new cart line
    ↓
No backend request
```

Normal flow:

- `Home` calls `addToCart({ ...p, quantity: 1 }, 1)` in `frontend-web/src/pages/Home.jsx:97-99`.
- `CartContext.addToCart()` appends `{...product, quantity}` in `frontend-web/src/context/CartContext.jsx`.

Alternative/error flow:

- Adding the same product repeatedly creates duplicate lines in web cart; there is no merge-by-product-id logic in web.

### 3. Data Flow Analysis

Database entity: `products` from `backend/database.js:63-71`.

| Field | Type | Meaning | Validation Rule |
| --- | --- | --- | --- |
| `id` | INTEGER PRIMARY KEY AUTOINCREMENT | Product identifier | Generated by SQLite |
| `name` | TEXT | Product display/search name | No DB constraint; search uses `LIKE` |
| `price` | INTEGER | Product price | No DB constraint; frontend formats with `Number(price)` |
| `description` | TEXT | Product description | No DB constraint |
| `imageUrl` | TEXT | Product image URL | No DB constraint; web image `alt` is empty |
| `category_id` | INTEGER | Category reference | No foreign key declared |

Lifecycle:

- Created by seed data in `backend/database.js:96-103`, product CRUD in `backend/server.js:167-196`, or CSV import in `backend/server.js:199-239`.
- Read by `GET /api/products`.
- Consumed by web home, mobile home, product detail, cart, checkout.
- Search input is not persisted.

### 4. Source Code Impact

| File Path | Function/Class | Layer | Responsibility | Related Workflow |
| --- | --- | --- | --- | --- |
| `frontend-web/src/pages/Home.jsx` | `Home` | Web UI | Product grid, search input, product fetch, renders search text/error HTML | Initial load, search, add to cart |
| `frontend-web/src/pages/Home.jsx` | `fetchProducts(query)` | Web UI/API client | Calls `/api/products?search=...`, stores products or HTML error | Initial load, search |
| `frontend-web/src/pages/Home.jsx` | `handleSearch(e)` | Web UI | Submits current search state | Search |
| `frontend-web/src/context/CartContext.jsx` | `addToCart(product, quantity)` | Web state | Appends product to cart | Add to cart |
| `frontend-mobile/App.js` | `fetchProducts(query)` | Mobile API client | Calls product endpoint, parses JSON or error string | Initial load, search |
| `frontend-mobile/App.js` | `handleSearch()` | Mobile UI | Runs product search | Search |
| `frontend-mobile/App.js` | `renderHome()` | Mobile UI | Displays search box and product list | Initial load, search |
| `backend/server.js` | `app.get("/api/products")` | Backend route | Full list or SQL `LIKE` search | Initial load, search |
| `backend/database.js` | `products` table creation | Database | Defines product fields | All product list/search workflows |

### 5. Function-Level Logic

| Function/Class | Input | Output / Side Effects | Logic and Dependencies |
| --- | --- | --- | --- |
| `Home.fetchProducts(query)` | `query`: string, default `""` | Updates `products` or `errorHtml` state | Sends axios GET to backend. If response is a string containing `<h1>`, treats it as HTML error; otherwise stores response data. Does not URL-encode query. |
| `Home.handleSearch(e)` | Form submit event | Calls `fetchProducts(search)` | Prevents default form submit and uses raw `search` state. |
| `CartContext.addToCart(product, quantity)` | Product object, quantity | Updates cart state | Appends a new line; does not merge duplicates or validate price/quantity. |
| `Mobile.fetchProducts(query)` | `query`: string | Updates `products`, `errorHtml`, `loadingProducts` | Fetches response as text, tries JSON parse, detects backend HTML error if string includes `<h1>`. |
| `GET /api/products` | Optional query `search` | JSON array or HTML error string | If `search` truthy, builds raw SQL string with `LIKE '%search%'`; otherwise runs `SELECT * FROM products`. |

### 6. Domain Testing Analysis

| Input Variable | Domain Partition | Valid/Invalid | Reason |
| --- | --- | --- | --- |
| `search` | Empty string / missing | Valid | Backend returns all products |
| `search` | Existing product substring, e.g. `iPhone` | Valid | Should return matching names |
| `search` | Non-existing substring | Valid | Should return `[]`; frontend lacks explicit empty state |
| `search` | Different case, e.g. `iphone` | Need verification | SQLite `LIKE` case sensitivity depends on configuration/collation |
| `search` | Vietnamese text with accents | Valid | Seed products contain Vietnamese descriptions and names; search only checks name |
| `search` | HTML string, e.g. `<img onerror=...>` | Invalid/security-sensitive | Web renders search term with `dangerouslySetInnerHTML` |
| `search` | SQL metacharacters, e.g. `'` or `%` | Invalid/security-sensitive | Backend interpolates directly into SQL |
| Product `price` | Numeric integer | Valid | Frontend formats with `Number(price).toLocaleString()` |
| Product `price` | String numeric | Tolerated | Product detail intentionally returns price string for even product IDs, and UI casts with `Number()` in list |
| Product `imageUrl` | Empty/null | Need verification | No validation; image may render broken |

### 7. Boundary Testing Analysis

| Variable | Boundary | Expected Behavior |
| --- | --- | --- |
| `search.length` | `0` | Full product list returned |
| `search.length` | `1` | Single-character substring search should return matches or `[]` |
| `search.length` | Very long string | Backend should not crash; current code has no length guard |
| `search` | Contains single quote `'` | Current implementation likely returns HTML database error or enables SQL injection behavior |
| `products.length` | `0` | UI should show empty state; current code shows blank product area |
| `products.length` | Seed count `5` | UI displays five product cards and result count |
| Product `price` | `0`, negative, non-number in DB | UI formatting can show `0`, negative, or `NaN`; backend does not validate for list |

These boundaries matter because the search query crosses directly from UI to SQL, and product list rendering assumes array-like response data and numeric prices.

### 8. Summary

Impact summary:

| Metric | Count / Notes |
| --- | --- |
| Workflows | 3 |
| Affected files | 5 |
| Important functions/classes | 8 |
| Main testing risks | SQL injection/search errors, XSS in web search/error rendering, missing empty/loading states in web, duplicate cart lines |

File and function inventory:

| Feature | File | Function/Class | Purpose |
| --- | --- | --- | --- |
| FR-05 | `frontend-web/src/pages/Home.jsx` | `Home`, `fetchProducts`, `handleSearch` | Web product list/search |
| FR-05 | `frontend-web/src/context/CartContext.jsx` | `addToCart` | Web add-to-cart from product card |
| FR-05 | `frontend-mobile/App.js` | `fetchProducts`, `handleSearch`, `renderHome` | Mobile product list/search |
| FR-05 | `backend/server.js` | `GET /api/products` | Product list/search API |
| FR-05 | `backend/database.js` | `products` table | Product catalog data |

---

## FR-10 Order State Machine

### 1. Overview

Business purpose: control lifecycle of orders from checkout through confirmation, shipping, delivery, or cancellation.

Actors:

| Actor | Entry Point | Permission |
| --- | --- | --- |
| Customer | Web profile, mobile profile | Valid JWT; only own orders for cancellation/listing |
| Admin UI user | Admin orders tab | Client-side admin login check; backend only validates token |
| Backend caller | `/api/orders/*`, `/api/admin/orders/*` | Valid JWT for mutation/listing except public `GET /api/orders/:id` |

Implemented states:

- `pending`
- `confirmed`
- `shipping`
- `delivered`
- `canceled`

Confirmed transition logic:

| Current | Requested | Backend Result |
| --- | --- | --- |
| `pending` | `confirmed` | Allowed |
| `pending` | `canceled` | Allowed |
| `confirmed` | `shipping` | Allowed |
| `confirmed` | `canceled` | Allowed |
| `shipping` | `delivered` | Allowed |
| `canceled` | `delivered` | Allowed in current code |
| `delivered` | Any | Rejected by admin endpoint |
| Any other transition | Any | Rejected |

Need verification:

- Backend does not check `role === "admin"` on admin routes.
- Backend allows `canceled -> delivered`, although requirements describe `canceled` as final.
- Customer cancellation blocks only `delivered` and `canceled`; it allows canceling `shipping`, although mobile UI hides cancel for shipping and web UI shows it.

Components:

| Layer | Component | Responsibility |
| --- | --- | --- |
| Backend API | `POST /api/checkout` | Creates order in `pending` status |
| Backend API | `GET /api/orders/my-orders` | Lists current user's orders |
| Backend API | `PUT /api/orders/:id/cancel` | User cancellation |
| Backend API | `GET /api/admin/orders` | Admin list all orders joined with user names |
| Backend API | `PUT /api/admin/orders/:id/status` | Admin state transition validation and update |
| Web frontend | `Profile()` | Shows user orders and cancel button |
| Admin frontend | `App.updateOrderStatus()` and orders tab | Shows admin actions per state |
| Mobile frontend | `fetchOrders()`, `cancelOrder()`, `statusLabel()` | Mobile order history/cancel |
| Database | `orders` table | Persists state |

### 2. End-to-End Workflows

#### Workflow 1: Checkout Creates Pending Order

Purpose: create a new order in initial state.

Trigger: user confirms checkout from web or mobile.

Actor: authenticated customer.

Flow:

```
User confirms checkout
    ↓
Frontend sends POST /api/checkout with total_amount and optional payload fields
    ↓
authenticateToken verifies JWT
    ↓
Backend reads req.user.id, total_amount, shipping_address
    ↓
SQLite INSERT orders(..., status='pending', ...)
    ↓
Response returns orderId
```

Normal flow:

- Backend inserts `status = "pending"` in `backend/server.js:297-308`.
- `orders.created_at` defaults to current timestamp from schema in `backend/database.js:73-81`.

Alternative flow:

- Web sends `items` and `coupon_id`, but backend ignores both in `backend/server.js:297-303`.
- Mobile sends `items: cart.length > 1 ? cart.slice(0, -1) : cart`, but backend ignores `items` in `frontend-mobile/App.js:390-394`.

Error flow:

- Missing/invalid token returns `401 Unauthorized` or `403 Forbidden` from `authenticateToken()`.
- Database insert error returns `500`.
- Missing `shipping_address` is accepted and stored as `NULL`/undefined because checkout UI does not send it.

#### Workflow 2: Admin Advances or Cancels Order

Purpose: move orders through backend transition rules.

Trigger: admin clicks action button in orders tab.

Actor: admin UI user, but backend accepts any valid JWT.

Flow:

```
Admin opens orders tab
    ↓
Admin UI fetchData() calls GET /api/admin/orders
    ↓
Admin clicks state-specific action button
    ↓
frontend-admin updateOrderStatus(id, status)
    ↓
PUT /api/admin/orders/:id/status { status }
    ↓
Backend loads current status
    ↓
Backend checks transition table
    ↓
SQLite UPDATE orders SET status = ?
    ↓
Admin UI refetches data
```

Normal flow:

- Admin UI buttons map `pending -> confirmed/canceled`, `confirmed -> shipping/canceled`, `shipping -> delivered`, and `canceled -> delivered` in `frontend-admin/src/App.jsx:814-868`.
- Backend validates the same transitions in `backend/server.js:537-551`.

Alternative flow:

- Direct API call can request any string status. Unknown values are rejected unless the current/requested pair matches an allowed condition.

Error flow:

- Nonexistent order returns `404 Order not found`.
- Invalid transition returns `400 Invalid state transition from <current> to <status>`.
- Database update callback does not check `err` in `backend/server.js:559-564`; database failure may still return success. Need verification.

#### Workflow 3: Customer Cancels Own Order

Purpose: let customer cancel an order they own.

Trigger: cancel button in profile/order history.

Actor: authenticated customer.

Flow:

```
Customer clicks Hủy đơn
    ↓
Frontend sends PUT /api/orders/:id/cancel
    ↓
authenticateToken verifies JWT
    ↓
Backend SELECTs order by id AND user_id
    ↓
If status is not delivered/canceled, UPDATE status='canceled'
    ↓
Response success and frontend refetches orders
```

Normal flow:

- Ownership is enforced by `WHERE id = ? AND user_id = ?` in `backend/server.js:321-324`.
- `pending` and `confirmed` are cancelable.

Alternative flow:

- `shipping` is also cancelable by backend because only `delivered` and `canceled` are blocked in `backend/server.js:328-330`.
- Web profile displays cancel button for any status except `delivered` and `canceled` in `frontend-web/src/pages/Profile.jsx`.
- Mobile profile displays cancel only for `pending` and `confirmed` in `frontend-mobile/App.js:960-968`.

Error flow:

- Other user's order or nonexistent order returns `404 Order not found`.
- `delivered` or `canceled` returns `400 Cannot cancel this order.`

#### Workflow 4: View Order State

Purpose: display state to user/admin.

Trigger: profile screen or admin orders tab loads/refetches.

Actor: customer/admin.

Flow:

```
Frontend fetches order list
    ↓
Backend returns order records with status
    ↓
Frontend maps status to Vietnamese label and visual style/action buttons
```

Normal flow:

- User list: `GET /api/orders/my-orders`, filtered by JWT user ID, ordered descending in `backend/server.js:311-318`.
- Admin list: `GET /api/admin/orders`, joined to `users.name` in `backend/server.js:510-522`.
- Status labels exist in web profile, mobile app, and admin app.

Error flow:

- `GET /api/orders/:id` is public and not ownership-protected in `backend/server.js:344-348`; anyone can fetch an order by ID. This is related to state visibility, not the state transition itself.

### 3. Data Flow Analysis

Database entity: `orders` from `backend/database.js:73-81`.

| Field | Type | Meaning | Validation Rule |
| --- | --- | --- | --- |
| `id` | INTEGER PRIMARY KEY AUTOINCREMENT | Order identifier | Generated by SQLite |
| `user_id` | INTEGER | Owner user ID | Set from JWT in checkout; no foreign key constraint |
| `total_amount` | INTEGER | Persisted order total | Taken from client request; no backend recalculation/validation |
| `status` | TEXT DEFAULT `pending` | Order lifecycle state | Transition validation only in admin status endpoint; checkout sets `pending`; cancel endpoint blocks delivered/canceled |
| `shipping_address` | TEXT | Shipping destination | Read from checkout request; current web/mobile checkout does not send it |
| `created_at` | DATETIME DEFAULT CURRENT_TIMESTAMP | Creation timestamp | Generated by SQLite |

Lifecycle:

- Created by `POST /api/checkout`.
- Modified by `PUT /api/orders/:id/cancel` or `PUT /api/admin/orders/:id/status`.
- Consumed by user profile, mobile profile, admin orders tab, and direct order detail endpoint.

### 4. Source Code Impact

| File Path | Function/Class | Layer | Responsibility | Related Workflow |
| --- | --- | --- | --- | --- |
| `backend/server.js` | `app.post("/api/checkout")` | Backend route | Creates pending order | Checkout creates order |
| `backend/server.js` | `app.get("/api/orders/my-orders")` | Backend route | Lists own orders | View state |
| `backend/server.js` | `app.put("/api/orders/:id/cancel")` | Backend route | Customer cancellation | Customer cancel |
| `backend/server.js` | `app.get("/api/admin/orders")` | Backend route | Lists all orders with user name | Admin view |
| `backend/server.js` | `app.put("/api/admin/orders/:id/status")` | Backend route | Admin transition validation and update | Admin transition |
| `frontend-admin/src/App.jsx` | `updateOrderStatus(id, status)` | Admin API client | Sends transition request and refetches | Admin transition |
| `frontend-admin/src/App.jsx` | `statusLabel(status)`, `statusStyle(status)` | Admin UI | Displays state labels/styles | Admin view |
| `frontend-admin/src/App.jsx` | Orders tab render | Admin UI | State-specific action buttons | Admin transition |
| `frontend-web/src/pages/Profile.jsx` | `fetchOrders()`, `cancelOrder(orderId)`, `statusLabel(status)` | Web UI/API client | User order list and cancel | User view/cancel |
| `frontend-mobile/App.js` | `fetchOrders()`, `cancelOrder(orderId)`, `statusLabel(status)` | Mobile UI/API client | Mobile order list and cancel | User view/cancel |
| `backend/database.js` | `orders` table | Database | Persists state machine | All order workflows |

### 5. Function-Level Logic

| Function/Class | Input | Output / Side Effects | Logic and Dependencies |
| --- | --- | --- | --- |
| `POST /api/checkout` | JWT, body `{ total_amount, shipping_address }` | Inserts row into `orders`, returns `{ message, orderId }` | Uses `req.user.id`; ignores `items` and `coupon_id`; always sets status `pending`. |
| `PUT /api/admin/orders/:id/status` | JWT, route `id`, body `{ status }` | Updates `orders.status` if transition is allowed | Loads current status; sets `isValidTransition` for hard-coded pairs; allows `canceled -> delivered`; returns 400 on invalid. |
| `PUT /api/orders/:id/cancel` | JWT, route `id` | Updates own order to `canceled` | Verifies ownership through SQL query; blocks only `delivered` and `canceled`; allows `shipping -> canceled`. |
| `GET /api/admin/orders` | JWT | Returns all orders joined with `users.name` | No role check; orders sorted by descending ID. |
| `frontend-admin.updateOrderStatus(id, status)` | Order ID, target status | Sends PUT; calls `fetchData()` or alerts error | Uses global axios auth header set after admin login. |
| `frontend-web.Profile.cancelOrder(orderId)` | Order ID | Sends PUT cancel; refetches orders | Button visible for all non-delivered/non-canceled statuses. |
| `frontend-mobile.cancelOrder(orderId)` | Order ID | Sends PUT cancel; refetches orders | UI only shows button for pending/confirmed. |

### 6. Domain Testing Analysis

| Input Variable | Domain Partition | Valid/Invalid | Reason |
| --- | --- | --- | --- |
| Current status | `pending` | Valid initial state | Created by checkout |
| Current status | `confirmed` | Valid intermediate | Admin can set from pending |
| Current status | `shipping` | Valid intermediate | Admin can set from confirmed |
| Current status | `delivered` | Valid final | Admin can set from shipping |
| Current status | `canceled` | Valid final by requirement, but not final in implementation | Backend allows canceled to delivered |
| Target status | `confirmed` from `pending` | Valid | Hard-coded backend transition |
| Target status | `shipping` from `confirmed` | Valid | Hard-coded backend transition |
| Target status | `delivered` from `shipping` | Valid | Hard-coded backend transition |
| Target status | `canceled` from `pending`/`confirmed` | Valid | Hard-coded backend transition and user cancel |
| Target status | Unknown string | Invalid | No transition condition matches |
| User cancel ownership | Own order | Valid | SQL filters by `user_id` |
| User cancel ownership | Other user's order | Invalid | Backend returns 404 |
| Auth token | Missing/invalid | Invalid | `authenticateToken()` blocks protected routes |
| Role | Non-admin token on admin endpoint | Invalid by requirement, valid in backend | No role check in backend admin routes |

### 7. Boundary Testing Analysis

| Variable | Boundary | Expected Behavior |
| --- | --- | --- |
| Status transition matrix | Every pair among 5 statuses, 25 pairs total | Only implemented allowed pairs succeed; invalid pairs return 400 |
| `orderId` | Existing order ID | Status operation uses that row |
| `orderId` | Nonexistent ID | 404 |
| `orderId` | Existing but owned by another user for cancel | 404 |
| `status` body | Missing/empty | 400 invalid transition |
| `status` body | Case mismatch, e.g. `Delivered` | 400 invalid transition |
| Terminal state | `delivered -> canceled` | 400 by backend |
| Terminal state | `canceled -> delivered` | Currently 200 by backend; test should catch requirement mismatch |
| Customer cancellation | `shipping` | Currently 200 by backend; mobile UI hides it, web UI exposes it |

These boundaries matter because the feature is a finite-state machine: complete transition-pair coverage is practical and likely to reveal mismatches.

### 8. Summary

Impact summary:

| Metric | Count / Notes |
| --- | --- |
| Workflows | 4 |
| Affected files | 5 |
| Important functions/classes | 10 |
| Main testing risks | Invalid final-state transition, missing admin role enforcement, inconsistent web/mobile cancel visibility, public order detail endpoint, client-controlled totals created by checkout |

File and function inventory:

| Feature | File | Function/Class | Purpose |
| --- | --- | --- | --- |
| FR-10 | `backend/server.js` | `POST /api/checkout` | Creates pending order |
| FR-10 | `backend/server.js` | `PUT /api/admin/orders/:id/status` | Admin state transition |
| FR-10 | `backend/server.js` | `PUT /api/orders/:id/cancel` | User cancellation |
| FR-10 | `backend/server.js` | `GET /api/orders/my-orders`, `GET /api/admin/orders` | Order visibility |
| FR-10 | `frontend-admin/src/App.jsx` | `updateOrderStatus`, orders tab | Admin transition UI |
| FR-10 | `frontend-web/src/pages/Profile.jsx` | `fetchOrders`, `cancelOrder` | Web user history/cancel |
| FR-10 | `frontend-mobile/App.js` | `fetchOrders`, `cancelOrder` | Mobile user history/cancel |
| FR-10 | `backend/database.js` | `orders` table | Order lifecycle persistence |

---

## FR-16 Product Import from CSV

### 1. Overview

Business purpose: let admin import multiple products from a CSV-like file through the admin product management screen.

Actors:

| Actor | Entry Point | Permission |
| --- | --- | --- |
| Admin UI user | Admin Products tab | Client-side admin role check after login |
| Backend caller | `POST /api/admin/import-products` | Any valid JWT in current backend |

Scope confirmed in code:

- Admin UI has a file input, template download link, preview table, and import button.
- CSV file is parsed entirely in the browser using `FileReader` and manual `split("\n")` / `split(",")`.
- Backend receives JSON array under `products`, not a file upload.
- Backend validates only that array is non-empty and each row has a truthy `name`.
- Backend inserts valid rows and reports per-row errors; no transaction/rollback.

Need verification:

- No file extension or MIME validation exists.
- No RFC 4180 parser exists; quoted commas are not supported.
- No positive-price validation exists.
- No all-or-nothing transaction exists.
- No backend admin role check exists.

Components:

| Layer | Component | Responsibility |
| --- | --- | --- |
| Admin frontend | CSV import UI in `frontend-admin/src/App.jsx:337-475` | File selection, parsing, preview, JSON mapping, import API call |
| Backend API | `POST /api/admin/import-products` in `backend/server.js:199-239` | Validates array/name, inserts products, returns result |
| Database | `products` table | Receives inserted product rows |

### 2. End-to-End Workflows

#### Workflow 1: Select CSV and Preview Rows

Purpose: parse a local file and show row preview before import.

Trigger: admin selects a file in the products tab.

Actor: admin UI user.

Flow:

```
Admin selects file
    ↓
onChange stores File in csvFile state
    ↓
FileReader.readAsText(file)
    ↓
reader.onload trims text, splits by newline
    ↓
First line split by comma becomes headers
    ↓
Each remaining line split by comma becomes object values
    ↓
Preview table renders rows
```

Normal flow:

- Header `name,price,description,imageUrl,category_id` maps correctly for simple comma-separated rows without quoted commas.
- Preview count is `importPreview.length`.

Alternative flow:

- Header aliases are accepted during import mapping: `ten`, `Name`, `gia`, `Price`, `mo_ta`, `Description`, `image`, `Image`, `danh_muc`.
- Extra columns remain in preview but are ignored by import mapping.

Error flow:

- Empty file can cause `headers` from `lines[0]` to be `[""]` after `text.trim()`. No explicit UI error is set.
- Quoted commas split into multiple values because parser uses `line.split(",")`.
- File extension is not checked.

#### Workflow 2: Import Valid Preview Rows

Purpose: persist parsed products.

Trigger: admin clicks Import button.

Actor: admin UI user.

Flow:

```
Admin clicks Import
    ↓
Frontend maps preview rows to product objects
    ↓
POST /api/admin/import-products { products: prods }
    ↓
authenticateToken verifies JWT
    ↓
Backend checks products is non-empty array
    ↓
Backend prepares INSERT statement
    ↓
For each row:
      if no name, record error
      else run INSERT into products
    ↓
stmt.finalize responds with inserted count and errors
    ↓
Admin UI displays result and refetches data
```

Normal flow:

- Backend inserts every row with `name`.
- Missing `description` becomes `""`.
- Missing `imageUrl` becomes `""`.
- Missing `category_id` becomes `1`.

Alternative flow:

- `price` can be string, zero, negative, non-numeric, or empty; backend still attempts insert because SQLite column has no constraint.
- A row-level insert error is added to `errors` but does not stop other rows.

Error flow:

- Missing/invalid token returns `401`/`403`.
- Body missing `products`, non-array `products`, or empty array returns `400 { error: "Không có dữ liệu để import" }`.
- Rows without `name` are skipped and reported, but other rows are inserted.

#### Workflow 3: Partial Import With Row Errors

Purpose: import valid rows while reporting invalid ones.

Trigger: import payload contains mixed valid/invalid rows.

Actor: admin UI user or API caller.

Flow:

```
POST products array with some rows missing name
    ↓
Backend loops through rows
    ↓
Rows without name push "Hàng <line>: Thiếu tên sản phẩm"
    ↓
Rows with name are inserted
    ↓
Response inserted < rows.length and errors array
```

Current behavior:

- This is a successful HTTP 200 with partial data changes.
- There is no rollback.

### 3. Data Flow Analysis

Database entity: `products`.

| Field | Type | Meaning | Validation Rule |
| --- | --- | --- | --- |
| `name` | TEXT | Required for import by backend | `if (!row.name)` rejects row |
| `price` | INTEGER | Imported price | No validation in frontend/backend; UI mapping default `0` |
| `description` | TEXT | Imported description | Defaults to `""` in backend |
| `imageUrl` | TEXT | Imported image URL | Defaults to `""` in backend |
| `category_id` | INTEGER | Imported category ID | Frontend `parseInt(...)`; backend defaults falsy value to `1`; no category existence check |

Lifecycle:

- Created from selected file content in admin browser memory.
- Transformed into `importPreview`.
- Transformed again into `prods` JSON payload.
- Inserted into `products`.
- Read back by admin `fetchData()` and product list APIs.

### 4. Source Code Impact

| File Path | Function/Class | Layer | Responsibility | Related Workflow |
| --- | --- | --- | --- | --- |
| `frontend-admin/src/App.jsx` | CSV import `onChange` handler | Admin UI | Reads file, splits text into preview rows | Select/preview |
| `frontend-admin/src/App.jsx` | Import button `onClick` handler | Admin API client | Maps preview rows and calls import endpoint | Import |
| `frontend-admin/src/App.jsx` | `fetchData()` | Admin API client | Refetches products after import | Post-import refresh |
| `backend/server.js` | `app.post("/api/admin/import-products")` | Backend route | Validates payload, inserts rows, returns import report | Import |
| `backend/database.js` | `products` table | Database | Stores imported products | Import |

### 5. Function-Level Logic

| Function/Class | Input | Output / Side Effects | Logic and Dependencies |
| --- | --- | --- | --- |
| CSV file `onChange` handler | Browser `File` from `<input type="file">` | Sets `csvFile`, `importPreview` | Uses `FileReader`; `text.trim().split("\n")`; `headers = lines[0].split(",")`; each data line uses `line.split(",")`. |
| Import button `onClick` | `importPreview` array | Calls backend, sets `importResult`, calls `fetchData()` | Maps possible header aliases; defaults missing price to `0`, category to `1`; sends `{ products: prods }`. |
| `POST /api/admin/import-products` | JWT, body `{ products: rows }` | Inserts product rows, returns `{ message, inserted, errors }` | Rejects empty/non-array payload; prepares insert statement; skips rows missing `name`; inserts the rest; no transaction. |

### 6. Domain Testing Analysis

| Input Variable | Domain Partition | Valid/Invalid | Reason |
| --- | --- | --- | --- |
| File extension | `.csv` | Expected valid by requirement; not checked in code | Test should show any extension is accepted |
| File content | Header exactly `name,price,description,imageUrl,category_id` | Valid | Template uses this header |
| File content | Header aliases such as `Name`, `Price` | Valid in current UI mapping | Alias mapping implemented before API call |
| File content | Missing header `name` but has no alias | Invalid/partial | Rows map to empty `name`, backend reports errors |
| CSV field | Quoted comma in description | Invalid in current parser | Manual split breaks RFC 4180 case |
| `products` body | Missing/non-array/empty array | Invalid | Backend returns 400 |
| Row `name` | Non-empty string | Valid | Required by backend |
| Row `name` | Empty/missing | Invalid row | Backend skips row and reports error |
| Row `price` | Positive number | Expected valid | No validation, but business domain likely expects this |
| Row `price` | `0`, negative, non-numeric, empty | Accepted by current code | No positive-number validation |
| Row `category_id` | Existing category ID | Valid | No backend existence check |
| Row `category_id` | Missing/invalid/NaN | Accepted with caveats | Backend uses `row.category_id || 1`; `NaN` is falsy and becomes `1` |

### 7. Boundary Testing Analysis

| Variable | Boundary | Expected Behavior |
| --- | --- | --- |
| `products.length` | `0` | Backend returns 400 |
| `products.length` | `1` | One row inserted if `name` exists |
| `products.length` | Large N | Backend loops and inserts; no explicit limit or transaction |
| Row line number | First data row | Error message reports `Hàng 2` because header is row 1 |
| `name.length` | `0` | Row skipped and error reported |
| `name.length` | Very long | Insert accepted; no 255-character limit |
| `price` | `0` | Accepted currently |
| `price` | `-1` | Accepted currently |
| `price` | Smallest positive `1` | Accepted |
| `price` | Very large integer | SQLite stores if possible; no app limit |
| `category_id` | Missing | Defaults to `1` |
| CSV newline | Trailing blank lines | `trim()` removes trailing blanks |
| CSV comma | Comma inside quotes | Parsed incorrectly into shifted columns |

These boundaries matter because CSV import mixes parsing, validation, and database writes. The current implementation has partial success semantics, so tests should verify both response report and actual inserted records.

### 8. Summary

Impact summary:

| Metric | Count / Notes |
| --- | --- |
| Workflows | 3 |
| Affected files | 3 |
| Important functions/classes | 5 |
| Main testing risks | No real CSV parser, no file extension check, no price validation, no atomic rollback, partial imports return 200, missing admin role enforcement |

File and function inventory:

| Feature | File | Function/Class | Purpose |
| --- | --- | --- | --- |
| FR-16 | `frontend-admin/src/App.jsx` | CSV input `onChange` | Parse selected file and preview rows |
| FR-16 | `frontend-admin/src/App.jsx` | Import button `onClick` | Map preview rows and call backend |
| FR-16 | `frontend-admin/src/App.jsx` | `fetchData` | Refresh products after import |
| FR-16 | `backend/server.js` | `POST /api/admin/import-products` | Insert imported rows |
| FR-16 | `backend/database.js` | `products` table | Imported product storage |

---

## Mobile App Checkout

### 1. Overview

Business purpose: allow a mobile user to review cart contents, apply coupon, confirm checkout, create an order, record coupon usage, clear local cart, and refresh order history.

Actors:

| Actor | Entry Point | Permission |
| --- | --- | --- |
| Mobile customer | `frontend-mobile/App.js` cart/checkout views | Must be logged in before opening checkout |
| Backend caller | `/api/checkout`, `/api/apply-coupon`, `/api/coupon-usage` | Checkout and usage require JWT; apply-coupon does not require JWT |

Scope confirmed in code:

- Mobile app uses hard-coded LAN API base URL `http://192.168.10.13:3000/api`.
- Cart is local React state, not backend cart.
- Checkout total input is displayed read-only and sourced from `cartTotal`.
- Coupon application uses `cartTotal`.
- Checkout sends `total_amount` computed client-side.
- Backend persists `total_amount` exactly as received; it does not recalculate from products or cart.
- After checkout, mobile clears cart.

Need verification:

- No payment system exists.
- No shipping address is sent by mobile checkout.
- Backend ignores checkout `items` and `coupon_id`.
- Mobile intentionally sends `items: cart.length > 1 ? cart.slice(0, -1) : cart`, but backend ignores `items`.
- Coupon formula in backend appears incorrect for percent discounts: `total_amount * (1 - coupon.discount_value)` rather than `total_amount * discount_value / 100`.

Components:

| Layer | Component | Responsibility |
| --- | --- | --- |
| Mobile frontend | `cartTotal` useMemo | Calculates cart total |
| Mobile frontend | `addToCart()` | Adds/merges product into local cart |
| Mobile frontend | `renderCart()` | Shows cart and checkout entry button |
| Mobile frontend | `openCheckout()` | Requires login and prepares checkout state |
| Mobile frontend | `handleApplyCoupon()` | Calls coupon API |
| Mobile frontend | `handleConfirmCheckout()` | Calls checkout and coupon usage APIs, clears cart |
| Backend API | `POST /api/apply-coupon` | Calculates discount/final amount |
| Backend API | `POST /api/checkout` | Creates pending order |
| Backend API | `POST /api/coupon-usage` | Records coupon use |
| Database | `orders`, `coupon_usage`, `coupons`, `products` | Checkout and coupon data |

### 2. End-to-End Workflows

#### Workflow 1: Add Product to Mobile Cart

Purpose: prepare cart for checkout.

Trigger: user opens product detail and taps add to cart.

Actor: mobile customer/guest.

Flow:

```
User enters quantity and taps add to cart
    ↓
normalizeQuantity() converts invalid/non-positive input to 1
    ↓
addToCart() checks if product already exists in cart
    ↓
If existing: increments quantity
Else: appends product line
    ↓
Cart total recalculates via useMemo
```

Normal flow:

- `normalizeQuantity()` returns positive integer or `1` in `frontend-mobile/App.js:129-132`.
- `addToCart()` merges by product ID and increments quantity in `frontend-mobile/App.js:134-152`.
- `cartTotal` is `sum(item.price * item.quantity)` in `frontend-mobile/App.js:75-77`.

Alternative/error flow:

- Product price may be string for even product IDs from backend detail endpoint; multiplication coerces numeric strings.
- Non-numeric price can produce `NaN`.

#### Workflow 2: Open Checkout

Purpose: enforce login and render checkout review.

Trigger: user taps "Tiến hành thanh toán" in cart.

Actor: mobile customer.

Flow:

```
User taps checkout
    ↓
openCheckout()
    ↓
If no user: alert and navigate to login
If user exists: reset coupon/success state, set editableTotal, switch view to checkout
    ↓
Checkout renders product lines, read-only total, coupon input, confirm button
```

Normal flow:

- `openCheckout()` blocks unauthenticated users in `frontend-mobile/App.js:342-354`.
- Checkout total field is read-only with `editable={false}` in `frontend-mobile/App.js:684-690`.

Alternative flow:

- Empty cart checkout button is not visible because cart view renders empty-state instead of checkout action.

#### Workflow 3: Apply Coupon in Mobile Checkout

Purpose: calculate discount and final amount before order creation.

Trigger: user enters coupon code and taps apply.

Actor: logged-in mobile customer.

Flow:

```
User enters coupon code
    ↓
handleApplyCoupon()
    ↓
POST /api/apply-coupon { code: uppercase, total_amount: cartTotal, user_id }
    ↓
Backend validates code, active flag, threshold, expiry, usage count if user_id exists
    ↓
Backend calculates discount_amount and final_amount
    ↓
Mobile displays success or error
```

Normal flow:

- Mobile uppercases code via `couponCode.trim().toUpperCase()` in `frontend-mobile/App.js:365-368`.
- Backend checks active coupon by exact code in `backend/server.js:369-377`.
- Backend requires `total_amount > coupon.min_order_amount` in `backend/server.js:379`, not `>=`.
- Backend checks expiry and usage count when `user_id` exists.

Error flow:

- Blank coupon does nothing because the button is disabled and function returns early.
- Unknown/inactive code returns 404.
- Expired coupon returns 400.
- Threshold failure returns 400.
- Usage limit reached returns 400.
- Percent discount calculation can produce negative/incorrect values because `SAVE10` yields `total * (1 - 10)`.

#### Workflow 4: Confirm Mobile Checkout

Purpose: create order, optionally record coupon usage, clear cart, refresh orders.

Trigger: user taps "Xác Nhận Thanh Toán".

Actor: logged-in mobile customer.

Flow:

```
User taps confirm
    ↓
handleConfirmCheckout() sets checkoutLoading
    ↓
finalAmount = couponResult.final_amount or cartTotal
    ↓
POST /api/checkout with Authorization and body:
  items, total_amount, coupon_id
    ↓
Backend authenticateToken()
    ↓
Backend inserts orders row with user_id, total_amount, status='pending', shipping_address from body
    ↓
If coupon applied, mobile POSTs /api/coupon-usage
    ↓
Mobile sets success, clears cart/coupon state, fetches orders
```

Normal flow:

- Mobile checkout request is in `frontend-mobile/App.js:380-395`.
- Backend checkout insert is in `backend/server.js:297-308`.
- Coupon usage request is in `frontend-mobile/App.js:399-407`.
- Cart is cleared in `frontend-mobile/App.js:410-414`.

Alternative flow:

- Without coupon, `total_amount = cartTotal`.
- With coupon, `total_amount = couponResult.final_amount`.

Error flow:

- Missing/expired token makes backend return 401/403; mobile alerts error.
- Backend DB error returns 500.
- If checkout succeeds but coupon usage request fails, mobile does not check `response.ok` for coupon usage; it proceeds to success. Need verification for failed usage persistence.

#### Workflow 5: View Mobile Order History After Checkout

Purpose: show created order in profile/history.

Trigger: checkout success calls `fetchOrders(token)` or user opens profile.

Actor: logged-in mobile customer.

Flow:

```
Mobile calls GET /api/orders/my-orders
    ↓
Backend returns own orders by JWT user ID
    ↓
Mobile renders order cards with total and status label
```

Normal flow:

- Mobile maps statuses through `statusLabel()` in `frontend-mobile/App.js:331-340`.
- Cancel button appears only for `pending` or `confirmed` in `frontend-mobile/App.js:960-968`.

### 3. Data Flow Analysis

Entities:

| Entity | Role |
| --- | --- |
| `products` | Source of product ID, name, price used in mobile cart |
| Mobile `cart` state | Local, client-side checkout basket |
| `coupons` | Coupon validation/discount source |
| `coupon_usage` | Records coupon use after checkout |
| `orders` | Persisted checkout result |

Important fields:

| Field | Type | Meaning | Validation Rule |
| --- | --- | --- | --- |
| Cart `item.id` | number/string | Product identity | Used for merge in `addToCart()` |
| Cart `item.price` | number/string | Unit price | No validation; used in multiplication |
| Cart `item.quantity` | number/string | Quantity | `normalizeQuantity()` enforces positive integer on add; cart edit adds 1 to parsed value |
| `cartTotal` | number | Sum of line totals | Client-side only |
| Coupon `code` | TEXT | Coupon identifier | Backend exact match active coupon |
| Coupon `min_order_amount` | INTEGER | Minimum threshold | Backend uses strict `>` |
| Coupon `max_uses_per_user` | INTEGER | Usage cap | Checked only when `user_id` supplied |
| Order `total_amount` | INTEGER | Persisted paid total | Taken from client request |
| Order `status` | TEXT | Starts as `pending` | Set by backend checkout |
| Order `shipping_address` | TEXT | Shipping destination | Mobile checkout does not send it |

Lifecycle:

- Product data is fetched and placed into cart state.
- Cart state is transformed into `cartTotal`.
- Coupon result is calculated server-side using client total.
- Order is created from client final amount.
- Coupon usage is inserted in a separate request after checkout.
- Mobile clears cart locally; backend cart is not involved.

### 4. Source Code Impact

| File Path | Function/Class | Layer | Responsibility | Related Workflow |
| --- | --- | --- | --- | --- |
| `frontend-mobile/App.js` | `cartTotal` useMemo | Mobile state | Calculates total | Cart, checkout |
| `frontend-mobile/App.js` | `normalizeQuantity(value)` | Mobile validation | Converts quantity to positive integer or 1 | Add to cart |
| `frontend-mobile/App.js` | `addToCart(selectedProduct, selectedQuantity)` | Mobile state | Adds or merges product lines | Add to cart |
| `frontend-mobile/App.js` | `renderCart()` | Mobile UI | Shows cart, quantity editor, checkout button | Open checkout |
| `frontend-mobile/App.js` | `openCheckout()` | Mobile flow guard | Requires login and enters checkout view | Open checkout |
| `frontend-mobile/App.js` | `handleApplyCoupon()` | Mobile API client | Calls coupon calculation | Apply coupon |
| `frontend-mobile/App.js` | `handleConfirmCheckout()` | Mobile API client | Creates order, records coupon usage, clears cart | Confirm checkout |
| `frontend-mobile/App.js` | `fetchOrders()` | Mobile API client | Refreshes order history | Post-checkout history |
| `backend/server.js` | `POST /api/apply-coupon` | Backend route | Coupon validation/calculation | Apply coupon |
| `backend/server.js` | `POST /api/checkout` | Backend route | Order creation | Confirm checkout |
| `backend/server.js` | `POST /api/coupon-usage` | Backend route | Coupon usage persistence | Confirm checkout |
| `backend/database.js` | `orders`, `coupons`, `coupon_usage`, `products` | Database | Checkout and coupon persistence | All checkout workflows |

### 5. Function-Level Logic

| Function/Class | Input | Output / Side Effects | Logic and Dependencies |
| --- | --- | --- | --- |
| `cartTotal` | `cart` state | Number total | Reduces `item.price * item.quantity`. |
| `normalizeQuantity(value)` | Any value from quantity input | Positive integer or `1` | Uses `parseInt`; rejects non-finite and `<= 0`. |
| `addToCart(selectedProduct, selectedQuantity)` | Product object, quantity | Updates local cart | Merges by product ID; increments existing quantity. |
| `renderCart()` quantity editor | Text input | Updates local cart quantity | Parses text; if valid positive, stores `parsed + 1`, otherwise `1`. This means entering `1` stores quantity `2`. |
| `openCheckout()` | Current `user`, `cartTotal` | Switches view or navigates to login | Blocks unauthenticated checkout; resets coupon state. |
| `handleApplyCoupon()` | `couponCode`, `cartTotal`, `user.id` | Updates `couponResult` or `couponError` | Sends coupon calculation request; uses uppercase trimmed code. |
| `handleConfirmCheckout()` | Cart, token, coupon result | Creates order; maybe records coupon usage; clears cart | Sends client-computed final amount; ignores coupon usage response status; fetches orders after success. |
| `POST /api/apply-coupon` | `{ code, total_amount, user_id }` | Discount response or error | Validates code, threshold, expiry, usage count. Percent formula currently uses `total * (1 - discount_value)`. |
| `POST /api/checkout` | JWT, `{ total_amount, shipping_address }` | Inserts pending order | No backend recalculation from cart/products. |
| `POST /api/coupon-usage` | JWT, `{ coupon_id }` | Inserts `coupon_usage` row | Does not validate coupon exists before insert. |

### 6. Domain Testing Analysis

| Input Variable | Domain Partition | Valid/Invalid | Reason |
| --- | --- | --- | --- |
| User auth state | Logged in | Valid | `openCheckout()` allows checkout |
| User auth state | Not logged in | Invalid for checkout | Mobile alerts and switches to login |
| Cart | Empty | Invalid by UI path | Checkout button hidden in empty cart |
| Cart | One product | Valid | Checkout sends full cart in `items` |
| Cart | Multiple products | Valid by UI, but sent `items` omits last item | Backend ignores `items`; domain issue still testable via API payload inspection |
| Quantity input on add | Positive integer string | Valid | Normalized to parsed integer |
| Quantity input on add | `0`, negative, non-numeric | Invalid but coerced | Normalized to `1` |
| Quantity edit in cart | Positive integer string | Valid-intended, but implementation stores `parsed + 1` | Boundary-sensitive defect |
| Coupon code | Existing active code | Valid if threshold/expiry/usage pass | Backend returns discount result |
| Coupon code | Blank | Invalid/no-op | Button disabled/function returns |
| Coupon code | Unknown/inactive | Invalid | 404 |
| Coupon code | Expired | Invalid | 400 |
| Coupon threshold | `total_amount > min_order_amount` | Valid in code | Strict greater-than |
| Coupon threshold | `total_amount == min_order_amount` | Invalid in code | Boundary mismatch if expected `>=` |
| Coupon usage | Under max uses | Valid | Usage count check passes |
| Coupon usage | At max uses | Invalid | Backend returns 400 |
| Checkout total | Client-computed normal number | Valid | Persisted as order total |
| Checkout total | Negative/zero via direct API | Accepted by backend if authenticated | No backend validation |

### 7. Boundary Testing Analysis

| Variable | Boundary | Expected Behavior |
| --- | --- | --- |
| Cart quantity add | `1` | Quantity becomes 1 when adding from product detail |
| Cart quantity add | `0` | Coerced to 1 |
| Cart quantity add | `-1` | Coerced to 1 |
| Cart quantity add | Non-numeric | Coerced to 1 |
| Cart quantity edit | Enter `1` | Current implementation stores 2 |
| Cart line count | `1` | Checkout sends one item |
| Cart line count | `2` | Checkout sends only first item due to `slice(0, -1)`, but backend ignores it |
| `cartTotal` | `0` | Checkout can be attempted by direct state/API; backend accepts total 0 |
| Coupon threshold | `min_order_amount - 1` | Backend rejects |
| Coupon threshold | `min_order_amount` | Backend rejects due to strict `>` |
| Coupon threshold | `min_order_amount + 1` | Backend continues validation |
| Percent coupon | `SAVE10` on 300001 | Backend calculates negative discount/final amount due to formula |
| Coupon usage count | `max_uses_per_user - 1` | Apply succeeds |
| Coupon usage count | `max_uses_per_user` | Apply fails |
| Token | Missing on checkout | 401 |
| Token | Invalid on checkout | 403 |
| Coupon usage persistence | Checkout succeeds, usage insert fails | Mobile still shows success because usage response is not checked |

These boundaries matter because mobile checkout combines client-side arithmetic, coupon server logic, and order persistence. The backend trusts the submitted total, so API-level tests should complement UI tests.

### 8. Summary

Impact summary:

| Metric | Count / Notes |
| --- | --- |
| Workflows | 5 |
| Affected files | 3 |
| Important functions/classes | 12 |
| Main testing risks | Client-trusted order total, missing shipping address, coupon percent formula, strict coupon threshold, quantity edit off-by-one, coupon usage not transactional with checkout |

File and function inventory:

| Feature | File | Function/Class | Purpose |
| --- | --- | --- | --- |
| Mobile Checkout | `frontend-mobile/App.js` | `cartTotal`, `normalizeQuantity`, `addToCart` | Mobile cart calculation and validation |
| Mobile Checkout | `frontend-mobile/App.js` | `renderCart`, `openCheckout`, `renderCheckout` | Mobile checkout UI flow |
| Mobile Checkout | `frontend-mobile/App.js` | `handleApplyCoupon`, `handleConfirmCheckout` | Coupon and order API calls |
| Mobile Checkout | `frontend-mobile/App.js` | `fetchOrders`, `statusLabel` | Post-checkout order history |
| Mobile Checkout | `backend/server.js` | `POST /api/apply-coupon` | Coupon validation/calculation |
| Mobile Checkout | `backend/server.js` | `POST /api/checkout` | Order creation |
| Mobile Checkout | `backend/server.js` | `POST /api/coupon-usage` | Coupon usage persistence |
| Mobile Checkout | `backend/database.js` | `orders`, `coupons`, `coupon_usage`, `products` | Data persistence |

