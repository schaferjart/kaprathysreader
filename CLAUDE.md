# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Self-hosted EPUB reader web app designed for reading books alongside LLMs. Two-stage architecture: preprocess EPUBs into pickle files, then serve via a web UI.

## Commands

```bash
uv run reader3.py <file.epub>   # Process an EPUB into {bookname}_data/book.pkl + images
uv run server.py                # Start FastAPI server on localhost:8123
```

Package manager is `uv`. Python 3.10+. No tests exist.

## Architecture

**`reader3.py`** — CLI script that processes EPUB files:
- Extracts HTML chapters via ebooklib, cleans them with BeautifulSoup (strips scripts, styles, comments)
- Extracts images, rewrites paths
- Parses TOC recursively
- Serializes a `Book` dataclass (containing `BookMetadata`, `List[ChapterContent]` spine, `List[TOCEntry]` toc, images dict) to pickle

**`server.py`** — FastAPI app with routes:
- `/` — library view (lists all `*_data/book.pkl` files)
- `/read/{book_id}` — redirects to first chapter
- `/read/{book_id}/{chapter_index}` — reader view
- `/read/{book_id}/images/{image_name}` — serves extracted images
- Books loaded via `@lru_cache`

**`templates/`** — Jinja2 templates:
- `library.html` — card grid of processed books
- `reader.html` — split-screen layout (sidebar TOC + chapter content), prev/next navigation, `findAndGo()` JS function maps TOC filenames to spine indices
