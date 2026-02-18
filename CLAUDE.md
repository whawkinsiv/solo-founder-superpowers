# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Repository Purpose

A Claude Code plugin for non-technical founders building SaaS applications with AI tools (Claude Code, Lovable, Replit, Cursor). Contains 43 skills covering the full build lifecycle plus business, marketing, and growth strategy.

## Structure

This is a Claude Code plugin. The manifest lives at `.claude-plugin/plugin.json`.

### `skills/` — All Skills

Each subdirectory is a skill containing `SKILL.md` (required) plus optional supporting files.

**Development & Technical:** build, database, integrations, ai-features, deploy, secure, test, debug, optimize, monitor

**Design & UX:** design, brand, navigation, onboarding, accessibility

**SEO & Content:** seo, technical-seo, content, copywriting

**Growth & Marketing:** launch, landing-page, growth, conversion, email, ads, sales, social-media, community

**Strategy & Planning:** plan, validate, customer-research, market-research, pricing, prioritize, feedback, analytics

**Business Operations:** legal, finances, accounting, payments, hiring, support, retention

### `commands/` — Claude Code Custom Commands

- `improve-prompt.md` - Transforms vague coding requests into detailed, specific prompts

## File Conventions

- All files use ALL CAPS with hyphens: `DEBUG-PROMPTS.md`, not `debug_prompts.md`
- SKILL.md files use YAML frontmatter with `name` and `description` fields
- Skills are designed for progressive disclosure: SKILL.md first, then supporting files
- Never more than 1 level deep from SKILL.md

## Design Philosophy

These skills assume Claude's intelligence — they focus on:
- Non-technical founder perspective and common mistakes
- Tool selection criteria (when to use Lovable vs Claude Code vs Replit)
- Actionable checklists and "Tell AI:" copy-paste prompts
- What's out of scope (preventing premature optimization)
- Concise, actionable content that avoids explaining concepts Claude already knows
