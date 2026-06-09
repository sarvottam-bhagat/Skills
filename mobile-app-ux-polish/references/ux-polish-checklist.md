# UX Polish Checklist

Use this checklist to turn transcript-derived UX advice into practical app improvements.

## 1. Motion and Microinteractions

Look for places where the app currently jumps, swaps, or waits silently. Upgrade those moments with stateful motion.

High-leverage targets:

- Tab and page transitions: slide, fade, scale, or spring between sections instead of instantly replacing content.
- Loading and AI work: show "searching", "analyzing", source reveal, streaming, shimmer, or progress tied to the user's action.
- Mode switches: animate voice input, dictation, edit/send/checkmark states, and expanding controls from the tapped element.
- Completion: add satisfying confirmation, success haptic, checkmark morph, streak update, confetti-lite, or badge reveal.
- Drag/touch objects: add tilt, parallax, shine, depth, or a holographic feel for collectible badges or cards.

Plain-English implementation prompts:

- "Make the tab content slide horizontally with a small spring bounce when the selected tab changes. Tune it until it feels native and subtle."
- "When the AI starts analyzing, replace the static loading text with a searching state, animated gradient, and result sources that reveal one by one."
- "When dictation starts, expand a dark background from the microphone button, rotate the send icon into a checkmark, and add light haptic feedback."
- "When the user unlocks a badge, make it feel like a collectible sticker: slight tilt on drag, highlight following the finger, and a success haptic."

Quality bar:

- Motion should be noticeable in a screen recording but not distracting during normal use.
- The user should understand what changed, what is happening, or what they earned.
- No animation should delay a repeated workflow or block input unnecessarily.

## 2. Haptics and Touch Feedback

Add tactile feedback to reinforce important state changes.

Use haptics for:

- Selecting tabs, chips, filters, and segmented controls.
- Starting/stopping dictation or voice capture.
- Completing a task, saving an entry, logging a meal, or confirming a transaction.
- Crossing a drag threshold, unlocking a badge, or extending a streak.
- Error states that need attention, used sparingly.

Keep haptics light and consistent. Avoid making routine typing, scrolling, or every small tap vibrate.

## 3. Mascots, Illustrations, and Character

Use a mascot or illustration system when the app benefits from warmth, habit formation, coaching, progress, onboarding, or AI companionship.

Good uses:

- Onboarding screens that feel like a story instead of a form.
- Empty states that invite the next action.
- Streaks, badges, goals, and progress rituals.
- AI assistants, wellness apps, food tracking, planning, learning, finance coaching, and habit products.

Guidelines:

- Create a style that fits the app instead of copying a famous mascot.
- Mash multiple references only to extract qualities like depth, softness, posture, or expression; do not clone an existing character.
- Prefer a small reusable pose set: welcome, thinking, success, encouragement, warning, idle, and celebration.
- Animate a mascot lightly on splash, login, onboarding, or reward moments when it improves the perceived quality.

Implementation prompts:

- "Create an onboarding illustration system around this mascot with one pose per screen: welcome, log first entry, progress, and celebration."
- "Animate the mascot subtly on the login screen with a looping blink/breath motion that does not distract from signing in."
- "Replace this empty state with a character illustration, one concise line of copy, and a primary action."

## 4. Onboarding

Onboarding should show the transformation and make users curious about the next screen.

Audit for:

- Static or generic setup screens.
- Too much explanation before the user sees value.
- Missing character, motion, or product-specific imagery.
- No obvious bridge into the core loop or paywall.

Upgrade with:

- A tight sequence of screens that each shows one benefit, one action, or one emotional promise.
- Animated illustrations or mascot moments that make progression feel rewarding.
- Input collection only when it personalizes the first meaningful result.
- Permission priming before system dialogs.
- A handoff into the first completed action, not just an empty home screen.

## 5. Contextual Prompts and AI Surfaces

Never present a blank AI box, voice input, or command center without guidance.

Use:

- Suggested action chips such as "Plan my day", "What's next?", "Log breakfast", or "Summarize spending".
- Context-aware prompts based on the current screen, time, recent user behavior, and unfinished setup.
- Voice-first affordances when speaking is naturally faster than typing.

Prompt ideas:

- "Add 3 contextual chips above the input based on the user's current state and make the first one the most likely next action."
- "Design this assistant screen so a new user can succeed without typing a custom prompt."

## 6. Iconography

Icon quality is a fast way to make an app feel professional or cheap.

Rules:

- Use one icon set across the app whenever possible.
- Do not mix filled, outlined, rounded, sharp, thick, and thin icons without a clear system.
- A strong default pattern: inactive tab icons are outline; active tab icons become filled.
- Use icons that match the platform and product tone.
- Replace generic or mismatched AI-generated icons before shipping.

Good icon sources mentioned in the transcript:

- Heroicons
- Font Awesome
- Nucleo

## 7. Typography

Typography should make the app easier to scan thousands of times per month.

Audit:

- Too many font sizes or weights.
- Weak hierarchy between title, subtitle, metadata, labels, and values.
- Low contrast or tiny secondary text.
- Hero-sized type inside compact tool surfaces.
- Numeric values that do not align or scan cleanly.

Improve:

- Define a small type scale for large title, screen title, section title, body, label, caption, and numeric emphasis.
- Use weight and color to clarify hierarchy before adding more size.
- Keep line lengths and wrapping stable on small devices.
- Test the longest real labels, currencies, dates, food names, task names, and button text.

## 8. Widgets and OS Surfaces

Widgets make an app feel premium and can materially improve retention by occupying daily attention.

Consider:

- Home screen widgets for current progress, next task, calories left, budget remaining, habit streak, or today's plan.
- Lock screen widgets for glanceable status and one-tap deep links.
- Apple Watch apps or complications for quick capture and high-frequency checks.
- Deep links from every widget into the exact next action.

Quality bar:

- The widget should be useful without opening the app.
- The design should be recognizable as the app's brand.
- The widget should show live state, not generic marketing.
- It should help the user build or resume a habit.

Implementation prompt:

- "Design a small, medium, and lock screen widget for this app. Each should show the most useful current state and deep link into the next action."

## 9. App Store Screenshots

Screenshots are the first product experience for many users.

Audit:

- Do they communicate the app's main promise in the first screenshot?
- Do they show the distinctive interaction, not just generic screens?
- Do they include the mascot, visual identity, or motion concept when relevant?
- Are they legible at App Store thumbnail size?
- Do they look as polished as the app itself?

Upgrade:

- Lead with the outcome or "aha" moment.
- Show the core UI in realistic device frames only if it improves clarity.
- Use concise captions that sell the user's transformation.
- Keep visual identity consistent with the app, onboarding, and widgets.

## 10. Inspiration Loop

Continuously study excellent apps to build taste and identify tiny details worth adapting.

Sources mentioned in the transcript:

- Mobbin for screen patterns and app UI libraries.
- 60fps for interaction and animation references.
- Spotted in Prod for polished product animation examples.
- Screenshot-first curation accounts for App Store screenshot inspiration.

When using references:

- Identify the exact detail that feels good: timing, transition, icon treatment, density, hierarchy, state reveal, or gesture.
- Adapt the principle to the product rather than copying the surface.
- Record the screen and slow it down if the interaction feels good but is hard to describe.

## 11. Prioritization Matrix

If time is limited, prioritize in this order:

1. Primary action feedback: make the core loop feel responsive and alive.
2. Loading and AI states: replace static waiting with meaningful progress.
3. Onboarding first impression: add character, specificity, and a smooth handoff.
4. Icon and typography cleanup: remove obvious inconsistency.
5. Widget or lock screen surface: improve retention and perceived app quality.
6. App Store screenshots: improve install conversion.

## 12. Red Flags

Treat these as signals that the app may feel AI-generated or rushed:

- Instant screen swaps with no transition logic.
- Static loading text for important AI or processing moments.
- Mixed icon styles or inconsistent stroke widths.
- Blank assistant input with no suggested actions.
- Generic onboarding with no product-specific visual identity.
- No empty state design.
- No haptics on high-value touch moments.
- App Store screenshots that look like an afterthought.
- A competitive category app with no distinctive interaction pattern.
