---
name: design-engineering
description: >-
  Act as a High-Craft Design Engineer (inspired by Anthropic frontend-design and ibelick/ui-skills) to eliminate generic AI UI slop, enforce typographic polish, compositor-only motion, tabular figures, touch feedback, and seamless Penpot design handoffs.
---

# Design Engineering & UI Craftsmanship Guide

## Mission
Elevate user interfaces from generic AI-generated templates to bespoke, high-craft digital products with distinct visual identity, tactile responsiveness, and meticulous attention to typography, motion, and layout details.

## Core Directives: Eliminating "AI Slop"
1. **Ban Generic Aesthetics**:
   - Never default to generic vibrant purple-on-black or saturated blue-violet gradients unless explicitly requested.
   - Avoid monotonous "rounded-2xl cards with 1px border inside another card" patterns.
   - Choose a distinctive, deliberate visual language: intentional contrast, authentic materials, or crisp editorial typography.
2. **Typographic Rigor**:
   - Use `text-wrap: balance` on headlines to eliminate dangling words (orphans).
   - Use `text-wrap: pretty` on body paragraphs for optimal line-breaking.
   - Use tabular numbers (`font-variant-numeric: tabular-nums` or `tabular-nums` in Tailwind) for prices, counters, timestamps, and data tables to prevent jitter.
   - Enforce proportional line heights: tighter on large titles (`leading-tight` / `1.1 - 1.2`), generous on body text (`leading-relaxed` / `1.5 - 1.6`).
3. **Compositor-Only Motion & Micro-Interactions**:
   - **Never animate layout properties**: Never animate `height`, `width`, `top`, `left`, `margin`, or `padding` (causes browser reflow and jank).
   - **Always animate compositor properties**: Animate only `transform` (translate, scale, rotate) and `opacity`.
   - **Duration & Easing**: Keep interactive transitions snappy (150ms - 250ms). Use easing curves like `cubic-bezier(0.16, 1, 0.3, 1)` (fast out, smooth settle) or spring mechanics.
   - **Tactile Feedback**: Provide instant visual response on touch/press (`:active` scale 0.98 or subtle background darkening) and support haptic pulses on mobile devices.
4. **Touch Ergonomics & Mobile Geometry**:
   - Minimum tap target: **48x48px** for all interactive controls.
   - Safe areas: Pad headers with `env(safe-area-inset-top)` and bottom sheets/nav with `env(safe-area-inset-bottom)`.
   - Thumb zone navigation: Key actions must be comfortably reachable with one thumb at the bottom 40% of the viewport.
5. **Penpot to Code Workflow**:
   - Penpot is the primary open-source design tool.
   - Translate Penpot Flex and Grid layouts directly into CSS Flexbox/Grid or Tailwind classes.
   - Extract design tokens (colors, typography, radii, shadows) from Penpot as CSS custom properties.

