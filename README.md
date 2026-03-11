# LetsDoIt 🚀

**AI-Powered Startup Collaboration Platform**

> Where startup ideas become companies. A global digital space for founders, developers, designers, marketers, and investors to share ideas, validate them, and build companies together.

[![Next.js](https://img.shields.io/badge/Next.js-14-black?style=flat-square&logo=next.js)](https://nextjs.org)
[![React](https://img.shields.io/badge/React-18-blue?style=flat-square&logo=react)](https://react.dev)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-3-06B6D4?style=flat-square&logo=tailwindcss)](https://tailwindcss.com)
[![Anthropic](https://img.shields.io/badge/AI-Claude-orange?style=flat-square)](https://anthropic.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## ✨ Features

### Core Platform
- 🔐 **Authentication** — Signup/login with role-based profiles (Founder, Developer, Designer, Marketer, Investor)
- 📊 **Founder Dashboard** — Personalized feed with trending ideas, AI trend intelligence, and metrics
- 💡 **Idea Submission** — 3-step wizard with AI pre-validation before publishing
- 🗳️ **Community Validation** — Vote ideas as High Potential / Needs Improvement / Not Viable
- 💬 **Threaded Discussions** — Nested comments with founder updates and feedback threads
- 🤝 **Collaboration System** — Request to join as Co-founder, Developer, Designer, or Marketer
- ⭐ **Reputation System** — Earn points for posting, validating, and collaborating
- 🔍 **Advanced Search** — Filter by industry, tags, status, AI score, popularity

### AI Engine (9 Intelligence Features)
| Feature | Description |
|---|---|
| 🧠 **Idea Validator** | Auto-scores every idea 0–100 on market demand, scalability, risk |
| 📈 **Market Research** | Generates competitor analysis, trends, market sizing |
| 🗺️ **Roadmap Generator** | Custom 90-day execution plan with tech stack recommendations |
| 🤝 **Co-founder Matching** | Matches collaborators by skills, interests, and activity |
| 💡 **Improvement Engine** | Suggests better markets, models, and product directions |
| 🔍 **Duplicate Detection** | Flags similar ideas already on the platform |
| 📊 **Success Probability** | Estimates startup success % across multiple parameters |
| 🔮 **Trend Prediction** | Identifies sectors likely to grow rapidly |
| 💰 **Investor Readiness** | Scores funding-readiness with actionable improvements |

---

## 🖥️ Screenshots

> The platform features a dark-mode-first design with animated score rings, card-based idea feeds, and a full AI analysis panel per idea.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14, React 18, Tailwind CSS |
| Backend | Node.js, Express.js |
| Database | PostgreSQL + Redis |
| AI | Anthropic Claude API |
| Auth | NextAuth.js + JWT |
| Hosting | Vercel (frontend) + AWS ECS (backend) |
| Search | Elasticsearch (at scale) |

---

## 🚀 Quick Start

### Prerequisites
- Node.js 20+
- PostgreSQL 15+
- Redis 7+
- Anthropic API key → [console.anthropic.com](https://console.anthropic.com)

### 1. Clone the repo

```bash
git clone https://github.com/yourusername/letsdoit.git
cd letsdoit
```

### 2. Install dependencies

```bash
# Frontend
cd frontend && npm install

# Backend
cd ../backend && npm install
```

### 3. Set up environment variables

```bash
# Frontend
cp frontend/.env.example frontend/.env.local

# Backend
cp backend/.env.example backend/.env
```

Fill in your values — see [Environment Variables](#environment-variables) below.

### 4. Set up the database

```bash
cd backend
psql -U postgres -c "CREATE DATABASE letsdoit;"
psql -U postgres -d letsdoit -f src/db/schema.sql
```

### 5. Run locally

```bash
# From root — runs both frontend and backend
npm run dev
```

Frontend: http://localhost:3000  
Backend API: http://localhost:4000

---

## ⚙️ Environment Variables

### Frontend (`frontend/.env.local`)

```env
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXTAUTH_SECRET=your-nextauth-secret-here
NEXTAUTH_URL=http://localhost:3000
```

### Backend (`backend/.env`)

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/letsdoit
JWT_SECRET=your-jwt-secret-here
JWT_EXPIRES_IN=7d
ANTHROPIC_API_KEY=sk-ant-your-key-here
REDIS_URL=redis://localhost:6379
PORT=4000
ALLOWED_ORIGINS=http://localhost:3000
SMTP_HOST=smtp.resend.com
SMTP_PORT=465
SMTP_KEY=your-email-key
```

---

## 📁 Project Structure

```
letsdoit/
├── frontend/                    # Next.js 14 App Router
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx             # Landing / Auth
│   │   ├── dashboard/page.tsx
│   │   ├── explore/page.tsx
│   │   ├── ideas/[id]/page.tsx
│   │   ├── submit/page.tsx
│   │   ├── profile/[id]/page.tsx
│   │   └── admin/page.tsx
│   ├── components/
│   │   ├── IdeaCard.tsx
│   │   ├── IdeaDetail.tsx
│   │   ├── AIReport.tsx
│   │   ├── Dashboard.tsx
│   │   ├── SubmitForm.tsx
│   │   └── CollabPanel.tsx
│   └── lib/
│       ├── api.ts
│       ├── auth.ts
│       └── ai.ts
│
├── backend/                     # Node.js + Express
│   └── src/
│       ├── routes/              # auth, ideas, users, ai, admin
│       ├── middleware/          # auth, rateLimit, validate
│       ├── services/            # aiService, emailService, reputationService
│       └── db/schema.sql
│
├── LetsDoIt-Platform.jsx        # Complete React prototype
├── LetsDoIt-Architecture.md     # Full architecture docs
├── docker-compose.yml
└── README.md
```

---

## 🐳 Docker

Run the entire stack with one command:

```bash
docker-compose up --build
```

---

## 🌐 Deployment

### Frontend → Vercel

```bash
cd frontend
npx vercel --prod
```

### Backend → Railway (easiest)

```bash
railway login
railway init
railway up
```

### Backend → AWS ECS (production)

See `infra/` directory for Terraform configs and full AWS deployment guide.

---

## 📊 API Overview

```
POST  /api/auth/register       Create account
POST  /api/auth/login          Login → JWT
GET   /api/ideas               List ideas (filter, paginate)
POST  /api/ideas               Submit idea (triggers AI analysis)
GET   /api/ideas/:id           Idea detail + AI report
POST  /api/ideas/:id/vote      Vote on idea
POST  /api/ideas/:id/comments  Post comment
POST  /api/ideas/:id/collabs   Request collaboration
POST  /api/ai/analyze          Trigger AI deep analysis
POST  /api/ai/roadmap          Generate custom roadmap
GET   /api/ai/trends           Industry trend predictions
GET   /api/admin/analytics     Platform analytics
```

Full API reference: [`LetsDoIt-Architecture.md`](./LetsDoIt-Architecture.md)

---

## ⭐ Reputation System

| Action | Points |
|---|---|
| Submit an idea | +50 |
| Idea voted "High Potential" | +20 per vote |
| Post a comment | +10 |
| Comment liked | +5 per like |
| Validate ideas | +5 per vote |
| Collaboration approved | +30 |
| Complete a collaboration | +100 |
| Idea goes trending | +50 |

**Tiers:** Newcomer → Builder → Expert → Elite

---

## 🔒 Security

- JWT authentication with refresh tokens
- Rate limiting (100 req/min per user)
- Input validation via Zod
- Parameterized queries (no SQL injection)
- CSP headers (XSS prevention)
- bcrypt password hashing (12 rounds)
- AI prompt injection prevention

---

## 🗺️ Roadmap

- [ ] Real-time notifications (WebSockets)
- [ ] Idea pitch deck generator (AI)
- [ ] Investor matchmaking
- [ ] Video pitch uploads
- [ ] Mobile app (React Native)
- [ ] Startup progress tracker
- [ ] Public API for developers

---

## 🤝 Contributing

Contributions welcome! Please open an issue first to discuss what you'd like to change.

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📄 License

MIT © 2024 LetsDoIt

---

<p align="center">Built with ❤️ and AI · <a href="https://letsdoit.app">letsdoit.app</a></p>
