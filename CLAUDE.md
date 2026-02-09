# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Repository Purpose

A Claude Code plugin for non-technical founders building SaaS applications with AI tools (Claude Code, Lovable, Replit, Cursor). Contains 36 skills covering the full build lifecycle plus business, marketing, and growth strategy.

## Structure

This is a Claude Code plugin. The manifest lives at `.claude-plugin/plugin.json`.

### `skills/` — All Skills

Each subdirectory is a skill containing `SKILL.md` (required) plus optional supporting files.

**Development Lifecycle:**
- **scope** - Turn ideas into AI-ready specs (Quick Feature Specs, Full Project Scopes)
- **design** - UX/UI patterns for SaaS (Laws of UX, component patterns, design systems)
- **build** - AI-assisted development workflows and tool selection
- **secure** - Security checklists and OWASP vulnerability guidance
- **optimize** - Speed, code, database, and dependency optimization
- **test** - Testing scenarios, edge cases, intentional bug hunting
- **debug** - Systematic debugging workflows and error interpretation
- **monitor** - Production health monitoring and incident response
- **deployment-going-live** - Hosting selection, domain setup, and going from local to production

**Strategy:** product-strategy-prioritization, pricing-monetization-strategy, saas-financial-modeling-metrics, saas-legal-business-formation

**Marketing & Growth:** seo-content-strategy, content-marketing-build-in-public, email-lifecycle-marketing, founder-led-sales-outreach, paid-acquisition-ad-copy, growth-plg, launch-go-to-market

**Product & Design:** ux-ui-design, onboarding-activation-design, information-architecture, customer-research-personas, customer-support-documentation

**Brand & Messaging:** brand-identity-visual-design, copywriting-microcopy, landing-page-strategy, market-research-competitive-analysis, conversion-rate-optimization

**Operations:** payments-billing-operations, churn-prevention-retention, hiring-technical-help

**Technical:** analytics-instrumentation, accessibility, technical-seo-expert

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
