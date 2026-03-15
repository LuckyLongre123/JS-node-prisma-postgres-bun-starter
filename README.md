# Prisma + PostgreSQL + Node.js Setup (Without TypeScript) using ES Modules and Bun

This guide explains how to integrate **Prisma ORM**, **PostgreSQL**, **Node.js**, and **Neon** using **ES Modules**, without TypeScript.

 ### **Note:** This guide uses **Bun** for installing packages because it is very fast. Bun is optional — if you don't have it, you can use **npm** and **npx** instead.
---

# Step 1 — Initialize Node.js Project

Initialize a Node.js project:

```bash
npm init -y
```

Open **package.json** and make the following changes:

1. Change module type:

```json
"type": "module"
```

2. Add a development script:

```json
"scripts": {
  "dev": "nodemon index.js"
}
```

---

# Step 2 — Install Required Packages

Install the essential dependencies using **Bun**:

```bash
bun add @prisma/adapter-pg @prisma/client dotenv pg prisma
```

Install development dependency:

```bash
bun add --dev nodemon
```

---

# Step 3 — Initialize Prisma

Run the Prisma initialization command:

```bash
bunx prisma init
```

This command creates the following files:

```
prisma/schema.prisma
prisma.config.ts
.env
```

Add your **PostgreSQL connection string** inside the `.env` file:

```env
DATABASE_URL=your_database_connection_string
```

## Getting a PostgreSQL Connection String from Neon

If you don't have a PostgreSQL database, you can quickly create one using **Neon**, a serverless PostgreSQL platform.

### Step I — Create an Account

Go to:

```
https://neon.com
```

Sign up using **GitHub, Google, or email**.

---

### Step II — Create a New Project

After logging in:

1. Click **Create Project**
2. Enter a project name
3. Select a region close to you
4. Click **Create Project**

Neon will automatically create a **PostgreSQL database instance**.

---

### Step III — Copy the Connection String

Inside your project dashboard:

1. Go to **Dashboard**
2. click **Connect** button, the window opens
3. in window, locate *Copy snippet* and click, the connection string copies to your clipboard

It will look similar to this:

```env
postgresql://username:password@hostname/database?sslmode=require
```

---

### Step IV — Add It to Your `.env` File

Paste the connection string into your `.env` file:

```env
DATABASE_URL=postgresql://username:password@hostname/database?sslmode=require
```

Now Prisma can connect to your **Neon PostgreSQL database**.

---

# Step 4 — Configure Prisma for JavaScript

Prisma generates **TypeScript configuration by default**, but we will still use the same configuration with JavaScript.

### Update `prisma.config.ts`

```javascript
import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
  },
  datasource: {
    url: env("DATABASE_URL"),
  },
});
```

---

### Update `schema.prisma`

Modify the **generator** and **datasource** configuration.

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
}
```

---

# Step 5 — Define Your Database Models

Create your schema models inside:

```
prisma/schema.prisma
```

Example:

```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
}
```

You can create additional models as needed.

---

# Step 6 — Run Migration and Generate Prisma Client

Run the migration command: **(this will take time)**

```bash
bunx prisma migrate dev --name init
```

This command:

* Converts your **Prisma schema into SQL queries**
* Success message after successfully run this command: *Your database is now in sync with your schema.*
* Creates migration files inside: => **./prisma/migrations**

Now generate the Prisma Client:

```bash
bunx prisma generate
```
* Success message after successfully run this command: *Generated Prisma Client ...*

---

# Step 7 — Create Prisma Client in Your Code

### Create `db.js` in the *lib* Folder

Add the following code:

```javascript
import dotenv from "dotenv";
dotenv.config();

import pkg from "@prisma/client";
const { PrismaClient } = pkg;

import { PrismaPg } from "@prisma/adapter-pg";

const adapter = new PrismaPg({
  connectionString: process.env.DATABASE_URL,
});

const prisma = new PrismaClient({ adapter });

export async function checkDB() {
  try {
    await prisma.$connect(); // connect without querying data
    console.log("✅ Database connected successfully");
  } catch (error) {
    console.error("❌ Database connection failed:", error);
  } finally {
    await prisma.$disconnect();
  }
}

export default prisma;
```

---

### Create `index.js` in the Root Directory

Add the following code:

```javascript
import { checkDB } from "./lib/db.js";

checkDB();
```

---

# Step 8 — Run the Project

Start the development server:

```bash
bun run dev
```

If everything is configured correctly, you should see:

```
✅ Database connected successfully
```

---

# 🎉 Congratulations

Your **Prisma + PostgreSQL + Node.js setup (without TypeScript)** using **ES Modules and Bun** is now successfully configured.
