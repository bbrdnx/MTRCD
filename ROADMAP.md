# MTRCD Product Roadmap

**North star:** Make MTRCD the tool engineers and designers actually have open while they work — not the one they read once and bookmark. The official WCAG docs are a specification. We're a practice guide.

**What we're not building:** Another automated scanner. axe-core, Lighthouse, and Deque exist. Our value is human-readable context, workflow integration, and the judgment calls that tools can't make.

---

## 1. Testing Guidance Per Criterion

### The story

Right now each card tells you *what* the rule is. What it doesn't tell you is *how to verify* that your implementation actually passes. This is the most immediate gap between MTRCD and daily usefulness for engineers.

A developer implementing a custom dropdown shouldn't have to consult three separate resources to know: Tab to it, press Enter to open, use arrow keys to navigate, press Escape to close — and here's how to confirm that with a screen reader. That full picture should live on the 2.1.1 card.

### Build plan

Add a "How to test" section to every card with three layers:

1. **Keyboard test** — Exact key sequence to run. Written as steps, not prose. ("1. Tab to the element. 2. Press Enter. 3. Confirm the action fires on key up, not key down.")
2. **Screen reader test** — Specific commands for NVDA+Chrome and VoiceOver+Safari, the two most common pairings. What announcement to listen for. What failure sounds like.
3. **Automated test** — The axe-core rule ID that covers this criterion (where one exists), plus a note for criteria that have no automated coverage, since those are the ones most likely to slip through CI.

This is entirely a content build — no architectural changes required. The card UI already has sections. It's 56 × 3 pieces of content. The first pass can focus on Level A + the most commonly violated AA criteria (1.4.3, 2.4.7, 2.4.11, 1.4.11).

### Open decisions

**Which screen readers?** NVDA+Chrome is the most common among users with disabilities. VoiceOver+Safari dominates on mobile. JAWS is prevalent in enterprise. Including all three is thorough but triples the content. Recommendation: NVDA and VoiceOver to start — cover JAWS in a Pro tier.

**How prescriptive?** Some criteria have no single correct keyboard interaction — they depend on the component. 2.1.1 (Keyboard) for a slider is different from 2.1.1 for a date picker. Do we write generic guidance or per-component guidance? Generic first, component-specific when we build the component view.

---

## 2. Component-Level View

### The story

Engineers and designers don't think in criteria — they think in components. When someone is building a modal, they don't go through all 56 criteria asking "does this apply?" They want to open a "Modal" page and see exactly what they need to know: the 8 criteria that apply, in the order that matters, with a working implementation.

This is the feature that would make MTRCD functionally irreplaceable. The W3C docs cannot offer this because the spec is organized by principle, not by UI pattern. It's also the clearest differentiator from axe-core (which runs against built code) and Figma plugins (which work at the design stage but can't give you implementation code).

### Build plan

Phase 1: Start with 10 high-priority components — the ones that come up in every codebase:

| Component | Criteria count (est.) |
|---|---|
| Modal / Dialog | 9 |
| Dropdown / Select | 8 |
| Form + Validation | 11 |
| Navigation | 6 |
| Data Table | 7 |
| Carousel / Slider | 5 |
| Tooltip | 4 |
| Accordion | 5 |
| Date Picker | 10 |
| Toast / Alert | 4 |

Each component page shows: the applicable criteria pulled from the existing data, a complete accessible implementation in code, and a working live demo (iframe or inline rendered HTML).

Phase 2: Add a "Which criteria apply to this component?" tag to every criterion card — so both views cross-reference.

### Open decisions

**Criteria that apply to almost everything.** 1.4.3 (contrast), 1.3.1 (info and relationships), and 4.1.2 (name, role, value) technically apply to every component. If we list them on every component page, they become noise. If we omit them, we're incomplete. Recommendation: list them in a "Universal criteria — applies everywhere" section at the top of each component, separate from the component-specific ones.

**Live demos require a decision on infrastructure.** Rendered HTML demos in iframes stay static. Interactive demos (open the modal, trigger the dropdown) require JS per-component. This is more build time but dramatically higher value. The tough question: do we build demos ourselves or embed CodePen/StackBlitz? Self-hosted means full control and offline capability. Embedded means faster build but external dependency and potential breakage.

**Maintenance.** Every time a framework releases a breaking change or a new ARIA pattern supersedes an old one, component pages need updating. This is the biggest long-term cost of this feature. The criteria cards are relatively stable (WCAG versions are years apart). Component best practices change faster.

---

## 3. Framework-Specific Code Snippets

### The story

The current code snippets are vanilla HTML and JavaScript. Most engineers working on production code aren't writing vanilla. A React developer looking at the modal focus trap example has to mentally translate it — and most will just go find a React-specific resource instead.

Adding React, Vue, and optionally Angular tabs to each snippet means the tool becomes directly copy-pasteable for the majority of the frontend ecosystem. This is a high-effort content build but low architectural complexity.

### Build plan

Add a framework tab switcher to each code block. The selected framework persists across cards (localStorage) so if you're a React developer, you never see vanilla again.

Start with React only — it covers roughly 45% of the frontend market and is what most startups and product companies use. Vue second. Angular third if there's demonstrated demand.

For each criterion, the snippet needs to be genuinely useful, not just syntactically translated. A React version of the focus trap shouldn't be `useRef` wrapped vanilla code — it should show the pattern in idiomatic React, using hooks correctly.

### Open decisions

**Versions.** React 18 vs 19 have meaningful differences (especially around concurrent mode and transitions). Vue 2 is still in wide use but EOL. Do we target current versions only, or maintain version-specific snippets? Recommendation: current major version only, with a visible version tag on each snippet and a note to check the migration guide for older versions.

**Who writes the framework snippets?** This is the hardest question on this feature. Framework-idiomatic code requires genuine expertise in each framework — not just syntax familiarity. A React accessibility snippet written by someone who doesn't use React daily will feel wrong to engineers who do, and that erodes trust in the whole tool. This almost certainly requires bringing in contributors or reviewing against established resources (React Aria, Radix, Headless UI).

**Don't duplicate libraries that already exist.** React Aria (Adobe) and Radix UI already provide accessible, unstyled primitives for many common components. Rather than competing, the better play for snippets is to show the *pattern* in vanilla/React and link out to these libraries for production use. That's more honest and more useful.

---

## 4. Built-In Contrast Checker

### The story

1.4.3 is the most-checked criterion in existence. Every designer, every code review, every accessibility audit touches contrast. Right now, checking contrast means leaving the tool, going to WebAIM or Coolors, entering the values, and coming back. We can eliminate that step.

The contrast checker is also the feature most likely to bring in users who aren't accessibility specialists — designers doing routine color work — and convert them into people who care about the rest of the criteria.

### Build plan

The core calculation is straightforward — relative luminance from WCAG 1.4.3's own formula, implemented in ~20 lines of JavaScript. The UI is two color inputs (hex + color picker), a large ratio display, and clear pass/fail indicators for small text (4.5:1), large text (3:1), and non-text UI (3:1 from 1.4.11).

**Where it lives** is the real build decision (see below).

Additional value-adds once the core is built:
- Show the nearest passing shade if the current combo fails (auto-suggest a darker/lighter version)
- Shareable URL with colors encoded in the query string (`/contrast?fg=1a1612&bg=f5f0e8`)
- Side-by-side text preview so designers can see how the combination actually reads

### Open decisions

**Where does it live?** Three options:

1. **Embedded in the 1.4.3 card only.** Lowest build cost, most contextually relevant. Downside: hidden unless you find the card. Designers probably won't find it.
2. **Standalone page at `/contrast`.** Easy to link to, shareable, can be its own SEO surface. Downside: splits the tool into multiple pages, increases nav complexity.
3. **Persistent floating tool accessible from anywhere.** High value, always available. Downside: most complex to build, could feel intrusive.

Recommendation: standalone page at `/contrast` first. Add a link from the 1.4.3 card. If usage data shows high traffic, promote it to the persistent nav.

**This is the clearest Pro feature candidate.** The basic checker is free. Pro gets: saved color palettes, bulk checking (paste your whole design system's color tokens), export to PDF or CSV, and APCA contrast (the next-generation algorithm that WCAG 3.0 will use). APCA is particularly valuable to offer because it's not in the official WCAG 2.2 docs at all.

---

## 5. Severity and Priority Ranking

### The story

Not all accessibility failures are equal. A missing skip link is a real barrier for keyboard users but it's fixable in 10 minutes and doesn't block all users. A form that can't be submitted without a mouse blocks an entire category of users from completing a transaction. Teams with accessibility debt need to know where to start, and the WCAG spec deliberately doesn't tell them — it treats all criteria as equal because it's a standard, not a triage guide.

MTRCD can be opinionated in a way the spec cannot.

### Build plan

Add a severity tier to each criterion: **Critical**, **High**, **Medium**. Criteria are assigned based on:

- **User impact:** Does failure block task completion entirely, or just make it harder?
- **Breadth:** Does it affect many users (motor, visual, cognitive) or a narrow subset?
- **Legal exposure:** Is this criterion commonly cited in ADA/WCAG litigation?
- **Detection:** Is it caught by automated tools (lower urgency) or only catchable manually?

Add a new filter: "Show Critical only" — for teams doing a first-pass triage. Add an optional sort: "Sort by severity."

### Open decisions

**This is the most editorially risky feature.** Severity is context-dependent. 1.2.2 (Captions for prerecorded video) is Critical for a video-first platform and Medium for a mostly-text blog. Any global ranking will be wrong for someone. Two paths:

- **Global ranking with a disclaimer.** Simpler, ships faster, openly acknowledges the limitation. Probably fine for 80% of use cases.
- **Context-aware ranking.** User sets their site type (video-heavy, form-heavy, e-commerce, content) and severity adjusts. Much more valuable, significantly more complex to build and maintain.

Recommendation: ship a well-reasoned global ranking first. Document the methodology transparently — "here's how we scored these and why." Invite disagreement. Use feedback to build toward context-aware ranking.

**Don't call anything "Low."** If something is in WCAG it matters to someone. "Medium" is the floor. Calling a criterion low-severity implies you can ignore it, which is the wrong message for an accessibility tool to send.

---

## 6. The "Make the Case" Layer

### The story

Designers and engineers often know what needs to be fixed. The blocker is getting prioritization from a PM or stakeholder who sees accessibility as a nice-to-have. MTRCD can give designers and engineers the language they need to have that conversation.

Each criterion should have a mode that reframes the technical requirement into: who is affected and how, what the business risk is, and what the simplest fix looks like — without assuming the reader knows what a focus indicator is.

### Build plan

A collapsible "Make the case" section on each card with three parts:
1. **Who it affects** — plain description of the users impacted, with rough prevalence numbers where known (e.g., "~2.2 billion people globally have some form of vision impairment")
2. **The risk** — framed around usability, brand, and legal exposure without making specific legal guarantees
3. **The fix, in one sentence** — for when a stakeholder asks "well how hard is it?"

This is content-only work once the card structure supports a third body section.

### Open decisions

**Legal language is a minefield.** We cannot say "you will be sued" or "this violates the ADA." We also can't be so vague that we're useless. The right tone is: "This criterion is frequently cited in accessibility litigation under the ADA and similar legislation. Teams that address it reduce legal exposure and improve usability for [X million] users." Specific enough to be credible, hedged enough to be accurate.

**Tone calibration.** The "make the case" content is written for a different reader than the technical content — someone who makes prioritization decisions, not someone who writes code. That means two very different voices living on the same card. This needs careful copy editing so it doesn't feel like the card is talking to two different people at once.

---

## Cross-Cutting Decisions

### Free vs. Pro

The current Pro callout ("Go Pro — $9") is a placeholder. The roadmap forces a real decision about what goes behind a paywall. Some principles:

- **The reference is free, forever.** All 56 criteria, all explanations, vanilla code snippets. This is what builds trust and traffic.
- **Workflow tools are Pro.** The contrast checker with saved palettes, exported checklists, bulk color checking, APCA — these are for teams that have made accessibility a regular practice, not first-timers.
- **Framework snippets are probably free.** Putting React code behind a paywall would feel extractive on what is essentially a reference tool. The network effect of engineers sharing MTRCD links is worth more than the paywall.
- **The "Make the case" layer is free.** It helps individuals advocate internally — gating it would be contrary to the mission.

The strongest Pro features are things teams use repeatedly on a subscription basis: saved checklists, project tracking, team sharing, and eventually the component audit tool.

### Static vs. backend

Right now the entire tool is static HTML/JS. That means zero infrastructure cost and instant load times. Most of the roadmap can stay static. But some features create pressure toward a backend:

- **Saved color palettes** (contrast checker) — needs user accounts or at minimum localStorage-only
- **Project checklists** — per-project state has to live somewhere
- **Usage analytics for severity ranking** — understanding what criteria users filter on most would improve the priority model

Recommendation: stay static as long as possible. Use localStorage for state that doesn't need to persist across devices. Add a lightweight backend (Vercel Functions + a small database) only when a feature genuinely requires it. Don't build user accounts until there's a clear retention reason.

### Content maintenance strategy

More features means more content to keep current. The component patterns in Feature 2, the framework snippets in Feature 3, and the severity rankings in Feature 5 will all need revisiting when WCAG 3.0 ships, when frameworks release breaking changes, or when new litigation sets new precedent.

This needs a decision before building: is MTRCD maintained by one person, or does it build toward a contributor model? If it stays a solo project, scope needs to match the maintenance capacity. If it opens to contributors, the data format (currently a JS array in a script tag) needs to become something more structured — a JSON file or a lightweight CMS — so contributors can submit updates without touching application code.

---

## Sequencing

**Phase 1 — High value, content-only (no architecture changes)**
1. Testing guidance per criterion — adds the most immediate daily value for engineers
2. "Make the case" layer — pure content, enables internal advocacy
3. Severity ranking — editorial work, single new data field per criterion

**Phase 2 — New UI, still static**
4. Framework snippets (React first) — tab switcher on code blocks
5. Built-in contrast checker — standalone `/contrast` page

**Phase 3 — New surfaces, higher build cost**
6. Component-level view — new navigation paradigm, live demos, cross-referencing

Phase 3 is where the tool becomes something genuinely difficult to replicate. Phases 1 and 2 can ship incrementally and start building the audience that makes Phase 3 worth building.
