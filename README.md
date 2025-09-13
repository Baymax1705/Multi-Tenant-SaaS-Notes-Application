# Multi-Tenant SaaS Notes Application

A full-stack multi-tenant SaaS notes application built with Next.js, TypeScript, Prisma, and deployed on Vercel. This application demonstrates strict tenant isolation, role-based access control, and subscription gating.

## 🏗️ Architecture

### Multi-Tenancy Approach

This application uses a **shared schema with tenant_id column** approach for multi-tenancy:

- **Single Database**: All tenants share the same database and schema
- **Tenant Isolation**: Data is isolated using `tenant_id` foreign keys
- **Strict Separation**: Every query includes tenant filtering to prevent data leakage
- **Scalable**: Easy to add new tenants without schema changes

### Database Schema

```sql
-- Tenants table
CREATE TABLE tenants (
  id TEXT PRIMARY KEY,
  slug TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  plan TEXT DEFAULT 'FREE',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Users table with tenant relationship
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  password TEXT NOT NULL,
  role TEXT DEFAULT 'MEMBER',
  tenant_id TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE
);

-- Notes table with tenant isolation
CREATE TABLE notes (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  tenant_id TEXT NOT NULL,
  user_id TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

## 🔐 Authentication & Authorization

### JWT-Based Authentication
- Secure token-based authentication
- 7-day token expiration
- Automatic token validation on protected routes

### Role-Based Access Control
- **Admin**: Can invite users and upgrade subscription plans
- **Member**: Can only manage notes (CRUD operations)

### Test Accounts
All test accounts use password: `password`

| Email | Role | Tenant |
|-------|------|--------|
| admin@acme.test | Admin | Acme |
| user@acme.test | Member | Acme |
| admin@globex.test | Admin | Globex |
| user@globex.test | Member | Globex |

## 💳 Subscription Plans

### Free Plan
- Maximum 3 notes per tenant
- Basic note management features

### Pro Plan
- Unlimited notes
- All features unlocked

### Upgrade Process
- Admin users can upgrade tenants via API
- Immediate plan activation
- No payment processing (demo purposes)

## 🚀 API Endpoints

### Authentication
- `POST /api/auth/login` - User login

### Health Check
- `GET /api/health` - Application health status

### Notes Management
- `GET /api/notes` - List all notes for current tenant
- `POST /api/notes` - Create a new note
- `GET /api/notes/[id]` - Get specific note
- `PUT /api/notes/[id]` - Update note
- `DELETE /api/notes/[id]` - Delete note

### Tenant Management
- `POST /api/tenants/[slug]/upgrade` - Upgrade tenant to Pro (Admin only)

## 🛡️ Security Features

### Tenant Isolation
- All database queries include tenant filtering
- JWT tokens contain tenant information
- API routes validate tenant ownership
- No cross-tenant data access possible

### Authorization Middleware
- `requireAuth()` - Ensures user is authenticated
- `requireRole(role)` - Ensures user has specific role
- Automatic token validation and user verification

### Input Validation
- Required field validation
- Data sanitization
- Error handling with appropriate HTTP status codes

## 🎨 Frontend Features

### Modern UI
- Responsive design with Tailwind CSS
- Clean, professional interface
- Mobile-friendly layout

### User Experience
- Real-time form validation
- Loading states and error handling
- Intuitive navigation
- Test account quick-fill buttons

### Key Components
- **LoginForm**: Authentication with test account shortcuts
- **NotesList**: Main dashboard with note management
- **NoteForm**: Create and edit notes
- **NoteCard**: Individual note display
- **Header**: User info and logout

## 🚀 Local Development Setup

### Prerequisites
- Node.js 18+ 
- npm or yarn

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd multi-tenant-notes-app
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Set up environment variables**
   ```bash
   cp env.example .env.local
   ```
   
   Edit `.env.local` and set your JWT secret:
   ```env
   JWT_SECRET=your-super-secret-jwt-key-change-in-production
   DATABASE_URL="file:./dev.db"
   ```

4. **Set up the database**
   ```bash
   # Generate Prisma client
   npm run db:generate
   
   # Push schema to database
   npm run db:push
   
   # Seed with test data
   npm run db:seed
   ```

5. **Start the development server**
   ```bash
   npm run dev
   ```

6. **Open your browser**
   Navigate to [http://localhost:3000](http://localhost:3000)

### Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint
- `npm run db:generate` - Generate Prisma client
- `npm run db:push` - Push schema changes to database
- `npm run db:seed` - Seed database with test data

## 🌐 Deployment on Vercel

### Prerequisites
- Vercel account
- GitHub repository

### Deployment Steps

1. **Push to GitHub**
   ```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

2. **Deploy on Vercel**
   - Connect your GitHub repository to Vercel
   - Set environment variables in Vercel dashboard:
     - `JWT_SECRET`: Your secure JWT secret
     - `DATABASE_URL`: Your production database URL
   - Deploy automatically on push

3. **Database Setup**
   - For production, use a proper database (PostgreSQL recommended)
   - Update `DATABASE_URL` in Vercel environment variables
   - Run migrations and seed data

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `JWT_SECRET` | Secret key for JWT token signing | Yes |
| `DATABASE_URL` | Database connection string | Yes |
| `NEXT_PUBLIC_API_URL` | Public API URL (for production) | No |

## 🧪 Testing the Application

### Health Check
```bash
curl https://your-app.vercel.app/api/health
# Expected: {"status":"ok"}
```

### Login Test
```bash
curl -X POST https://your-app.vercel.app/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@acme.test","password":"password"}'
```

### Notes API Test
```bash
# Get notes (requires authentication token)
curl -X GET https://your-app.vercel.app/api/notes \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## 🔍 Evaluation Checklist

- ✅ `/health` endpoint works
- ✅ Login works for all test accounts
- ✅ Tenant isolation enforced
- ✅ Role restrictions enforced
- ✅ Free plan limit enforced (3 notes)
- ✅ Pro plan removes limit
- ✅ Notes API CRUD works
- ✅ Frontend deployed and working
- ✅ Multi-tenancy properly implemented
- ✅ JWT authentication working
- ✅ Subscription gating functional

## 🛠️ Tech Stack

- **Frontend**: Next.js 14, React 18, TypeScript
- **Backend**: Next.js API Routes
- **Database**: Prisma ORM with SQLite (dev) / PostgreSQL (prod)
- **Authentication**: JWT with bcryptjs
- **Styling**: Tailwind CSS
- **Deployment**: Vercel
- **Language**: TypeScript

## 📁 Project Structure

```
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── auth/          # Authentication endpoints
│   │   ├── notes/         # Notes CRUD endpoints
│   │   ├── tenants/       # Tenant management
│   │   └── health/        # Health check
│   ├── globals.css        # Global styles
│   ├── layout.tsx         # Root layout
│   ├── page.tsx           # Home page
│   └── providers.tsx      # Context providers
├── components/            # React components
│   ├── Header.tsx         # App header
│   ├── LoginForm.tsx      # Login form
│   ├── NoteCard.tsx       # Individual note card
│   ├── NoteForm.tsx       # Note creation/editing
│   └── NotesList.tsx      # Notes dashboard
├── contexts/              # React contexts
│   └── AuthContext.tsx    # Authentication context
├── lib/                   # Utility libraries
│   ├── auth.ts            # Authentication utilities
│   ├── api.ts             # API client
│   └── db.ts              # Database connection
├── prisma/                # Database schema and migrations
│   ├── schema.prisma      # Prisma schema
│   └── seed.ts            # Database seeding
├── vercel.json            # Vercel deployment config
└── README.md              # This file
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License.

## 🆘 Support

For support or questions, please open an issue in the GitHub repository.

---

**Built with ❤️ using Next.js, TypeScript, and Prisma**
