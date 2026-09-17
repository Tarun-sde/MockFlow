# MockFlow

A lightweight, local-first mock interview and active-recall preparation application for developers. Upload your own technical question banks as Markdown files, practice formulating answers, self-evaluate against reference answers, and let an adaptive weighted algorithm automatically prioritize the questions you find most challenging.

---

## Highlights & Features

- **Upload Custom Question Banks** — Upload or paste any Markdown file formatted with questions and reference answers. Supports syntax highlighting, code blocks, and multi-paragraph answers.
- **Adaptive Spaced Selection** — Questions you miss or find difficult appear more frequently; mastered questions appear less often based on streak and last review interval.
- **Dark Mode & Light Mode** — Sophisticated layered dark theme (neutral charcoal grey, not pure black) and refined light theme, featuring seamless crossfade transitions, flash-prevention, and system theme detection.
- **Clean Developer-Tool UI** — Professional SVG score and grading indicators, clear action button hierarchy, and distraction-free typography.
- **Dual Flow Practice**:
  - **Standard Path**: Type your response in the answer cavity, submit to reveal the official reference answer side-by-side, and self-grade (*Correct* or *Incorrect*).
  - **Fast Path**: Click **"I Don't Know / Reveal"** to immediately record the question as missed, reveal the reference answer, and enable direct progression to the next question.
- **Session Lifecycle & Summaries**:
  - Live running score with tabular numbers and real-time accuracy percentage.
  - Dedicated **"End Session"** action to conclude the current session and review full statistics (total answered, correct marks, missed questions with side-by-side answer comparisons).
  - Starting or re-entering an interview automatically launches a fresh session.
  - Safe tab navigation: switching to *Subjects* or *Analytics* preserves ongoing active sessions.
- **Multi-Subject Ingestion & Validation** — Ingest multiple subject files into a unified pool with live Markdown preview, question count, and syntax diagnostic warnings.
- **Analytics & History** — Dedicated Analytics tab with detailed question bank breakdown, repetition weights, times seen, times wrong, and streaks.
- **100% Client-Side & Private** — Runs entirely in your browser via `localStorage`. No backend server, no login, no tracking, and no external requests after initial page load.

---

## How It Works

### 1. Practice Session Flow

```
           Start / Resume Session
                     ↓
               Question Card
                     ↓
       ┌───────────────────────────┐
       │   Type Technical Answer   │
       └─────────────┬─────────────┘
                     │
       ┌─────────────┴─────────────┐
       │                           │
 [Submit Answer]       [I Don't Know / Reveal]
       │                           │
  Answer Revealed             Answer Revealed
       │                      (Marked as Wrong)
  Self-Grade:                      │
[Correct / Incorrect]              │
       │                           │
       └─────────────┬─────────────┘
                     ↓
             [Next Question]
        (Adaptive Weighted Pick)
```

---

### 2. Adaptive Selection Algorithm

Every question is assigned a dynamic selection **weight** calculated from your review history:

| Review State | Weight | Priority Level |
| :--- | :--- | :--- |
| **New (Never seen before)** | `3.0x` | High Priority |
| **Answered Correctly (Streak: 1)** | `2.0x` | Medium Priority |
| **Answered Correctly (Streak: 2–4)** | `1.0x` | Normal Priority |
| **Mastered (Streak: ≥ 5)** | `0.5x` | Infrequent / Low Priority |
| **Missed / Wrong (Times Wrong: 1–2)** | `5.0x` | Urgent Priority |
| **Chronically Missed (Times Wrong: ≥ 3)** | `6.0x` | Maximum Priority |
| **Stale (Not reviewed for ≥ 3 days)** | `+50%` | Recency Boost applied to weight |

- **Cooldown Mechanism**: Recently asked questions are kept in a cooldown queue to prevent back-to-back repetitions in larger pools.

---

### 3. Markdown Question Bank Format

Create or edit your question banks using standard Markdown. MockFlow parses headings and answer blocks automatically:

```markdown
### Q1. What is a closure in JavaScript?

**A.**  
A closure is the combination of a function bundled together with references to its surrounding lexical state (the lexical environment). It gives an inner function access to an outer function's scope even after the outer function has closed.

### Q2. How does the event loop handle microtasks vs macrotasks?

**A.**  
After each macrotask completes, the JavaScript engine drains the entire microtask queue (promises, `process.nextTick`, `queueMicrotask`) before picking the next macrotask from the task queue.
```

**Supported Question Styles:**
- `### Q<N>. Question statement` with `**A.**` answer block (recommended).
- Numbered lists: `1. Question statement` followed by answer paragraphs.
- Section headings: `## Question` or `### Question` with succeeding body text as the answer.

> **Note:** Prefixes like `Q1.` or `1.` in the file are stripped from the practice card so you only see the session question counter (Question 1, Question 2, etc.).

---

## Included Demo Subjects

Five starter question banks are provided in the [`demo_subjects/`](demo_subjects/) directory:

| Subject File | Covered Topics |
| :--- | :--- |
| [`demo_subjects/javascript.md`](demo_subjects/javascript.md) | Closures, prototypes, scope chain, event loop, async/await, ES6+ |
| [`demo_subjects/node.md`](demo_subjects/node.md) | Libuv, streams, buffers, event emitter, cluster, middleware |
| [`demo_subjects/react.md`](demo_subjects/react.md) | Virtual DOM, reconciliation, useEffect lifecycle, custom hooks, memoization |
| [`demo_subjects/oop.md`](demo_subjects/oop.md) | Encapsulation, abstraction, inheritance, polymorphism, SOLID principles |
| [`demo_subjects/dbms.md`](demo_subjects/dbms.md) | ACID properties, indexing (B-Trees), normalization, transactions, locking |

To use any demo pack: open **Subjects**, upload or paste the file contents, and click **Save to subjects:all**.

---

## Quick Start

### Direct in Browser (Zero Installation)

```bash
# Clone repository
git clone https://github.com/Tarun-sde/MockFlow.git
cd MockFlow

# Open index.html directly
open index.html        # macOS
xdg-open index.html   # Linux
start index.html       # Windows
```

No npm install, no bundlers, and no compile step required.

### Local Server (Optional)

```bash
# Using Python 3 built-in HTTP server
python3 -m http.server 8000

# Visit http://localhost:8000 in your browser
```

---

## Project Structure

```
MockFlow/
├── index.html          # Complete self-contained single-page application
├── demo_subjects/      # Curated starter question banks
│   ├── javascript.md
│   ├── node.md
│   ├── react.md
│   ├── oop.md
│   └── dbms.md
├── .gitignore
└── README.md
```

---

## Architecture & Technology

| Area | Technology / Pattern |
| :--- | :--- |
| **Runtime & Core** | HTML5, React 18, Babel Standalone (in-browser execution) |
| **Styling & Theme** | Tailwind CSS + custom CSS design tokens with full light/dark support |
| **Transitions** | Universal `200ms ease` with radial gradient opacity crossfades |
| **Accessibility** | `@media (prefers-reduced-motion: reduce)` support and accessible focus rings |
| **Data Storage** | Local-first browser `localStorage` (`session:current`, `subjects:all`, `history:all`) |
| **Typography** | Inter (sans-serif), Playfair Display (headings), JetBrains Mono (code) |

---

## Privacy & Local Storage

- All question banks, notes, answers, and study statistics remain strictly inside your browser's `localStorage`.
- No analytics scripts, no tracking, and no external network calls after fonts/scripts load.
- To reset or backup your data, access browser DevTools (`F12` → **Application** → **Local Storage**).

---

## License

Released under the [MIT License](LICENSE). Free for personal practice, education, and modification.
