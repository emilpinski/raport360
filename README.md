# Raport360.pl — Polish Company Intelligence Portal

> Full financial and legal analysis for 700,000+ Polish companies.

**Live:** [raport360.pl](https://raport360.pl) · **Status:** Work in progress

## What it does
- KRS registry data: board members, shareholders, PKD codes, registration history  
- Financial statements: revenue, net profit, assets, equity (from eKRS filings)  
- Risk rating A–E with Altman Z''-Score  
- Interactive voivodeship heat map  
- Company comparison and executive directory

## Tech Stack
`Next.js 15` `TypeScript` `Tailwind CSS` `Supabase` `PostgreSQL` `Playwright` `Vercel`

## Architecture highlight
Playwright-based scraper with 20 parallel Chrome workers bypassing Incapsula WAF to extract financial XML/XHTML filings at ~14,000 companies/hour.
