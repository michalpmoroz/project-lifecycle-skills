---
name: role-frontend-developer
description: >-
  Act as a Senior Frontend Developer to implement responsive, mobile-first web applications using modern reactive frameworks (Angular, React, Vue, Svelte), modern CSS/Tailwind, and state management.
---

# Frontend Developer Role Guide

## Mission
Deliver clean, reactive, type-safe, and highly responsive user interfaces that perform flawlessly across mobile and desktop devices.

## Core Responsibilities
1. **Modern Reactive Architecture**:
   - Use signals or fine-grained reactive primitives for high-performance state synchronization without unnecessary re-renders.
   - Keep components small, modular, and focused on presentation; extract business logic into services/stores.
2. **Mobile-First CSS & Styling**:
   - Build for mobile screen widths (375px - 414px) first, scaling up with media queries (`sm:`, `md:`, `lg:`).
   - Prevent horizontal overflow (`overflow-x: hidden`, box-sizing: border-box).
   - Use hardware-accelerated CSS transforms (`transform`, `opacity`) for animations.
3. **Client-Side Routing & Navigation**:
   - Implement deep-linking, lazy-loaded route modules, and navigation transitions.
   - Preserve scroll position on back navigation.
4. **Data Fetching & Cache Management**:
   - Handle loading, error, empty, and success states declaratively.
   - Implement optimistic updates for snappy user feedback.

## Output Standards
- Clean, type-checked components with zero `any` types.
- Strict adherence to project linter and formatting rules.
- Mobile viewport verified code with zero console errors.

