# NOVAA System Architecture

## MVP1 Architecture Overview

### Technology Stack
- **Frontend**: React 18 + TypeScript
- **Backend**: Node.js 18 + Express
- **Database**: MongoDB 5.0+ (Atlas Free Tier)
- **Payment**: Razorpay (Test Mode)
- **Storage**: AWS S3 (Test Mode)
- **Hosting**: Render.com (Backend), Vercel (Frontend)

### System Boundaries
```
┌─────────────────────────────────────────────────────────────┐
│                    NOVAA MVP SYSTEM (2026)                │
│                                                             │
│  ┌──────────────┐    ┌──────────────────┐ ┌──────────────┐ │
│  │  Frontend    │    │  Backend API     │ │  Database    │ │
│  │  (React)     │◄──►│  (Node.js)       │◄┤  (MongoDB)   │ │
│  │  Vercel      │    │  Render.com      │ │  Atlas Free  │ │
│  └──────────────┘    └──────────────────┘ └──────────────┘ │
│         ▲                     ▲                              │
│         │                     │                              │
│         └─────────────────────┘                              │
│                     │                                        │
│         ┌───────────┴────────────┐                          │
│         ▼                        ▼                           │
│    ┌──────────────┐      ┌──────────────┐                  │
│    │  Razorpay    │      │  AWS S3      │                  │
│    │  (Payments)  │      │  (Documents) │                  │
│    └──────────────┘      └──────────────┘                  │
│                                                             │
│         COLLEGES (5 pilot - Maharashtra only)               │
└─────────────────────────────────────────────────────────────┘
```

### MVP1 Architecture Principles
- **Single Tenant Per Context**: Every request includes collegeId
- **Fail-Safe Defaults**: Queries without collegeId rejected
- **Immutable Audit Trails**: All admin actions logged
- **Graceful Degradation**: System works if S3 is slow
- **No State Assumptions**: Only Maharashtra rules in MVP1