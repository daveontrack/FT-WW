# 🪑 TF Wood Works — Furniture Ordering Platform

A full-stack web application that helps a small furniture business manage products and customer orders digitally.
Admins post furniture and receive order notifications, while clients browse the catalog and place orders with just their **name and phone number** — no account required.

Built with an Agile approach: a working MVP first, expanding sprint by sprint.

---

## ✨ Features

### 🔐 Admin (protected)
- Secure email + password login (JWT, bcrypt-hashed passwords)
- Role-based access — only admins can reach `/admin/*` pages and admin APIs
- Dashboard overview — furniture count, pending orders, unread notifications
- **Furniture CRUD** — add / edit / delete items with name, description, price (ETB), category, image URL and availability
- **Order management** — view all client orders, search/filter, mark as *completed* or *cancelled*
- **Order notifications** — every new client order creates a notification showing the selected furniture, order date and the client's name + phone
- Two seeded admin accounts (father & partner)

### 👥 Client (public)
- Browse the catalog without signing in
- Filter by category, sort collections, view product details
- Place an order with just **full name + phone number**
- Wishlist — save favorite pieces (localStorage, no login)
- Call or email the admins directly from the site

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 16 (App Router), TypeScript, Tailwind CSS 4 |
| Backend | Node.js, Express 5, TypeScript |
| Database | **Supabase (PostgreSQL)** via Prisma 7 driver adapter |
| Auth | JWT + bcrypt |
| Tooling | ts-node-dev, ESLint |

> Originally MongoDB was planned for notifications/logs; the project now uses **Supabase PostgreSQL for everything** through Prisma.

---

## 🗄 Database Schema

```
Admin (id, name, email, password, phone, role, createdAt)
 1:N → Furniture

Category (id, name)
 1:N → Furniture

Furniture (id, name, description, price, imageUrl, available,
           adminId → Admin, categoryId → Category, timestamps)
 1:N → Order

Order (id, customerName, customerPhone, status PENDING|COMPLETED|CANCELLED,
       furnitureId → Furniture, createdAt)
 1:1 → Notification

Notification (id, message, isRead, orderId → Order, createdAt)
```

---

## 📡 API Endpoints

| Method | Endpoint | Access | Description |
|---|---|---|---|
| POST | `/api/auth/login` | public | Admin login → JWT token |
| GET | `/api/categories` | public | List categories |
| POST | `/api/categories` | admin | Create category |
| GET | `/api/furniture` | public | List furniture (`?category=Chairs`) |
| GET | `/api/furniture/:id` | public | Single item |
| POST | `/api/furniture` | admin | Create item |
| PUT | `/api/furniture/:id` | admin | Update item |
| DELETE | `/api/furniture/:id` | admin | Delete item |
| POST | `/api/orders` | public | Client places order (name + phone) |
| GET | `/api/orders` | admin | List orders (`?status=PENDING`) |
| PATCH | `/api/orders/:id/status` | admin | Update order status |
| GET | `/api/notifications` | admin | List notifications |
| PATCH | `/api/notifications/:id/read` | admin | Mark one as read |
| POST | `/api/notifications/read-all` | admin | Mark all read |

Admin endpoints require `Authorization: Bearer <token>`.

---

## 🚀 Getting Started

### Prerequisites
- Node.js ≥ 20
- A [Supabase](https://supabase.com) project (or any PostgreSQL database)

### Backend

```bash
cd backend
npm install
```

Create `backend/.env`:

```env
PORT=5000
JWT_SECRET="change_me"

# Session pooler URL (recommended for Prisma driver adapter)
DATABASE_URL="postgresql://postgres.<ref>:<password>@aws-1-eu-west-1.pooler.supabase.com:5432/postgres"
DIRECT_URL="postgresql://postgres.<ref>:<password>@aws-1-eu-west-1.pooler.supabase.com:5432/postgres"
```

> ⚠️ URL-encode special characters in your password (e.g. `@` → `%40`).

Push the schema and seed default data (admins + categories + sample catalog):

```bash
npx prisma db push
node prisma/seed.cjs
```

Seed script creates:
- Default admin accounts *(edit emails/passwords in `prisma/seed.cjs` before running)*
- Six categories — Chairs, Tables, Sofas, Dining, Beds, Kitchen
- 20 sample furniture pieces

Start the API:

```bash
npm run dev        # http://localhost:5000
```

### Frontend

```bash
cd client
npm install
npm run dev        # http://localhost:3000
```

Optionally add `client/.env.local` if the backend runs elsewhere:

```env
NEXT_PUBLIC_API_URL=http://localhost:5000/api
```

---

## 📁 Project Structure

```
FT-WW/
├── backend/
│   ├── prisma/
│   │   ├── schema.prisma        # Supabase schema
│   │   └── seed.cjs             # Default admins, categories & catalog
│   └── src/
│       ├── app.ts               # Express app + route wiring
│       ├── server.ts
│       ├── lib/prisma.ts        # PrismaClient + pg adapter
│       ├── middleware/          # JWT auth guard
│       └── routes/              # auth, furniture, category, order, notification
├── client/
│   ├── app/
│   │   ├── page.tsx             # Home (live data)
│   │   ├── collections/         # Full catalog w/ filters
│   │   ├── product/[id]/        # Product detail
│   │   ├── order/               # Public order form
│   │   ├── wishlist/            # Saved items
│   │   ├── category/[slug]/
│   │   └── admin/               # login, dashboard, furniture, orders, notifications
│   ├── components/
│   ├── lib/api.ts               # Fetch helper + token storage
│   └── lib/wishlist.ts          # localStorage wishlist
└── 
```

---

## 🔒 Security Notes

- `.env` files are git-ignored — never commit credentials
- Passwords are hashed with bcrypt; sessions use signed JWTs (8h expiry)
- All admin APIs return `401` without a valid token

---

## 🗺 Roadmap (future sprints)

- Image upload via Supabase Storage
- Payments integration
- Sales analytics dashboard
- Stock/quantity tracking

---

## Status

🚧 MVP complete — core ordering loop is live end-to-end.
