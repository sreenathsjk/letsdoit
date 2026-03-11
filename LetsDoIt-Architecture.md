# LetsDoIt — AI-Powered Startup Collaboration Platform
## Complete Architecture & Deployment Guide

---

## Project Structure

```
letsdoit/
├── frontend/                    # Next.js 14 App Router
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx             # Landing/Auth
│   │   ├── dashboard/page.tsx
│   │   ├── explore/page.tsx
│   │   ├── ideas/[id]/page.tsx
│   │   ├── submit/page.tsx
│   │   ├── profile/[id]/page.tsx
│   │   └── admin/page.tsx
│   ├── components/
│   │   ├── ui/                  # Shared UI components
│   │   ├── IdeaCard.tsx
│   │   ├── IdeaDetail.tsx
│   │   ├── AIReport.tsx
│   │   ├── Dashboard.tsx
│   │   ├── SubmitForm.tsx
│   │   └── CollabPanel.tsx
│   ├── lib/
│   │   ├── api.ts               # API client
│   │   ├── auth.ts              # NextAuth config
│   │   └── ai.ts                # AI utility functions
│   └── types/index.ts
│
├── backend/                     # Node.js + Express API
│   ├── src/
│   │   ├── routes/
│   │   │   ├── auth.ts
│   │   │   ├── ideas.ts
│   │   │   ├── users.ts
│   │   │   ├── comments.ts
│   │   │   ├── collabs.ts
│   │   │   ├── ai.ts
│   │   │   └── admin.ts
│   │   ├── middleware/
│   │   │   ├── auth.ts          # JWT verification
│   │   │   ├── rateLimit.ts
│   │   │   └── validate.ts
│   │   ├── services/
│   │   │   ├── aiService.ts     # AI orchestration
│   │   │   ├── emailService.ts
│   │   │   └── reputationService.ts
│   │   └── db/
│   │       ├── schema.sql
│   │       └── migrations/
│   └── package.json
│
├── ai-engine/                   # AI microservice (optional)
│   ├── validators/
│   ├── market-research/
│   └── matching/
│
└── infra/                       # Infrastructure as Code
    ├── docker-compose.yml
    ├── Dockerfile.frontend
    ├── Dockerfile.backend
    └── nginx.conf
```

---

## Database Schema (PostgreSQL)

```sql
-- Users
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(50) CHECK (role IN ('Founder','Developer','Designer','Marketer','Investor')),
  skills TEXT[],
  country VARCHAR(100),
  bio TEXT,
  avatar_url VARCHAR(500),
  reputation_score INTEGER DEFAULT 0,
  ideas_count INTEGER DEFAULT 0,
  collabs_count INTEGER DEFAULT 0,
  is_admin BOOLEAN DEFAULT FALSE,
  is_verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Startup Ideas
CREATE TABLE ideas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  author_id UUID REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(200) NOT NULL,
  problem TEXT NOT NULL,
  solution TEXT NOT NULL,
  target_users TEXT,
  market_size VARCHAR(100),
  monetization TEXT,
  industry VARCHAR(100),
  tags TEXT[],
  seeking_roles TEXT[],
  status VARCHAR(50) DEFAULT 'New', -- New, Hot, Trending, Archived
  views INTEGER DEFAULT 0,
  collaborators_count INTEGER DEFAULT 0,
  ai_score INTEGER,
  validation_score INTEGER,
  is_moderated BOOLEAN DEFAULT FALSE,
  is_duplicate BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- AI Analysis Results
CREATE TABLE ai_reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  idea_id UUID REFERENCES ideas(id) ON DELETE CASCADE UNIQUE,
  market_demand INTEGER,
  competition_level INTEGER,
  monetization_potential INTEGER,
  scalability INTEGER,
  risk_level INTEGER,
  success_probability INTEGER,
  investor_readiness INTEGER,
  competitors TEXT[],
  market_trends TEXT[],
  roadmap_steps TEXT[],
  tech_stack TEXT[],
  full_analysis TEXT,
  improvement_suggestions TEXT,
  duplicate_ids UUID[],
  generated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Votes
CREATE TABLE votes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  idea_id UUID REFERENCES ideas(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  vote_type VARCHAR(20) CHECK (vote_type IN ('high','improve','notViable')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(idea_id, user_id)
);

-- Comments
CREATE TABLE comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  idea_id UUID REFERENCES ideas(id) ON DELETE CASCADE,
  author_id UUID REFERENCES users(id) ON DELETE CASCADE,
  parent_id UUID REFERENCES comments(id),
  content TEXT NOT NULL,
  likes INTEGER DEFAULT 0,
  is_founder_update BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Collaboration Requests
CREATE TABLE collab_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  idea_id UUID REFERENCES ideas(id) ON DELETE CASCADE,
  requester_id UUID REFERENCES users(id) ON DELETE CASCADE,
  requested_role VARCHAR(50),
  message TEXT,
  status VARCHAR(20) DEFAULT 'pending', -- pending, approved, rejected
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Reputation Events
CREATE TABLE reputation_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  event_type VARCHAR(100),
  points INTEGER,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes for Performance
CREATE INDEX idx_ideas_author ON ideas(author_id);
CREATE INDEX idx_ideas_status ON ideas(status);
CREATE INDEX idx_ideas_industry ON ideas(industry);
CREATE INDEX idx_ideas_ai_score ON ideas(ai_score DESC);
CREATE INDEX idx_ideas_tags ON ideas USING GIN(tags);
CREATE INDEX idx_votes_idea ON votes(idea_id);
CREATE INDEX idx_comments_idea ON comments(idea_id);
CREATE INDEX idx_reputation_user ON reputation_events(user_id);
```

---

## Backend API Routes

```typescript
// Auth Routes
POST   /api/auth/register     - Create account
POST   /api/auth/login        - Login, returns JWT
POST   /api/auth/refresh      - Refresh token
POST   /api/auth/logout       - Invalidate token
GET    /api/auth/me           - Get current user

// Ideas Routes
GET    /api/ideas             - List ideas (filterable, paginated)
POST   /api/ideas             - Submit new idea (triggers AI analysis)
GET    /api/ideas/:id         - Get idea with AI report
PUT    /api/ideas/:id         - Update idea (author only)
DELETE /api/ideas/:id         - Delete idea (author/admin)
GET    /api/ideas/trending    - Trending ideas
GET    /api/ideas/recommended - AI-recommended for user

// Voting
POST   /api/ideas/:id/vote    - Cast vote (high/improve/notViable)
DELETE /api/ideas/:id/vote    - Remove vote

// Comments
GET    /api/ideas/:id/comments     - Get threaded comments
POST   /api/ideas/:id/comments     - Post comment
POST   /api/comments/:id/replies   - Reply to comment
PUT    /api/comments/:id           - Edit comment
DELETE /api/comments/:id           - Delete comment
POST   /api/comments/:id/like      - Like comment

// Collaborations
GET    /api/ideas/:id/collabs         - Get collab requests
POST   /api/ideas/:id/collabs         - Request to collaborate
PUT    /api/ideas/:id/collabs/:reqId  - Approve/reject request

// Users
GET    /api/users/:id          - Get user profile
PUT    /api/users/:id          - Update profile
GET    /api/users/:id/ideas    - User's ideas
GET    /api/users/:id/collabs  - User's collaborations
GET    /api/users/leaderboard  - Top users by reputation

// AI Endpoints
POST   /api/ai/analyze         - Trigger AI analysis for idea
POST   /api/ai/roadmap         - Generate custom roadmap
POST   /api/ai/suggestions     - Get improvement suggestions
POST   /api/ai/match           - Get co-founder matches
GET    /api/ai/trends          - Industry trend predictions
POST   /api/ai/duplicate-check - Check for duplicate ideas

// Admin
GET    /api/admin/users        - Manage users
PUT    /api/admin/users/:id    - Update user (ban, verify)
GET    /api/admin/ideas        - All ideas for moderation
PUT    /api/admin/ideas/:id    - Moderate idea
GET    /api/admin/analytics    - Platform analytics
DELETE /api/admin/spam/:id     - Remove spam content
```

---

## AI Integration Architecture

```typescript
// aiService.ts - Core AI orchestration

import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// 1. Full Idea Analysis (called on every submission)
export async function analyzeIdea(idea: IdeaInput): Promise<AIReport> {
  const prompt = buildAnalysisPrompt(idea);
  const response = await client.messages.create({
    model: 'claude-opus-4-6',
    max_tokens: 2000,
    system: VENTURE_ANALYST_SYSTEM_PROMPT,
    messages: [{ role: 'user', content: prompt }]
  });
  return parseAnalysisResponse(response.content[0].text);
}

// 2. Duplicate Detection
export async function checkDuplicate(idea: IdeaInput, existingIdeas: Idea[]): Promise<DuplicateResult> {
  // Use embedding similarity + LLM verification
  const embeddings = await generateEmbeddings(idea.title + idea.problem);
  const similar = await vectorSearch(embeddings, threshold: 0.85);
  if (similar.length > 0) {
    return { isDuplicate: true, similarIds: similar.map(s => s.id), confidence: similar[0].score };
  }
  return { isDuplicate: false };
}

// 3. Co-founder Matching
export async function matchCoFounders(user: User, idea: Idea, candidates: User[]): Promise<Match[]> {
  // Score each candidate using AI
  const scores = await Promise.all(candidates.map(async (candidate) => {
    const score = await scoreMatch(user, idea, candidate);
    return { user: candidate, score };
  }));
  return scores.sort((a, b) => b.score - a.score).slice(0, 10);
}

// 4. Trend Prediction Engine
export async function predictTrends(): Promise<TrendReport> {
  // Analyze idea volume by industry over time
  // Combine with market data via web search
  // Return ranked sectors
}

// 5. Investor Readiness Score
export async function investorReadiness(idea: Idea, aiReport: AIReport): Promise<ReadinessReport> {
  const factors = {
    teamStrength: scoreTeam(idea),
    marketSize: parseMarketSize(idea.marketSize),
    traction: idea.validationScore,
    businessModel: aiReport.monetizationPotential,
    competitive: 100 - aiReport.competition,
  };
  return calculateReadiness(factors);
}
```

---

## Reputation System

| Action                     | Points |
|---------------------------|--------|
| Submit idea               | +50    |
| Idea voted "High Potential"| +20 each |
| Post helpful comment       | +10    |
| Comment liked by others    | +5 each |
| Validate ideas (vote)      | +5     |
| Collab request approved    | +30    |
| Complete collaboration     | +100   |
| Daily login                | +2     |
| Invite user who joins      | +25    |
| Idea goes trending         | +50    |

**Reputation Tiers:**
- 0–499: Newcomer
- 500–1499: Builder  
- 1500–2999: Expert
- 3000+: Elite

---

## Deployment Guide

### Prerequisites
- Node.js 20+
- PostgreSQL 15+
- Redis 7+
- Anthropic API key

### Environment Variables

```bash
# Backend (.env)
DATABASE_URL=postgresql://user:pass@localhost:5432/letsdoit
JWT_SECRET=your-super-secret-key
JWT_EXPIRES_IN=7d
ANTHROPIC_API_KEY=sk-ant-...
REDIS_URL=redis://localhost:6379
SMTP_HOST=smtp.resend.com
SMTP_KEY=your-email-key
ALLOWED_ORIGINS=https://letsdoit.com

# Frontend (.env.local)
NEXT_PUBLIC_API_URL=https://api.letsdoit.com
NEXT_PUBLIC_APP_URL=https://letsdoit.com
NEXTAUTH_SECRET=your-nextauth-secret
NEXTAUTH_URL=https://letsdoit.com
```

### Docker Compose

```yaml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports: ["3000:3000"]
    environment:
      - NEXT_PUBLIC_API_URL=http://backend:4000
    depends_on: [backend]

  backend:
    build: ./backend
    ports: ["4000:4000"]
    environment:
      - DATABASE_URL=postgresql://postgres:pass@db:5432/letsdoit
      - REDIS_URL=redis://redis:6379
    depends_on: [db, redis]

  db:
    image: postgres:15
    volumes: [postgres_data:/var/lib/postgresql/data]
    environment:
      POSTGRES_DB: letsdoit
      POSTGRES_PASSWORD: pass

  redis:
    image: redis:7-alpine
    volumes: [redis_data:/data]

volumes:
  postgres_data:
  redis_data:
```

### Vercel Deployment (Frontend)
```bash
npm i -g vercel
cd frontend && vercel --prod
```

### AWS Deployment (Backend)
```bash
# Using ECS + RDS + ElastiCache
terraform init && terraform apply
# Or use Railway/Render for simpler setup
```

---

## Scalability Architecture

```
Users → CloudFront CDN → Load Balancer
                              ↓
              ┌──────────────────────────┐
              │   Next.js (Vercel Edge)  │
              └──────────────────────────┘
                              ↓
              ┌──────────────────────────┐
              │  Express API (AWS ECS)   │
              │  Multiple instances      │
              └──────────────────────────┘
                    ↓           ↓
            ┌──────────┐  ┌──────────┐
            │PostgreSQL│  │  Redis   │
            │(RDS Multi│  │(Cluster) │
            │   AZ)    │  │          │
            └──────────┘  └──────────┘
                              ↓
            ┌──────────────────────────┐
            │   AI Processing Queue    │
            │   (SQS + Lambda)         │
            └──────────────────────────┘
                              ↓
            ┌──────────────────────────┐
            │   Anthropic Claude API   │
            └──────────────────────────┘
```

**Supports 1M+ users through:**
- Horizontal API scaling via ECS
- Read replicas for PostgreSQL
- Redis caching for hot data
- AI analysis via async job queue (not blocking)
- CDN for static assets
- Edge caching for idea feeds
- Elasticsearch for search at scale

---

## Security Checklist

- [x] JWT authentication with refresh tokens
- [x] Rate limiting per user & IP (100 req/min)
- [x] Input validation & sanitization (Zod)
- [x] SQL injection prevention (Parameterized queries)
- [x] XSS prevention (CSP headers)
- [x] CORS configuration
- [x] Password hashing (bcrypt, 12 rounds)
- [x] Admin endpoints protected
- [x] AI prompt injection prevention
- [x] File upload scanning (if applicable)
