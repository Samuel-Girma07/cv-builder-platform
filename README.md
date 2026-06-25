# CV & Cover Letter Builder Platform

A Server-Side Rendered (SSR) web application for building professional CVs and generating AI-powered cover letters with ATS (Applicant Tracking System) scoring.

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js |
| Framework | Express.js |
| Templating | EJS (Server-Side Rendered) |
| Database | PostgreSQL (`pg`) |
| Sessions | `express-session` + `connect-pg-simple` |
| Authentication | `bcrypt` |
| File Upload | `multer` (memoryStorage, PDF only, 5MB limit) |
| PDF Parsing | `pdf-parse` |
| AI Engine | NVIDIA NIM via `openai` SDK (`meta/llama-3.1-70b-instruct`) |
| Config | `dotenv` |

## Architecture

This project follows a **strict MVC (Model-View-Controller)** pattern:

```
cv-builder-platform/
│
├── app.js                          # Express server entry point
├── package.json
├── .env                            # Environment variables (not committed)
│
├── config/
│   └── db.js                       # PostgreSQL connection pool
│
├── middlewares/
│   └── authMiddleware.js           # Route protection via session verification
│
├── models/                         # All SQL queries live here
│   ├── userQuery.js
│   ├── profileQuery.js
│   └── applicationQuery.js
│
├── routes/
│   ├── authRoutes.js               # Login / Register
│   ├── wizardRoutes.js             # Multi-step onboarding
│   ├── profileRoutes.js            # PDF upload & manual edits
│   └── applicationRoutes.js        # Cover letter generation & ATS scoring
│
├── controllers/
│   ├── authController.js
│   ├── wizardController.js
│   ├── profileController.js
│   └── applicationController.js
│
├── views/
│   ├── partials/                   # header.ejs, footer.ejs, navbar.ejs
│   ├── index.ejs                   # Dashboard
│   ├── auth/                       # login.ejs, register.ejs
│   ├── wizard/                     # step1.ejs, step2.ejs, etc.
│   ├── profile/                    # edit.ejs, upload.ejs
│   └── applications/              # new.ejs, view.ejs, fallback-template.ejs
│
├── public/
│   └── css/                        # Custom styles + @media print rules
│
└── database/
    └── schema.sql                  # PostgreSQL schema definition
```

## Core Features

### 1. Onboarding Wizard
Multi-step profile creation flow using session-based draft data. Collects personal info, experience, and skills — stores structured JSON in the database.

### 2. PDF Magic Parser
Upload a PDF resume → text extraction via `pdf-parse` → AI-powered structured JSON conversion via NVIDIA NIM → saved to database in the same schema as the wizard output.

### 3. ATS AI Matcher
Input a job description → AI compares it against your profile → returns an ATS compatibility score (0-100) and identifies missing skills.

### 4. Cover Letter Generator
Generate AI-powered cover letters with selectable tones (Formal, Confident, Concise). Falls back to a programmatic EJS "Mad-Libs" template if the AI API is unavailable.

### 5. Print-to-PDF Export
Clean `@media print` CSS hides navigation and formats the page as a formal A4 document for browser-native print-to-PDF export.

## Database Schema

- **users** — `id`, `email` (UNIQUE), `password_hash`, `created_at`
- **profiles** — `id`, `user_id` (FK), `parsed_json_data` (JSONB)
- **applications** — `id`, `user_id` (FK), `job_title`, `company`, `job_description`, `ats_match_score`, `missing_skills` (JSONB), `selected_tone`, `generated_cover_letter`, `created_at`

## Design

Minimalist black-and-white dashboard aesthetic with:
- Fixed left vertical sidebar navigation
- Clean rectangular cards with subtle borders
- Grayscale palette with subtle green/red score indicators
- Responsive CSS Grid & Flexbox layout

## Setup

```bash
# Clone the repository
git clone https://github.com/Samuel-Girma07/cv-builder-platform.git
cd cv-builder-platform

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Edit .env with your PostgreSQL connection string, NVIDIA API key, and session secret

# Initialize the database
psql -U your_user -d your_db -f database/schema.sql

# Start the server
npm start
```

## Environment Variables

```env
DATABASE_URL=postgresql://user:password@localhost:5432/cv_builder
NVIDIA_API_KEY=your_nvidia_nim_api_key
SESSION_SECRET=your_session_secret
PORT=3000
```

## License

MIT
