# Generative Instagram with AI

![TS.3.3 Badge](https://img.shields.io/badge/TS.3.3-ORM-blue) ![TS.3.4 Badge](https://img.shields.io/badge/TS.3.4-API-green)

A modern twist on Instagram where images are **AI-generated with DALL·E 2**, published to a feed, and shared with the world — featuring **art styles, surprise prompts, and interactive hearts**.

---

## Table of Contents
- [Project Overview](#project-overview)  
- [Features](#features)  
- [Tech Stack](#tech-stack)  
- [Architecture](#architecture)  
- [Data Flow](#data-flow)  
- [Setup Instructions](#setup-instructions)  
- [Running the App](#running-the-app)  
- [Testing](#testing)  
- [Troubleshooting](#troubleshooting)  
- [License](#license)

---

## Project Overview

Instead of uploading photos, users can **generate images from text prompts using AI** and publish them to a **public feed**. Users can also **heart images**, with **animated effects**, and try **creative twists** like selecting an art style or pressing “Surprise Me” for fun prompts.  

This project demonstrates:
- Backend **ORM configuration with Prisma + Neon**
- **REST API design and data flow**
- Full integration with **OpenAI DALL·E 2**
- Testable, clean architecture

---

## Features

### Core Features (MVP)
- **AI Image Generation**
  - Users enter a prompt to generate an image using DALL·E 2
  - API endpoint: `POST /api/generate`
  - Validates prompt input
  - Returns `{ imageUrl, prompt }`

- **Publish Images**
  - Users can save generated images to PostgreSQL
  - API endpoint: `POST /api/publish`
  - Stores: `imageUrl`, `prompt`, `hearts` (default 0), `createdAt`
  - Returns full PublishedImage object

- **Public Feed**
  - API endpoint: `GET /api/feed`
  - Paginated and ordered by newest first
  - Default: 10 images per page, max 50
  - Returns `{ images: [], total, page, totalPages }`

- **Hearts System**
  - Users can “heart” images in feed
  - API endpoint: `PUT /api/feed`
  - Atomic update of `hearts` to prevent race conditions
  - Optimistic UI updates on frontend

### Creative Twists
- **Art Style Selection**
  - Users can choose styles like `realistic`, `cartoon`, `pixel art`, `anime`, `oil painting`
  - Style appended to prompt

- **Surprise Me**
  - Generates a random, fun AI prompt from a predefined list
  - Encourages creativity and engagement

- **Prompt History**
  - Frontend stores last 5 prompts for easy reuse

- **Animated Hearts**
  - Heart icon animates when clicked (CSS animation)

- **Optional Daily Challenge**
  - Preset prompts each day for users to generate and publish images
  - Showcased in feed

---

## Tech Stack

**Frontend**
- Next.js 16.1.1 (App Router)  
- React 19.2.3  
- CSS for styling (no Tailwind)  

**Backend**
- Next.js API Routes (Node.js)  
- Prisma ORM 7.2.0 with Neon adapter  
- PostgreSQL via Neon.com  

**AI Service**
- OpenAI DALL·E 2 API  

**Testing**
- Vitest 4.0.16  

**Package Manager**
- pnpm

---

## Architecture

Client (Next.js / React)
│
├── Generate Page (/app/page.js)
│ ├─ Prompt Input
│ ├─ Generate Button -> POST /api/generate
│ ├─ Loading State
│ ├─ Display Generated Image
│ └─ Publish Button -> POST /api/publish
│
├── Feed Page (/app/feed/page.js)
│ ├─ Fetch Images -> GET /api/feed
│ ├─ Display Images Grid
│ ├─ Heart Button -> PUT /api/feed
│ └─ Pagination Controls
│
API Routes (Next.js / Node.js)
│
├── /api/generate
│ └─ Calls OpenAI DALL·E 2
│
├── /api/publish
│ └─ Saves image & prompt to database
│
└── /api/feed
├─ GET -> returns paginated images
└─ PUT -> updates hearts atomically

Database (PostgreSQL via Neon)
└── published_images
├─ id (Int, PK)
├─ imageUrl (String)
├─ prompt (String)
├─ hearts (Int, default 0)
└─ createdAt (DateTime, default now())


---

## Data Flow

1. **Generate Image**
   - User submits prompt → `/api/generate` → DALL·E 2 → returns imageUrl → frontend displays

2. **Publish Image**
   - User clicks Publish → `/api/publish` → Prisma creates `PublishedImage` → database stores record → frontend confirms success

3. **Feed Retrieval**
   - Feed page loads → `/api/feed` GET → Prisma queries `published_images` with `skip` + `take` → frontend displays grid

4. **Heart Update**
   - User clicks heart → `/api/feed` PUT → Prisma updates hearts atomically → frontend updates UI optimistically

---

## Setup Instructions

### Prerequisites
- Node.js 18+  
- pnpm (`npm install -g pnpm`)  
- Neon.com account (free tier)  
- OpenAI API key with DALL·E 2 access  

### Install & Configure
```bash
git clone <your-repo-url>
cd <repo-name>
pnpm install

# Environment
cp .env.example .env
# Add your credentials:
# DATABASE_URL='postgresql://user:password@host/database?sslmode=require'
# OPENAI_API_KEY='sk-...'