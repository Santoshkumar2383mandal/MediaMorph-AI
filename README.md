# Social Media Content Analyzer

An AI-powered web application that analyzes uploaded social media content (from PDFs or images) and provides actionable insights to improve engagement.

## Features

### 📄 Document Upload
- **PDF Files**: Extract text from PDF documents with formatting preservation
- **Image Files**: Support for PNG, JPG, JPEG, and WEBP formats
- **Drag & Drop**: Intuitive drag-and-drop interface
- **File Picker**: Traditional file selection button

### 🔍 Text Extraction
- **PDF Parsing**: Server-side `pdf-parse` keeps paragraphs/line breaks intact
- **Image OCR**: Lightweight Express service wraps `tesseract.js` for fast on-device OCR

### 🤖 AI-Powered Analysis
- **Gemini 2.0 Flash** (optional): Generates summary, score, suggestions, ideas, hashtags
- **Rule-Based Fallback**: Deterministic heuristics kick in if Gemini fails or no key provided
- **Engagement Metrics**: Keeps the extracted text visible and copyable for review

### ✨ User Experience
- **Loading States**: Visual feedback during file processing
- **Error Handling**: Clear error messages for invalid files or processing failures
- **Modern UI**: Beautiful, responsive design with smooth animations
- **Copy to Clipboard**: Easy text copying functionality

## How It Works

1. **Upload** a PDF or image via drag-and-drop or the file picker (5 MB cap)
2. **Extract Text**
   - PDFs stay on the Next.js API route and go through `pdf-parse`
   - Images are streamed to the standalone `ocr-backend` (Express + Tesseract)
3. **Analyze**
   - Normalized text is sent to Gemini (when `GEMINI_API_KEY` is present)
   - Otherwise, the rule-based engine scores the copy locally
4. **Display** the insights, suggestions, ideas, hashtags, and original text

## Prerequisites

- Node.js 18+
- npm (or pnpm / bun / yarn)
- Google Gemini API Key (optional but recommended) – [get one here](https://makersuite.google.com/app/apikey)

## Installation & Local Dev

1. **Install root dependencies**
   ```bash
   npm install
   ```

2. **Install OCR backend dependencies**
   ```bash
   cd ocr-backend
   npm install
   ```

3. **Environment variables** (`.env.local`)
   ```env
   GEMINI_API_KEY=your_gemini_api_key   # optional – enables AI summaries
   OCR_BACKEND_URL=http://localhost:5001
   ```

4. **Start the OCR backend**
   ```bash
   cd ocr-backend
   npm start
   ```

5. **Start the Next.js app (root folder)**
   ```bash
   npm run dev
   ```

6. Visit [http://localhost:3000](http://localhost:3000) and upload a PDF/image.

> Need to verify Tesseract is installed properly? Run `node scripts/test-tesseract.js`.

## Project Structure

```
sm-zip/
├── src/
│   ├── app/
│   │   ├── api/analyze/route.ts   # Upload parsing + AI/rule analysis
│   │   └── page.tsx               # Landing page + workflow
│   ├── components/
│   │   ├── upload-zone.tsx        # Drag/drop UI + manual analyze flow
│   │   └── analysis-result.tsx    # Insight cards
│   └── lib/utils.ts
├── ocr-backend/
│   ├── server.js                  # Express + multer + tesseract.js
│   └── package.json
├── scripts/test-tesseract.js      # Smoke test for OCR worker
├── README.md
└── …
```

## Technologies Used

- **Next.js 16 + TypeScript** for the UI & API routes
- **Tailwind, Framer Motion, React Dropzone** for modern UX
- **pdf-parse** for PDF text extraction
- **Express + Tesseract.js** for OCR (deployed separately)
- **Google Gemini 2.0 Flash** (optional) for deeper analysis

## API Endpoint

`POST /api/analyze`

- Accepts `FormData` with a single `file`
- Detects PDF vs. image and routes accordingly
- Responds with:
  ```json
  {
    "text": "Extracted text content...",
    "analysis": {
      "summary": "string",
      "score": 72,
      "suggestions": ["string"],
      "trendingIdeas": ["string"],
      "hashtags": ["#tag"]
    }
  }
  ```

## Error Handling

- Frontend enforces file type + size and shows inline errors
- Upload API validates MIME type, size, extraction length, OCR failures
- Gemini failures fall back to rule-based analysis instead of crashing
- Detailed logs are written on the server for PDF/OCR/AI issues

## Building & Deployment

```bash
npm run build   # Next.js production build
npm start       # start Next.js server
```

- Deploy the Next.js app to **Vercel** (or any Node host)
  - Set `GEMINI_API_KEY` and `OCR_BACKEND_URL=https://your-ocr-host/ocr`
- Deploy the OCR backend (Render / Railway / Fly.io, etc.)
  - Set `PORT` environment variable if required by the platform
  - Mount `eng.traineddata` or ensure the container can download it
- Update the README with the production URLs once deployed

## Limitations

- OCR quality depends on source clarity; blurry photos may fail
- PDF uploads larger than 5 MB are rejected to avoid timeouts
- Gemini usage requires a valid API key and counts against your quota
- Currently supports a single file per analysis

## Approach & Tools (≤200 words)

I built the analyzer with a pragmatic split between quick heuristics and optional AI. All uploads hit a single Next.js API route, which inspects MIME type, enforces the 5 MB guardrail, and sends PDFs to `pdf-parse`. Images are streamed to a tiny Express service (Tesseract.js + Multer on memory storage) so the heavy OCR work stays out of the frontend bundle. The extracted text is normalized before analysis but the raw version is returned so users can keep the original formatting.

For insights, I default to a deterministic scoring engine that counts words, hashtags, links, and emojis to produce actionable suggestions, trending prompts, and hashtag lists. When a Gemini 2.0 Flash key is present, the same text is sent to the API; if it succeeds, its richer response replaces the fallback. Failures simply fall back to the heuristic output, keeping the UX snappy and reliable. The upload surface combines drag-and-drop with a confirmatory “Analyze” action, displays file metadata, and surfaces inline errors. Animations (Framer Motion) and Tailwind utility styles keep the UI polished while maintaining production-grade readability in the codebase.
