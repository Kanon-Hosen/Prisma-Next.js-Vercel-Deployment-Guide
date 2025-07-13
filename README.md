# 📦 Prisma + Next.js + Vercel Deployment Guide
## 📚 Table of Contents

- [Common Errors](#-common-errors-on-vercel)
- [Fix Guide](#-full-fix-guide)
- [Environment Variables Setup](#5-environment-variables-in-vercel)
- [Database Suggestions](#6-use-a-persistent-db-not-sqlite)
- [Prisma Client Setup](#7-setup-libprismats)
- [Folder Structure Example](#8-folder-structure-example)
- [Final Checklist](#9-final-checklist)
- [Deployment Commands](#-deploy)
- [Local Testing](#-local-test-before-push)
- [Bonus Tips](#-bonus-tips)
- [Contributing](#-contributing)

This README covers all common Prisma + Next.js issues when deploying on **Vercel** and how to fix them step-by-step.

---

## 🚫 Common Errors on Vercel

- `PrismaClientInitializationError`
- `Error: Cannot find module '.prisma/client/index.js'`
- `Query engine library for current platform not found`
- `Prisma Client is unable to run in Edge Runtime`

---

## ✅ Full Fix Guide

### 1. Install Prisma & Client as Production Dependencies

```bash
npm install prisma @prisma/client
```

> ⚠️ Don't put them in `devDependencies`!

---

### 2. Add Postinstall Script in `package.json`

```json
"scripts": {
  "postinstall": "prisma generate"
}
```

---

### 3. Update `schema.prisma`

✅ Use Binary Targets for Vercel Engine Compatibility:

```prisma
generator client {
  provider      = "prisma-client-js"
  binaryTargets = ["native", "rhel-openssl-1.1.x", "debian-openssl-1.1.x", "linux-musl", "linux-arm64-openssl-1.1.x"]
}
```

> You can also try: `["native", "debian-openssl-1.1.x"]`

---

### 4. Avoid Edge Runtime for Prisma

In your API route or server file:

```ts
export const config = {
  runtime: 'nodejs', // ✅ Not edge
};
```

---

### 5. Environment Variables in Vercel

Go to **Vercel > Project > Settings > Environment Variables**:

| Name              | Example Value                                |
|-------------------|-----------------------------------------------|
| `DATABASE_URL`    | `postgresql://user:pass@host:5432/dbname`     |
| `NEXT_PUBLIC_API` | `https://yourdomain.vercel.app/api`          |

---

### 6. Use a Persistent DB (not SQLite)

SQLite doesn't work well on serverless. Use:

✅ [Neon](https://neon.tech)  
✅ [PlanetScale](https://planetscale.com)  
✅ [Railway](https://railway.app)

---

### 7. Setup `lib/prisma.ts`

```ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: ['query'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

---

### 8. Folder Structure Example

```
/prisma/schema.prisma
/lib/prisma.ts
/app/api/...
```

---

### 9. Final Checklist

- [x] Installed Prisma & Client in dependencies  
- [x] Added `postinstall` script  
- [x] Updated `schema.prisma` with `binaryTargets`  
- [x] Avoided edge runtime in API routes  
- [x] Set proper environment variables in Vercel  
- [x] Using external DB (Postgres, etc)

---

## ✅ Deploy

```bash
git add .
git commit -m "fix: prisma vercel deployment setup"
git push
```

---

## 🧪 Local Test Before Push

```bash
npx prisma generate
npx prisma validate
```

---

## 📌 Bonus Tips

- Use `.env.local` for local and Vercel env vars separately.
- Use Vercel logs to debug (`Deployments > View Functions Logs`)
- Avoid deploying with `vercel --prod` until local works.

---

**Happy coding!** 🎉

---

## 🤝 Contributing

Found a new Prisma + Vercel issue or have a better fix?

📚 **Helpful Links:**
- Prisma Docs: https://www.prisma.io/docs
- Next.js Docs: https://nextjs.org/docs
- Vercel Docs: https://vercel.com/docs
- Prisma + Vercel Example: https://github.com/prisma/prisma-examples/tree/latest/deployment-platforms/vercel

We welcome contributions! Here's how you can help:

### 🐛 Found a Bug or Error?

1. Open an [Issue](https://github.com/your-username/your-repo/issues)  
2. Describe the error, steps to reproduce, and your fix (if any)

### 💡 Want to Improve the Guide?

1. Fork the repo  
2. Create a new branch  
   ```bash
   git checkout -b fix/my-new-tip
   ```  
3. Make your changes to `README.md`  
4. Commit & Push  
   ```bash
   git commit -m "docs: added fix for [your-topic]"
   git push origin fix/my-new-tip
   ```  
5. Create a Pull Request

---

### 📝 Contribution Rules

- Keep explanations clear and beginner-friendly  
- Use proper markdown formatting  
- Add comments or references (if needed)  
- Only submit tested fixes!

---

🔐 All contributions will be reviewed and merged as appropriate.

Thank you for helping improve this guide! 🙏