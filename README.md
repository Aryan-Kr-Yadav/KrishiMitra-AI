# 🌾 KrishiMitra AI — Smart Farming Companion

> An AI-powered agricultural assistant that helps Indian farmers detect crop diseases, understand treatments, and access government schemes — in Hindi and English.
>
> Built for the **Agents for Good** hackathon track.

![Status](https://img.shields.io/badge/status-hackathon--ready-10b981) ![License](https://img.shields.io/badge/license-MIT-blue)

---

## ✨ What it does

- 📷 **Image Diagnosis** — upload or drag-and-drop a crop photo, get an AI disease diagnosis powered by Gemini's multimodal model.
- 💬 **Symptom Diagnosis** — no camera? describe the symptoms in plain text instead.
- 🩺 **Full Treatment Plans** — organic + chemical treatment options, preventive measures, next steps, and guidance on when to see a real expert.
- 🏛️ **Government Schemes Finder** — pick a state + crop, get relevant Indian government schemes with eligibility, benefits, documents, and links.
- 🌐 **Hindi & English** — full UI and AI-response language toggle.
- 🌓 **Dark mode**, **PDF report download**, toast notifications, skeleton loaders, and mobile-first responsive design.

---

## 🧱 Tech Stack

| Layer | Tech |
|---|---|
| Frontend | React + Vite + Tailwind CSS + React Router + Framer Motion + Lucide Icons |
| Backend | Node.js + Express.js + Multer + CORS + Helmet |
| AI | Google Gemini API (`gemini-1.5-flash` for vision, `gemini-1.5-pro` for text reasoning) |
| Deployment | Frontend → Vercel · Backend → Render |

> **Note on "Gemini Vision":** Google's standalone `gemini-pro-vision` model has been retired — current Gemini 1.5 models are natively multimodal (text + image in one model). The app uses `GEMINI_VISION_MODEL` and `GEMINI_TEXT_MODEL` env vars so you can swap to whichever current model fits, including the older naming if your account has access to it.

---

## 📁 Project Structure

```
krishimitra-ai/
├── backend/
│   ├── server.js                 # Express app entry point
│   ├── routes/                   # diagnose.js, schemes.js, health.js
│   ├── controllers/               # request handlers
│   ├── services/geminiService.js  # Gemini API wrapper + JSON parsing
│   ├── middleware/                # upload, rate limiting, validation, errors
│   ├── utils/                     # prompt templates, response formatter, logger
│   ├── uploads/                   # multer scratch dir (gitignored)
│   ├── package.json
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── components/    # Navbar, Footer, UploadCard, SchemeCard, DiagnosisCard, etc.
│   │   ├── pages/          # LandingPage, DiagnosePage, ResultsPage, SchemesPage, AboutPage
│   │   ├── context/        # LanguageContext, ThemeContext
│   │   ├── hooks/          # useToast, useLocalStorage
│   │   ├── services/api.js # Axios client + API calls
│   │   ├── utils/          # constants, validators, image compression, PDF generator
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── index.html
│   ├── tailwind.config.js
│   ├── vite.config.js
│   ├── package.json
│   └── .env.example
│
└── README.md
```

---

## 🚀 Quick Start (Local Development)

### Prerequisites
- Node.js 18+
- A free [Gemini API key](https://aistudio.google.com/app/apikey)

### 1. Clone & install

```bash
# Backend
cd backend
npm install
cp .env.example .env
# then edit .env and paste your GEMINI_API_KEY

# Frontend (in a new terminal)
cd frontend
npm install
cp .env.example .env
```

### 2. Run both servers

```bash
# Terminal 1 — backend (http://localhost:5000)
cd backend
npm run dev

# Terminal 2 — frontend (http://localhost:5173)
cd frontend
npm run dev
```

Open **http://localhost:5173** — the frontend is pre-configured to call the backend at `http://localhost:5000/api`.

---

## 🔑 Environment Variables

### `backend/.env`
```env
PORT=5000
NODE_ENV=development
FRONTEND_URL=http://localhost:5173
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_VISION_MODEL=gemini-1.5-flash
GEMINI_TEXT_MODEL=gemini-1.5-pro
MAX_UPLOAD_SIZE=5242880
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

### `frontend/.env`
```env
VITE_API_BASE_URL=http://localhost:5000/api
```

---

## 📡 API Reference

All responses follow the shape:
```json
{ "success": true, "message": "...", "data": { ... }, "timestamp": "..." }
```

| Method | Endpoint | Body | Description |
|---|---|---|---|
| `GET` | `/api/health` | – | Health check + Gemini config status |
| `POST` | `/api/diagnose/image` | `multipart/form-data`: `image`, `cropType`, `language` | Diagnose disease from a photo |
| `POST` | `/api/diagnose/symptoms` | `json`: `{ symptoms, cropType, language }` | Diagnose disease from text |
| `POST` | `/api/schemes` | `json`: `{ state, crop, language }` | Get relevant government schemes |

**Diagnosis response shape (`data`):**
```json
{
  "diseaseName": "Early Blight",
  "confidencePercentage": 87,
  "severity": "Medium",
  "symptoms": ["..."],
  "treatment": { "organic": ["..."], "chemical": ["..."] },
  "preventiveMeasures": ["..."],
  "immediateNextSteps": ["..."],
  "consultExpertIf": "...",
  "limitations": null,
  "isHealthy": false
}
```

**Schemes response shape (`data`):**
```json
{
  "schemes": [
    {
      "schemeName": "PM-KISAN",
      "description": "...",
      "eligibility": ["..."],
      "benefits": ["..."],
      "requiredDocuments": ["..."],
      "officialWebsite": "https://pmkisan.gov.in",
      "applicationProcess": "..."
    }
  ]
}
```

---

## 🧠 Gemini Prompts

Prompt templates live in `backend/utils/promptTemplates.js` and are kept separate from API-call logic so they're easy to tune. They instruct Gemini to:
- Act as an expert agricultural advisor for Indian crops.
- Return **only** a strict JSON object (no markdown fences) matching a fixed schema.
- Never recommend unsafe or excessive chemical dosages.
- Honestly lower its confidence score and flag limitations when uncertain.
- Respond in the farmer's selected language (Hindi or English) while keeping JSON keys in English.

`geminiService.js` defensively strips any stray markdown fences and falls back to extracting the outermost `{...}` block before parsing, so minor formatting quirks from the model don't break the app.

---

## 🛡️ Security

- **Rate limiting** — general `/api/*` limiter + a stricter limiter on the AI endpoints (`express-rate-limit`).
- **File validation** — only JPG/PNG accepted, 5MB server-side limit (`multer`), plus an 8MB client-side pre-check and automatic compression.
- **Helmet** — sensible security headers by default.
- **CORS** — explicit allow-list via `FRONTEND_URL` (comma-separated for multiple origins).
- **Input sanitization** — symptom/state/crop text is trimmed and stripped of angle brackets before use.
- **Secrets** — `GEMINI_API_KEY` is never exposed to the frontend; all Gemini calls happen server-side.

---

## ☁️ Deployment

### Backend → Render

1. Push this repo to GitHub.
2. On [Render](https://render.com), click **New → Web Service**, connect the repo, and set:
   - **Root Directory:** `backend`
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
   - **Environment:** add all variables from `backend/.env.example` (use your real `GEMINI_API_KEY`, and set `FRONTEND_URL` to your deployed Vercel URL once you have it).
3. Deploy. Note your Render URL, e.g. `https://krishimitra-backend.onrender.com`.

### Frontend → Vercel

1. On [Vercel](https://vercel.com), click **New Project**, import the repo, and set:
   - **Root Directory:** `frontend`
   - **Framework Preset:** Vite
   - **Build Command:** `npm run build`
   - **Output Directory:** `dist`
   - **Environment Variable:** `VITE_API_BASE_URL=https://krishimitra-backend.onrender.com/api`
2. Deploy. Then go back to Render and update `FRONTEND_URL` to your Vercel domain so CORS allows it.

---

## 🗺️ Pages

| Route | Page |
|---|---|
| `/` | Landing page — hero, features, how it works, benefits, testimonials, FAQ |
| `/diagnose` | Image + symptom diagnosis tabs |
| `/results` | Diagnosis results with PDF download |
| `/schemes` | Government scheme search |
| `/about` | Mission, problem, impact, team, tech stack |

---

## ⚠️ Disclaimer

KrishiMitra AI provides AI-generated guidance to support — not replace — professional agricultural advice. Always consult a local agricultural extension officer or expert before applying chemical treatments, especially for high-severity cases. Scheme information should be verified on official government portals before applying.

---

Built with 💚 for Indian farmers — Agents for Good hackathon track.
#   K r i s h i M i t r a - A I  
 