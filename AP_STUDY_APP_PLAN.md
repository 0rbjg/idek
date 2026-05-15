# AP Study App Product Blueprint

## 1) Core Product Idea
Build a web app that turns student content into active AP prep materials:
- Upload class notes (PDF, DOCX, TXT, images) or textbook pages.
- Paste YouTube links to generate transcript-backed study notes.
- Auto-generate AP-style practice:
  - FRQs with rubrics and sample high-scoring responses
  - MCQs with explanations
  - Flashcards with spaced repetition tags
- "Watch + Ask" study mode:
  - Left: video player
  - Bottom-left: transcript (click-to-seek)
  - Right: study chatbot that answers in AP course context

## 2) MVP Scope (ship in 6–8 weeks)
### Must-have
1. Authentication (email + Google sign-in).
2. Course selection (APUSH, AP Bio, AP Calc AB/BC, etc.).
3. File upload + text extraction (PDF/text/images with OCR).
4. YouTube link ingestion:
   - Pull transcript if available.
   - If no transcript, run speech-to-text on audio.
5. Content chunking + vector index.
6. Generation tools:
   - 5/10-question MCQ quiz modes.
   - FRQ generator (prompt + rubric + scoring guidance).
   - Flashcard deck generation.
7. Watch + Ask mode UI.
8. Saved sessions/history.

### Nice-to-have (post-MVP)
- Teacher mode and assignment sharing.
- AP exam date countdown and adaptive study plans.
- Multiplayer challenge mode.
- Voice chatbot replies.

## 3) Recommended Tech Stack
- Frontend: Next.js + TypeScript + Tailwind + shadcn/ui.
- Backend: Next.js API routes or FastAPI service.
- Database: Postgres (Supabase or Neon).
- Object storage: S3-compatible bucket.
- Search: pgvector or managed vector DB.
- Queue/workers: BullMQ / Redis for long-running transcript + OCR jobs.

## 4) AI Architecture
### Pipeline
1. **Ingestion**
   - Parse docs and OCR images.
   - Fetch or generate YouTube transcript.
2. **Normalization**
   - Clean text, remove junk, detect AP subject.
3. **Chunking**
   - Semantic chunking by heading/time window.
4. **Embedding + retrieval**
   - Store chunks with metadata (source, timestamp, unit/topic).
5. **Content generation**
   - MCQ generator prompt template.
   - FRQ generator prompt template with AP rubric style.
   - Flashcard generator prompt template with difficulty tags.
6. **Validation layer**
   - Reject low-confidence or hallucinated outputs.
   - Require evidence snippets from sources for each answer explanation.

### Model notes
- There is no publicly documented "ChatGPT 5.5" naming guarantee; use currently available OpenAI models from the API model list at build time.
- Practical setup:
  - Fast model for chat and drafting.
  - Stronger model for FRQ/rubric generation.
  - Embedding model for retrieval.

## 5) Feature Design Details
### A) Upload Notes/Textbook Pages
- Accept PDF, DOCX, TXT, PNG/JPG.
- OCR for scanned pages.
- Show extracted text preview before saving.
- Let users label material by AP course + unit.

### B) YouTube → Notes
- Input: URL.
- Extract transcript with timestamps.
- Generate:
  - concise notes
  - key terms
  - "likely exam concepts"
- Keep time-linked references so clicking a note jumps video.

### C) Practice Generator
- MCQ output format:
  - question stem
  - 4 options
  - correct answer
  - explanation citing uploaded source/transcript chunk
- FRQ output format:
  - AP-style prompt
  - scoring rubric categories
  - model response outline
  - optional scored user answer feedback

### D) Flashcards
- Auto-generate Q/A cards from key concepts.
- Include tags: course, unit, topic, difficulty.
- Spaced repetition scheduling (SM-2 starter algorithm).

### E) Watch + Ask Layout
- **Left panel:** YouTube player.
- **Bottom-left:** transcript with current sentence highlight.
- **Right panel:** chatbot conversation.
- Chatbot behaviors:
  - answers using retrieved transcript/notes only by default
  - can expand to general AP context when user asks
  - provides confidence indicator and cited snippets

## 6) Safety, Quality, and Academic Integrity
- Add disclaimer that generated materials are study aids.
- Avoid directly reproducing copyrighted textbook passages beyond fair-use snippets.
- Let students report bad questions/answers.
- Keep moderation filters on chat inputs/outputs.
- For minors, ensure strong privacy defaults and minimal data retention.

## 7) Data Model (starter)
- `users`
- `courses`
- `sources` (uploaded docs, videos)
- `source_chunks` (embedded text segments)
- `generated_assets` (mcq/frq/flashcards/notes)
- `study_sessions`
- `chat_messages`

## 8) API Endpoints (example)
- `POST /api/sources/upload`
- `POST /api/sources/youtube`
- `POST /api/generate/mcq`
- `POST /api/generate/frq`
- `POST /api/generate/flashcards`
- `POST /api/chat`
- `GET /api/transcript/:sourceId`

## 9) Suggested 8-Week Build Plan
1. Week 1–2: auth, course setup, upload pipeline.
2. Week 3: transcript ingestion + storage.
3. Week 4: embeddings + retrieval.
4. Week 5: MCQ/flashcard generation.
5. Week 6: FRQ + rubric + feedback.
6. Week 7: Watch + Ask UI integration.
7. Week 8: testing, analytics, polish.

## 10) Immediate Next Steps
1. Pick 2 AP subjects for MVP (e.g., APUSH + AP Bio).
2. Finalize the first FRQ and MCQ prompt templates.
3. Build ingestion and retrieval first; generation quality depends on this.
4. Add evaluation set (20 teacher-verified examples) before launch.


## 11) How to Test This (Practical Checklist)

### A) Quick smoke test (10–15 minutes)
1. Create account and select an AP course.
2. Upload one PDF or image page and confirm extracted text preview is accurate.
3. Paste one YouTube URL with transcript and verify notes are generated with timestamps.
4. Generate a 5-question MCQ set and confirm each question has an explanation.
5. Generate one FRQ and verify rubric categories are included.
6. Generate flashcards and confirm cards are tagged by course/unit/topic.
7. Open Watch + Ask mode and ask 3 content questions; verify answers cite transcript/notes.

### B) End-to-end functional tests by feature
- **Upload/OCR**
  - Test: PDF text, scanned image, blurry image.
  - Pass criteria: extracted text is readable; failures surface a clear error + retry.
- **YouTube ingestion**
  - Test: video with transcript, without transcript, short clip (<3 min), long lecture (30+ min).
  - Pass criteria: transcript stored with timestamps; fallback speech-to-text works when needed.
- **Retrieval quality**
  - Test: 20 fixed questions with known source snippets.
  - Pass criteria: at least 85% of answers cite the correct chunk.
- **MCQ quality**
  - Test: generate 100 MCQs across 2 AP subjects.
  - Pass criteria: >=90% single-correct-answer validity; >=80% teacher-rated "good" or better.
- **FRQ quality**
  - Test: generate 30 FRQs and rubrics, then teacher review.
  - Pass criteria: >=80% judged AP-style and aligned with rubric.
- **Flashcards**
  - Test: 100 cards sampled for duplicates and factual consistency.
  - Pass criteria: duplicate rate <10%; factual error rate <5%.
- **Watch + Ask UI**
  - Test: transcript click-to-seek + side chat on desktop/tablet widths.
  - Pass criteria: seek sync <1.5s; no layout overlap.

### C) API checks (example commands)
```bash
# 1) Upload a source file
curl -X POST http://localhost:3000/api/sources/upload   -F "file=@./fixtures/apush_ch3.pdf"   -F "course=APUSH"

# 2) Ingest YouTube link
curl -X POST http://localhost:3000/api/sources/youtube   -H "Content-Type: application/json"   -d '{"url":"https://www.youtube.com/watch?v=VIDEO_ID","course":"APUSH"}'

# 3) Generate MCQs
curl -X POST http://localhost:3000/api/generate/mcq   -H "Content-Type: application/json"   -d '{"sourceId":"SOURCE_ID","count":5}'

# 4) Generate FRQ
curl -X POST http://localhost:3000/api/generate/frq   -H "Content-Type: application/json"   -d '{"sourceId":"SOURCE_ID"}'

# 5) Generate flashcards
curl -X POST http://localhost:3000/api/generate/flashcards   -H "Content-Type: application/json"   -d '{"sourceId":"SOURCE_ID","count":20}'
```

### D) Non-functional testing
- **Latency targets**
  - Chat response p95: <4s
  - MCQ set generation (5 questions): <12s
  - FRQ generation: <20s
- **Reliability**
  - Failed ingestion jobs auto-retry up to 3 times.
  - Dead-letter queue for manual inspection.
- **Security/privacy**
  - Verify auth guards on all user data routes.
  - Confirm signed URLs for upload/download and encrypted storage at rest.

### E) Before-launch quality gate
Ship MVP only if all are true:
1. Smoke test passes with no blocking defects.
2. Teacher eval set (20+ examples) meets thresholds above.
3. No P1 security/privacy issues open.
4. Observability dashboards exist for ingestion, generation, and chat errors.
