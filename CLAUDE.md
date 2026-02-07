# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Repository Purpose

A comprehensive knowledge base for non-technical founders building SaaS applications with AI tools (Claude Code, Lovable, Replit, Cursor). Contains markdown guides, checklists, and reference material — no executable code.

## Structure

### `developer-skills/` — Software Development Lifecycle

Eight numbered directories following a build progression:

1. **1-scope** - Turn ideas into AI-ready specs (Quick Feature Specs, Full Project Scopes)
2. **2-design** - UX/UI patterns for SaaS (Laws of UX, component patterns, design systems)
3. **3-build** - AI-assisted development workflows and tool selection
4. **4-secure** - Security checklists and OWASP vulnerability guidance
5. **5-optimize** - Speed, code, database, and dependency optimization
6. **6-test** - Testing scenarios, edge cases, intentional bug hunting
7. **7-debug** - Systematic debugging workflows and error interpretation
8. **8-monitor** - Production health monitoring and incident response

Each directory contains:
- `SKILL.md` - Main overview and workflow (always under 500 lines)
- Supporting `[TOPIC].md` files - Detailed patterns, templates, and examples

Also includes `COMPLETE-SKILLS-SUMMARY.md` with a master index and cross-references.

### `non-code-skills/` — Business, Product & Marketing

22 single-file skills covering the non-coding side of building a SaaS:

- **Strategy:** product-strategy-prioritization, pricing-monetization-strategy, saas-financial-modeling-metrics, saas-legal-business-formation
- **Marketing & Growth:** seo-content-strategy, content-marketing-build-in-public, email-lifecycle-marketing, founder-led-sales-outreach, paid-acquisition-ad-copy, growth-plg
- **Product & Design:** ux-ui-design, onboarding-activation-design, information-architecture, customer-research-personas, customer-support-documentation
- **Brand & Messaging:** brand-identity-visual-design, copywriting-microcopy, landing-page-strategy, market-research-competitive-analysis, conversion-rate-optimization
- **Technical:** analytics-instrumentation, accessibility

Each contains a single `SKILL.md` with expert-level guidance for that domain.

### `commands/` — Claude Code Custom Commands

- `improve-prompt.md` - Transforms vague coding requests into detailed, specific prompts
- `README.md` - Installation and usage guide

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
