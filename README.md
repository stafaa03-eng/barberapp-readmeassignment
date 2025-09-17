<!-- Logo / Banner -->
<p align="center">
  <img src="assets/logoreal.png" alt="BarberApp logo" width="720">
</p>

<h1 align="center">BarberApp</h1>
<p align="center">A scheduling and client management app for barbers, with an online booking site for customers.</p>

---

## Table of Contents
- [Overview](#overview)
- [Quickstart](#quickstart)
- [Features](#features)
- [Architecture](#architecture)
- [Usage Examples](#usage-examples)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)

---

## Overview
BarberApp provides two interfaces:
- A **website** where customers can view availability and book appointments.
- A **mobile app** for barbers to manage clients, edit schedules, and add bookings.

All data is synced to a single Postgres database through a Node.js API with Clerk authentication.

---

## Quickstart
This section shows how a developer could clone and run the project locally.

> **Requirements:** Node.js (18+), npm, a Supabase project, a Clerk application.

1) **Clone the repository**
```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
```

```bash
Configure environment variables in .env.local (adjust to your structure):

# auth
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_key
CLERK_SECRET_KEY=your_clerk_secret

# data
SUPABASE_URL=your_supabase_url
SUPABASE_ANON_KEY=your_supabase_key
DATABASE_URL=postgres_connection_string

# api base
NEXT_PUBLIC_API_BASE_URL=http://localhost:3001
```

```bash
Install and run:

# API
cd api && npm install && npm run dev
# Website
cd ../web && npm install && npm run dev
# Open
open http://localhost:3000
```

Create schema (example):

```sql
create table barbers(
  id uuid primary key default gen_random_uuid(),
  name text,
  contact text
);
create table clients(
  id uuid primary key default gen_random_uuid(),
  barber_id uuid references barbers(id),
  name text,
  phone text
);
create table schedules(
  id uuid primary key default gen_random_uuid(),
  barber_id uuid references barbers(id),
  day date,
  slots jsonb
);
create table bookings(
  id uuid primary key default gen_random_uuid(),
  client_id uuid references clients(id),
  barber_id uuid references barbers(id),
  slot timestamptz,
  status text
);
```

GIFs  
Embed short demos (keep them zoomed and under ~10s):

![Install demo](assets/gifs/install.gif "Install and run")  
![Booking demo](assets/gifs/booking.gif "Customer booking flow")

---

## Features
- Customer website: browse availability and book appointments.  
- Barber app: manage clients, edit schedules, and create bookings.  
- Authentication: Clerk for secure sign-in and session management.  
- Database: Postgres (via Supabase) as the single source of truth.  

**Spotlight: Schedule editing**  
Barber updates availability in the app → API verifies session → writes schedules → website shows updated slots.

---


![Architecture diagram](assets/techcomdiagram.drawio.png "System containers and flows")

**Components**
- Website: React on Vercel  
- Mobile App: React Native  
- API: Node.js on Vercel  
- Auth: Clerk  
- DB: Supabase Postgres  
- Mobile secure store for session tokens  

---

## Usage Examples

**Create booking (cURL)**

```bash
curl -X POST "$NEXT_PUBLIC_API_BASE_URL/bookings" \
  -H "Authorization: Bearer <CLERK_JWT>" \
  -H "Content-Type: application/json" \
  -d '{"clientId":"<id>","barberId":"<id>","slot":"2025-09-20T15:00:00Z"}'
```

**Fetch availability (JavaScript)**

```javascript
const res = await fetch(
  `${process.env.NEXT_PUBLIC_API_BASE_URL}/availability?barberId=${barberId}`
);
const data = await res.json();
```

**Edit schedule (JavaScript)**

```javascript
await fetch(`/schedules/${barberId}`, {
  method: 'PUT',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    day: '2025-09-20',
    slots: ['15:00','16:00']
  })
});
```

---

## FAQ

**Does it support multiple barbers?**  
Yes. Each client, schedule, and booking is linked to a barber_id.

**Do I need Clerk to run this?**  
Yes, authentication depends on Clerk. Another JWT provider could replace it with modifications.

**How do I deploy?**  
Deploy the website and API to Vercel. Use Supabase for the database. Set the environment variables in Vercel’s dashboard.

**Accessibility?**  
All images and GIFs in this README include alt text.

---

## Contributing
Pull requests are welcome. Open an issue first for major changes.

---

## License
MIT
