# Our Next.js standards

## Table of Contents

- [Technical Stack](#technical-stack)
- [Next.js 15 App Router - Enterprise Folder Structure Guide](#nextjs-15-app-router---enterprise-folder-structure-guide)
  - [Complete Project Structure](#complete-project-structure)
  - [Key Distinctions & When to Use](#key-distinctions--when-to-use)
    - [`lib/` vs `services/` vs `utils/`](#lib-vs-services-vs-utils)
    - [`app/api/` vs `services/` - Security & Architecture](#appapi-vs-services---security--architecture)
    - [Route Groups & Private Folders](#route-groups--private-folders)
  - [Enterprise Companies Using These Patterns](#enterprise-companies-using-these-patterns)
  - [Best Practices for Scale](#best-practices-for-scale)
    - [Evolution Guidelines](#evolution-guidelines)
    - [Security Rules for `services/` Folder](#security-rules-for-services-folder)
    - [Migration to Feature-Based Organization](#migration-to-feature-based-organization)

---

## Technical Stack

- Next.js - App Router
- Coding : TypeScript
- Styling : [Material UI](https://material-ui.com/) (MUI)
- Modals : [react-toastify](https://www.npmjs.com/package/react-toastify)
- Translation : [next-intl](https://next-intl.dev/). [Our setup here](https://next-intl.dev/docs/getting-started/app-router/without-i18n-routing).
  > _Use [VSCode integration for next-intl](https://next-intl.dev/docs/workflows/vscode-integration) ([i18n Ally extension here](https://next-intl.dev/docs/workflows/vscode-integration#i18n-ally)) to manage messages within a single, large file. Don't split messages purely for organizational reasons._ - ([see more](https://next-intl.dev/docs/usage/configuration#messages))

## Next.js 15 App Router - Enterprise Folder Structure Guide

### **Complete Project Structure**

```
src/
├── app/                    # App Router - Routes & API endpoints
│   ├── layout.tsx          # Root layout, import MUI theme.ts
│   ├── page.tsx            # Home page (/)
│   ├── loading.tsx         # Global loading UI
│   ├── error.tsx           # Global error boundary
│   ├── not-found.tsx       # 404 page
│   ├── (auth)/             # 🔧 Route group (URL: /)
│   │   ├── layout.tsx      # Auth-specific layout
│   │   ├── login/
│   │   │   └── page.tsx    # /login
│   │   └── register/
│   │       └── page.tsx    # /register
│   ├── dashboard/
│   │   ├── _components/    # 🔒 Private folder (route-specific)
│   │   │   ├── Chart.tsx
│   │   │   └── Analytics.tsx
│   │   ├── _hooks/         # 🔒 Private hooks
│   │   │   └── useAnalytics.ts
│   │   ├── layout.tsx      # Dashboard layout
│   │   ├── page.tsx        # /dashboard
│   │   └── settings/
│   │       └── page.tsx    # /dashboard/settings
│   └── api/                # HTTP Route Handlers (public endpoints)
│       ├── auth/
│       │   └── route.ts    # POST /api/auth (OAuth callbacks)
│       ├── webhooks/
│       │   └── stripe/
│       │       └── route.ts # POST /api/webhooks/stripe
│       └── public/
│           └── contact/
│               └── route.ts # POST /api/public/contact
├── components/             # Reusable UI components
│   ├── ui/                 # Atomic components (Button, Card, Modal)
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   └── index.ts    # Export barrel
│   │   └── Card/
│   ├── layout/             # Layout components (Header, Footer, Sidebar)
│   │   ├── Header/
│   │   └── Navigation/
│   └── features/           # Feature-specific components
|       ├── someFeature/    # SomeFeatureCard, SomeFeatureForm, etc.
│       ├── auth/           # AuthForm, LoginForm, etc.
│       └── dashboard/      # DashboardWidget, UserTable, etc.
├── lib/                    # Core configurations & business logic
│   ├── auth.ts             # Auth config (NextAuth, sessions)
│   ├── db.ts               # Database client setup
│   ├── supabase.ts         # Supabase client configuration
|   ├── errors.ts           # Custom error classes
│   ├── validations.ts      # Zod schemas, form validations
│   ├── constants.ts        # App-wide constants
│   └── utils.ts            # Business logic utilities
├── services/               # Data layer & external integrations
│   ├── admin/              # Admin operations
│   │   ├── userAdmin.service.ts
│   │   └── analytics.service.ts
|   ├── someFeature/        # someFeature domain
|   │   ├── someFeature.service.ts
|   │   ├── someFeature.repository.ts
|   │   ├── someFeature.types.ts
|   │   ├── someFeature.hooks.ts
|   │   └── index.ts
│   ├── user/               # User operations  
│   │   ├── profile.service.ts
│   │   └── preferences.service.ts
│   ├── system/             # Auto-personalization
│   │   ├── recommendations.service.ts
│   │   └── ml.service.ts
│   └── external/           # External API calls
│       ├── railsApi.service.ts
│       └── stripeApi.service.ts
├── utils/                  # Pure helper functions (no side effects)
│   ├── formatters.ts       # formatDate, formatCurrency, etc.
│   ├── validators.ts       # Input validation helpers
│   ├── cn.ts              # Class name utility (clsx)
│   └── parsers.ts         # Data parsing utilities
├── types/                  # TypeScript definitions
|   ├── api.types.ts        # Global API types (shared across services)
|   ├── auth.types.ts       # Authentication types (used app-wide)
|   ├── common.types.ts     # Common utility types
│   └── index.ts            # Export barrel
├── hooks/                  # Global Custom React hooks
|   ├── useAuth.ts
|   ├── useLocalStorage.ts
|   └── useDebounce.ts
├── theme.ts              # MUI : Client-side theme (colors, typography)
└── theme.server.ts       # MUI : Server-safe theme elements  

messages/                   # Translation with next-intl
├── en.json                 # English translation (with VSCode exension, better to group all in same file)
└── fr.json                 # French translation

public/
├── favicon.ico                # Main site icon (browser) (32x32px)
├── apple-icon.png             # Icon for iOS/Safari (180x180px)
├── icon-192x192.png           # Icon for PWA standard (small)
├── icon-512x512.png           # Icon for PWA standard (large) 
├── icon-maskable-192x192.png  # Icon for PWA adaptive Android (small)
├── icon-maskable-512x512.png  # Icon for PWA adaptive Android (large)
├── images/                    # Assets visuels de l'app
│   ├── logo_my_app.png        # Some logos...
│   └── logo_something.png
├── icons/
│   └── history.svg
└── fonts/
    ├── RedditSans-Regular.woff2
    ├── RedditSans-Regular.ttf
    └── RedditSans-Regular.otf
```

### **Key Distinctions & When to Use**

#### **`lib/` vs `services/` vs `utils/`**

| Folder | Purpose | Example | Has Side Effects | When to Use |
|--------|---------|---------|------------------|-------------|
| **`lib/`** | **Configuration & Setup** | `supabase.ts` client setup, `auth.ts` NextAuth config | ✅ | Third-party clients, app configuration, business logic |
| **`services/`** | **Data Operations** | `getUserProfile()`, `updateUserSettings()` | ✅ | CRUD operations, API calls, complex business workflows |
| **`utils/`** | **Pure Functions** | `formatDate()`, `cn()`, `validateEmail()` | ❌ | Data transformation, formatting, pure calculations |

**Example to clarify:**
```typescript
// lib/supabase.ts - Configuration
export const supabase = createClient(url, key)

// services/user.service.ts - Data operations using lib
import { supabase } from '@/lib/supabase'
export const getUserProfile = (id: string) => supabase.from('users').select().eq('id', id)

// utils/formatters.ts - Pure helper
export const formatUserName = (first: string, last: string) => `${first} ${last}`
```

#### **`app/api/` vs `services/` - Security & Architecture**

| Location | Purpose | Visibility | Security Level | Use Cases |
|----------|---------|------------|----------------|-----------|
| **`app/api/`** | **Public HTTP endpoints** | 🌐 Exposed to internet | 🔒 High security needed | OAuth callbacks, webhooks, public APIs |
| **`services/`** | **Internal data layer** | 🏠 Internal only | 🔓 Lower risk | Component data fetching, internal business logic |

**Security Rules:**
- **app/api**: Always validate inputs, authenticate requests, rate limit
- **services**: Can assume trusted environment, focus on business logic

#### **Route Groups & Private Folders**

**Route Groups `(name)`:** Organizational only - don't appear in URLs
```
app/(auth)/login/page.tsx → /login (not /auth/login)
app/(marketing)/about/page.tsx → /about (not /marketing/about)
```

**Private Folders `_name`:** Ignored by router, safe for colocation
```
app/dashboard/_components/Chart.tsx → Not accessible via URL
```

## **Enterprise Companies Using These Patterns**

**Tier 1 (Verified Users):**
- **Vercel** - Creator of Next.js
- **Netflix** - Job portal and internal tools
- **Stripe** - Black Friday site (17M+ edge requests)
- **Sonos** - 75% faster builds after migration

**Tier 2 (Industry Standard):**
- **Airbnb** - Host dashboard and booking flows
- **TikTok** - Creator tools and analytics
- **Twitch** - Creator dashboard
- **Hulu** - Content management tools

**Characteristics of their folder structures:**
- ✅ Separation over colocation (98% of code in dedicated folders)
- ✅ Feature-based organization in components/
- ✅ Clear lib/ vs services/ distinction
- ✅ Extensive use of route groups for organization
- ✅ Private folders for route-specific code

## **Best Practices for Scale**

### **Small Apps (< 50 components)**
```
src/
├── app/
├── components/        # Flat structure OK
├── lib/
└── utils/
```

### **Medium Apps (50-200 components)**
```
src/
├── app/
├── components/
│   ├── ui/           # Start categorizing
│   └── features/
├── services/         # Add data layer
├── lib/
└── utils/
```

### **Large Apps (200+ components)**
```
src/
├── app/
├── components/
│   ├── ui/
│   ├── layout/
│   └── features/     # Deep feature organization
├── services/
│   ├── admin/        # Domain separation
│   ├── user/
│   └── external/
├── lib/
├── utils/
└── types/
```

### **Evolution Guidelines**

1. **Start with separation** - Don't fight the ecosystem
2. **Use colocation sparingly** - Only for truly route-specific code
3. **Organize by feature** - Not by file type when you scale
4. **Document your choices** - Team consistency beats perfect structure
5. **Refactor regularly** - Structure should evolve with your app

**The verdict: Use separation as your foundation, with strategic colocation for route-specific components.**

### **Security Rules for `services/` Folder**

#### **⚠️ Critical Security Consideration**

The `services/` folder code **is exposed to the client** when imported in Client Components. Follow these security rules:

```typescript
// ❌ DANGER - Exposed client-side (API keys visible in browser!)
'use client'
import { adminService } from '@/services/admin/users.service' // DON'T DO THIS

// ✅ SECURE - Server Components only
import { getUserProfile } from '@/services/user/profile.service' // Server-side execution
export default async function ProfilePage() {
  const user = await getUserProfile() // Runs on server only
}

// ✅ SECURE - Client calls via app/api endpoints
'use client'
const handleUpdate = () => fetch('/api/admin/users') // Goes through your secure endpoint
```

#### **Security Guidelines**

| Context | Services Usage | Security Level |
|---------|---------------|----------------|
| **Server Components** | ✅ Safe to import directly | 🔒 High - Server execution only |
| **Client Components** | ⚠️ Only non-sensitive data | 🔓 Medium - Code visible in browser |
| **API Routes** | ✅ Safe to import directly | 🔒 High - Server execution only |

**Rule of thumb:** If your service contains API keys, database credentials, or sensitive business logic, **never** import it in Client Components. Always route through `app/api` endpoints.

### **Migration to Feature-Based Organization**

See our detailed migration guide: [Migration to Feature-Based Organization](./migration_to_feature-based_organization.md).
