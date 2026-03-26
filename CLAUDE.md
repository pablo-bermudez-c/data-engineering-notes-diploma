# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About This Repository

Educational notes repository for a data engineering diploma (USACH). All content is written in **Spanish** with technical terms presented in Spanish followed by the English equivalent in parentheses (e.g., "rama (*branch*)"). Licensed under CC BY-NC 4.0.

## Repository Structure

- `diploma-courses/` — Notes and summaries from the diploma program courses
  - `modulo-01-fundamentos/` — Foundational concepts: IDEs, VS Code, AI assistants, SDD, TDD
- `tools-and-practices/` — Practical tutorials organized as sequential modules with numbered classes (`clase-XX-topic/README.md`)
  - `github/` — Git and GitHub tutorial (8 classes)
  - `python-ambientes-virtuales/` — Python virtual environments with uv (5 classes)

## Content Conventions

- This is a **documentation-only** repo — no source code, no build system, no tests
- All content lives in `README.md` files within class directories
- Tutorials are ordered sequentially and designed to be followed in order
- Each class includes: learning objectives, explanations with examples, and exercises

## Writing New Content

- Follow the existing pattern: `module/clase-XX-descriptive-name/README.md`
- Keep Spanish as the primary language; include English terms in parentheses for technical vocabulary
- Add new classes to the parent module's README table
- Add new modules to `tools-and-practices/README.md`
