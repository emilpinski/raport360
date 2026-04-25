# Raport360

> 🚧 **Work in Progress** — projekt w aktywnej fazie budowy
>
> Wywiad gospodarczy AI — due diligence polskich firm w 30 sekund.

![Screenshot](./screenshot.png)

## Co to jest

Raport360 to platforma do weryfikacji i analizy polskich firm. Baza obejmuje prawie 700 000 spółek KRS i 683 000+ przedsiębiorców CEIDG. System generuje automatyczny rating A-E, Risk Score 0-100 oparty na 12 czynnikach oraz pełne due diligence z analizą finansową ze sprawozdań MF.

Skierowana do prawników, doradców, firm leasingowych i każdego, kto potrzebuje szybkiej weryfikacji kontrahenta przed podpisaniem umowy lub udzieleniem finansowania.

## Funkcje

- **Risk Score AI** — ocena ryzyka 0-100 na podstawie: wieku firmy, kapitału, zmian zarządu, Altman Z-Score
- **Rating A-E** — automatyczna klasyfikacja kondycji finansowej (A=bezpieczna, E=ryzykowna)
- **Analiza finansowa** — przychody, zysk netto, bilans, wskaźniki z e-Sprawozdań Ministerstwa Finansów
- **Zarząd i powiązania** — prokurenci, rada nadzorcza, historia zmian, relacje kapitałowe
- **Weryfikacja kontrahenta** — status aktywności, NIP, REGON, KRS, dane identyfikacyjne
- **Live Feed** — real-time feed nowych wpisów i zmian w KRS
- **Ranking firm** — Top 30 i dynamiczne zestawienia branżowe
- **Kalkulator wskaźników** — ROE, ROA, current ratio, quick ratio
- **Bezpłatnie bez rejestracji** — podstawowe dane dostępne publicznie

## Stack

| Warstwa | Technologia |
|---------|-------------|
| Frontend | Next.js 15, React 19, TypeScript, Tailwind CSS v3 |
| Backend | Next.js API Routes |
| Baza danych | Supabase (PostgreSQL) |
| Wykresy | Recharts |
| Email | Resend |
| Ikony | Lucide React |
| Deploy | Vercel |

## Uruchomienie

```bash
git clone https://github.com/emilpinski/raport360
cd raport360
npm install
cp .env.example .env.local
# Uzupelnij zmienne srodowiskowe
npm run dev
```

## Zmienne środowiskowe

| Zmienna | Opis | Wymagana |
|---------|------|----------|
| `NEXT_PUBLIC_SUPABASE_URL` | URL projektu Supabase | ✅ |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Klucz publiczny Supabase | ✅ |
| `SUPABASE_SERVICE_ROLE_KEY` | Klucz serwisowy (backend) | ✅ |
| `RESEND_API_KEY` | Klucz API Resend (emaile) | ✅ |
| `NEXT_PUBLIC_APP_URL` | Publiczny URL aplikacji | ✅ |

## Status

Demo — [raport360.vercel.app](https://raport360.vercel.app)

---
Built by [Emil Piński](https://emilpinski.pl)
