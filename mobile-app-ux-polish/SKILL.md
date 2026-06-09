---
name: mobile-app-ux-polish
description: "Improve mobile app UX, retention, and perceived quality with high-impact polish patterns: animations, microinteractions, haptics, contextual prompts, mascots, illustrations, iconography, typography, onboarding, widgets, Apple Watch/OS surfaces, and App Store screenshots. Use when Codex is building, auditing, redesigning, or prompting improvements for iOS, Android, React Native, SwiftUI, Flutter, or mobile-first app experiences, especially when the user wants the app to feel premium, memorable, less static, less AI-generated, more delightful, more retentive, or more likely to stand out in a competitive category."
---

# Mobile App UX Polish

## Overview

Use this skill to make a mobile app feel alive, intentional, and memorable. Focus on the small details users feel before they can name them: motion, feedback, clarity, character, and daily-use surfaces.

## Workflow

1. Identify the app's core loop, target user, main competing apps, and the screens where users first decide quality: onboarding, home, primary action, success/error states, settings, paywall, widgets, and App Store screenshots.
2. Audit the current experience against the polish checklist in `references/ux-polish-checklist.md`.
3. Choose a short list of changes that compound across the product: one motion system, one interaction upgrade, one visual identity upgrade, one retention surface, and one first-impression upgrade.
4. Implement polish as product behavior, not decoration. Tie every animation, haptic, illustration, or widget to a state change, habit loop, user confidence moment, or repeated action.
5. Verify the feel on a real device or simulator when possible. Use screenshots for layout, screen recordings for motion, and repeated tapping/swiping to catch timing, bounce, clipping, overlap, and stale states.

## Audit Output

When asked to review or improve an app, produce:

- **Quality read**: What makes the app feel static, generic, inconsistent, or unfinished.
- **High-impact fixes**: 5-8 ranked changes with expected impact and implementation effort.
- **Interaction prompts**: Plain-English prompts the user can give to an AI coding agent for each improvement.
- **Verification notes**: What to check in screenshots, recordings, simulator, or device testing.

## Implementation Principles

- Prefer motion that explains state: searching, analyzing, saving, unlocking, completing, switching, dragging, expanding, or confirming.
- Keep microinteractions subtle enough to feel native. Tune duration, spring, damping, haptics, opacity, and easing after the first implementation.
- Add life to repeated transitions: tab switches, page changes, sheets, input mode changes, dictation, result loading, streaks, badges, empty states, and completion moments.
- Use haptics where the platform supports them, especially for selection, success, threshold crossing, drag feedback, and habit rewards.
- Avoid blank AI or voice-first surfaces. Add contextual chips or suggested commands that show what users can do next.
- Treat mascot, illustration, and icon style as product identity. Keep them consistent with the category and emotional promise of the app.
- Use one icon family and one icon treatment unless there is a deliberate selected/unselected state pattern.
- Treat widgets, lock screen widgets, complications, and watch apps as retention surfaces, not extras. They should reveal the current state and deep link into the next useful action.
- Treat App Store screenshots like the app's title and thumbnail. They must sell the core interaction, outcome, and visual identity before the user installs.

## Reference

Read `references/ux-polish-checklist.md` when doing a detailed audit, designing a mobile app flow, improving onboarding, adding retention surfaces, writing implementation prompts, or planning screenshots.
