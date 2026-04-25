# Raport360

![Status](https://img.shields.io/badge/Status-Demo-blue)

> AI-powered business intelligence — due diligence on Polish companies in 30 seconds.

![Screenshot](screenshot.png)

## What is it

Raport360 is a platform for verifying and analyzing Polish companies. The database covers nearly 700,000 KRS companies and 683,000+ CEIDG sole traders. The system generates an automatic A-E rating, a Risk Score 0-100 based on 12 factors, and full due diligence with financial analysis from Ministry of Finance financial statements.

Built for lawyers, advisors, leasing companies, and anyone who needs quick contractor verification before signing a contract or providing financing.

## Features

- **AI Risk Score** — risk assessment 0-100 based on: company age, capital, management changes, Altman Z-Score
- **A-E Rating** — automatic financial health classification (A=safe, E=risky)
- **Financial analysis** — revenue, net profit, balance sheet, ratios from Ministry of Finance e-Reports
- **Management and connections** — proxies, supervisory board, change history, capital relationships
- **Contractor verification** — activity status, NIP, REGON, KRS, identification data
- **Live Feed** — real-time feed of new entries and changes in KRS
- **Company ranking** — Top 30 and dynamic industry rankings
- **Ratio calculator** — ROE, ROA, current ratio, quick ratio
- **Free without registration** — basic data available publicly

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 15, React 19, TypeScript, Tailwind CSS v3 |
| Backend | Next.js API Routes |
| Database | Supabase (PostgreSQL) |
| Charts | Recharts |
| Email | Resend |
| Icons | Lucide React |
| Deploy | Vercel |

## Getting Started

```bash
git clone https://github.com/emilpinski/raport360
cd raport360
npm install
cp .env.example .env.local
# Fill in environment variables
npm run dev
```

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL | ✅ |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase public key | ✅ |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service key (backend) | ✅ |
| `RESEND_API_KEY` | Resend API key (emails) | ✅ |
| `NEXT_PUBLIC_APP_URL` | Public application URL | ✅ |

## Status

Demo — [raport360.pl](https://raport360.pl)

---
Built by [Emil Piński](https://emilpinski.pl)

## Screenshots

![Screenshot](screenshot.png)
![Screenshot](screenshot.png)
