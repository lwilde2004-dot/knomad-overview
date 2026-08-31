# Knomad

Upload a lecture, get something you can actually revise from. Knomad takes a PDF, PowerPoint or Word file and turns it into a structured wiki article, a set of practice questions, and a chatbot that answers from your own material rather than from the open internet.

**Live at [knomadstudy.co.uk](https://knomadstudy.co.uk).** Solo build — architecture, backend, frontend, deployment and cost control.

This repository is the overview. The application source is private.

---

## Why

I was already doing this by hand for my own degree: taking raw lecture files and rebuilding them into notes, flashcards and questions worth revising from. Knomad is that process productised, and made to work for any subject rather than just mine.

## What it does

1. **Ingest.** Accepts PDF, PPTX and DOCX, extracts text and structure, and stores the result against your account.
2. **Generate.** Produces a wiki-style article, then practice questions from the same source.
3. **Ask.** A question-answering layer scoped to your uploaded material, so answers cite what you gave it.
4. **Edit.** A rich-text editor with proper maths rendering, because engineering notes are unusable without it.

## Architecture

**Backend** — Node/Express with SQLite (better-sqlite3) on a persistent volume. Document parsing runs through `pdf-parse`, `officeparser`, `mammoth`, and JSZip with `fast-xml-parser` for PowerPoint XML. Auth is Clerk middleware applied globally with per-route guards. `helmet` and `express-rate-limit` on the edge. Billing runs on webhooks verified with `svix`.

Routes are split by concern: `upload`, `process`, `vault`, `folders`, `qa`, `questions`, `conversations`, `billing`, `webhooks`, `admin`, `feedback`.

**Frontend** — React 19 on Vite. TipTap for the editor with the mathematics extension, KaTeX for rendering, `dnd-kit` for reordering, and a markdown pipeline (`remark-math`, `rehype-katex`, `rehype-sanitize`) for generated content.

**Deployment** — frontend on Vercel, backend on Railway with a mounted volume holding the database and uploads.

## Decisions worth explaining

**SQLite, not Postgres.** Reads dominate, the write pattern is one user at a time against their own material, and a file-backed database on a mounted volume removes an entire service from the stack. It caps me at vertical scaling, and I would have to migrate before multi-region. For the current load that trade is clearly right.

**Model routing by task, not one model for everything.** Classification and article generation go to a cheap model; derivations and proof-style questions go to a stronger one. Quality only has to be paid for where it changes the answer.

**Hard budget caps.** Per-user monthly spend limits and a per-query ceiling, enforced server-side. An AI feature that bills per request is a runaway cost waiting to happen, and a student product cannot absorb that.

**Text extraction before generation.** Documents are parsed to structured text and stored first, so regenerating an article never re-parses the upload and never re-bills the user for work already done.

## Status

In active development. Working end to end and deployed; the roadmap is a vault redesign around folders, better conversation history, and a tiered plan structure.
