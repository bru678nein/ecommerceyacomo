# Yacomo — E-Commerce Platform

Full-stack e-commerce application with a Spring Boot backend and a separate web frontend.
Covers the full purchase flow: catalogue, cart, checkout with online payments, order
management and an administration panel.

**Live:** https://proyecto-yacomo.vercel.app

> `TODO: one or two sentences on what Yacomo actually sells and who it was built for.
> A reader who lands here should know what the product is before they read the stack.`

---

## Stack

**Backend** — Java 21, Spring Boot 3.5, Gradle

| Concern | Choice |
| --- | --- |
| Web layer | Spring Web (REST) |
| Security | Spring Security + JWT (jjwt) |
| Validation | Spring Validation (Jakarta Bean Validation) |
| Relational persistence | Spring Data JPA / Hibernate over MySQL |
| Document persistence | Spring Data MongoDB |
| Auditing | Spring Data Envers (entity revision history) |
| Payments | Mercado Pago Java SDK |
| Media storage | Cloudinary |
| Email | Spring Mail |
| PDF generation | OpenPDF |
| Boilerplate | Lombok |

**Frontend** — `TODO: React? Next? Vite? State management? Styling?`
Deployed on Vercel.

**API collection** — Postman collection under `JSON_POSTMAN/`.

---

## Architecture

```
PROYECTO_YACOMO/
├── Backend/BackendE_Commerce/   Spring Boot API
├── Frontend/                    Web client
├── JSON_POSTMAN/                Postman collection for the API
├── productos_seed.sql           Product catalogue seed data
└── DATOS-PRUEBAS-MP.txt         Mercado Pago sandbox test data
```

> `TODO: describe the backend package layout — controllers / services / repositories /
> entities, or whatever the actual structure is. Two or three lines is enough.`

---

## Design decisions

This is the section worth reading. Each of these was a choice with a cost.

### Stateless authentication with JWT

Authentication is handled with Spring Security and signed JSON Web Tokens rather than
server-side sessions. The API keeps no session state, so any instance can serve any
request and horizontal scaling costs nothing at the auth layer.

The trade-off is revocation: a signed token is valid until it expires, so logging out
does not invalidate it server-side.

> `TODO: say how you handled that — short-lived access tokens, a refresh flow, a
> denylist, or an accepted limitation. Be honest either way; "we accepted it for this
> scope" is a perfectly good answer.`

### Polyglot persistence: MySQL and MongoDB side by side

Transactional data — users, orders, payments, stock — lives in MySQL through Spring Data
JPA, because it needs foreign keys, transactions and constraints the database itself can
enforce.

> `TODO: what went into MongoDB, and why it did not belong in MySQL? This is the most
> interesting decision in the project and it needs one honest paragraph. If the real
> answer is "we wanted to try it", say that — it reads better than a rationalisation.`

The cost is operational: two databases to run, back up and keep consistent, and no
foreign keys across the boundary.

### Entity revision history with Spring Data Envers

Envers records a revision of an audited entity on every change, so the state of an order
or a product at any past moment can be reconstructed.

In an application that handles money this is not a nice-to-have: when a customer disputes
a price or an order status, the answer has to come from recorded history rather than from
whatever the row says today.

The cost is write volume and schema weight — every audited table gets a shadow table.

### Payments through Mercado Pago webhooks

Payment state is not decided by the frontend. Mercado Pago notifies the backend, and the
backend reconciles that notification against its own order records before advancing
anything.

> `TODO: describe your webhook handling — do you verify the notification signature? Is
> the handler idempotent if the same event arrives twice? Webhook delivery is not ordered
> and not exactly-once, so whatever you did here is worth stating explicitly.`

### Media offloaded to Cloudinary

Product images are stored in Cloudinary rather than on the application server or in the
database, so the API stays stateless and image delivery does not compete with API traffic.

---

## Running it locally

### Requirements

- JDK 21
- MySQL `TODO: version`
- MongoDB `TODO: version`
- Node `TODO: version` for the frontend

### Configuration

The backend reads its configuration from environment variables. Copy the example file and
fill it in:

```bash
cp .env.example .env
```

| Variable | Description |
| --- | --- |
| `DB_URL` | MySQL JDBC connection string |
| `DB_USERNAME` | MySQL user |
| `DB_PASSWORD` | MySQL password |
| `MONGODB_URI` | MongoDB connection string |
| `JWT_SECRET` | Signing key for JSON Web Tokens |
| `MP_ACCESS_TOKEN` | Mercado Pago access token (use a sandbox token locally) |
| `CLOUDINARY_URL` | Cloudinary credentials |
| `MAIL_USERNAME` / `MAIL_PASSWORD` | SMTP credentials |

> `TODO: replace this table with the real variable names from application.properties.
> And make sure no real values are committed anywhere in the repository.`

### Backend

```bash
cd Backend/BackendE_Commerce
./gradlew bootRun
```

Seed the catalogue:

```bash
mysql -u <user> -p <database> < productos_seed.sql
```

### Frontend

```bash
cd Frontend
npm install
npm run dev
```

### Tests

```bash
./gradlew test
```

---

## API

The Postman collection in `JSON_POSTMAN/` covers the available endpoints.

> `TODO: a short table of the main endpoint groups — auth, products, cart, orders,
> payments, admin — with one line each. Nobody will import a Postman collection just to
> find out what the API does.`

Mercado Pago sandbox test cards are in `DATOS-PRUEBAS-MP.txt`.

---

## Team

Built as a team project.

> `TODO: list the team and who did what. Say plainly which parts you wrote — in your case
> most of the backend. Being specific here helps you and is fairer to everyone else.`

---

## Known limitations

> `TODO: three or four honest lines. Things you would do differently, what is missing,
> what does not scale yet. A README that admits limitations reads as more competent than
> one that claims none — and in an interview it is the section people ask about.`
