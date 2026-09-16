# MockFlow

A lightweight, browser-based mock interview tool for technical interview preparation. Upload your own question banks as Markdown files, practice answering questions, self-grade your responses, and let the adaptive algorithm prioritize the questions you struggle with most.

---

## Features

- **Upload your own question bank** — Paste or upload any Markdown file with questions and answers.
- **Adaptive question selection** — Questions you get wrong appear more frequently; mastered questions are asked less often.
- **Self-grading** — After revealing the model answer, mark your response as correct or incorrect.
- **Session persistence** — Your session and all history are saved in the browser (localStorage) so you can resume where you left off.
- **Running score** — Live correct / incorrect count and accuracy percentage during each session.
- **End-of-session summary** — Final score, total questions answered, and accuracy breakdown.
- **Multi-subject support** — Load multiple subjects and switch between them.
- **Demo subjects included** — Five ready-to-use question banks (JavaScript, Node.js, React, OOP, DBMS).
- **No backend, no account, no install** — Pure client-side app. Open `index.html` in any browser and start practising.

---

## How It Works

### 1. Upload a Question Bank

Your Markdown file must follow this format:

```markdown
### Q1. What is a closure?

**A.**  
A closure is a function that retains access to its outer lexical scope even after the outer function has returned.

### Q2. What is the event loop?

**A.**  
The event loop processes the call stack, microtask queue, and macrotask queue in order...
```

**Supported formats:**
- `### Q<N>. Question text` with `**A.**` answer block (primary format)
- Numbered lists like `5. Question text`
- `## Heading` and `### Heading` fallback (headings treated as questions)

Question numbers from the source file are stripped from the display. You only see the session position (Question 1, Question 2, …).

---

### 2. Practice Session Flow

```
Start session
     ↓
Question displayed
     ↓
Type your answer (or dictate via your keyboard's built-in voice input on mobile)
     ↓
Reveal model answer
     ↓
Self-grade: ✅ Correct  or  ❌ Wrong
     ↓
Next question (adaptive selection)
```

---

### 3. Adaptive Algorithm

Each question has a **weight** that determines how likely it is to be selected next:

| Situation | Weight |
|-----------|--------|
| Never seen before | 3.0 (medium-high priority) |
| Answered correctly (streak 1) | 2.0 |
| Answered correctly (streak ≥ 2) | 1.0 |
| Mastered (streak ≥ 5) | 0.5 (appears rarely) |
| Answered wrong | 5.0 |
| Answered wrong 3+ times | 6.0 (highest priority) |
| Not seen for 3+ days | +50% boost on top of weight |

A **cooldown window** prevents the same question from appearing back-to-back in large pools.

---

### 4. Session Persistence

All data is stored in your browser's **localStorage** — nothing is sent to any server.

Stored data includes:
- All uploaded question banks (by subject slug)
- Per-question history: times correct, times wrong, current streak, last result, last seen date
- Active session state (current question, score, asked queue)

When you start a new session on a subject you've used before, the app offers to **resume** your previous session or start fresh.

---

## Getting Started

### Option A — Open directly in a browser

```bash
# Clone the repo
git clone https://github.com/Tarun-sde/MockFlow.git
cd MockFlow

# Open in browser
open index.html        # macOS
xdg-open index.html   # Linux
start index.html       # Windows
```

No build step. No dependencies to install. No internet connection required after the page loads (fonts and CDN libraries are fetched once).

### Option B — Serve locally (optional)

```bash
# Python 3
python3 -m http.server 8080

# Then open http://localhost:8080
```

---

## Demo Subjects

Five question banks are included in `demo_subjects/` for immediate use:

| File | Topics covered |
|------|---------------|
| `javascript.md` | Closures, prototypes, event loop, async/await, ES6+ |
| `node.md` | Event-driven architecture, streams, middleware, Express |
| `react.md` | Hooks, reconciliation, state, context, performance |
| `oop.md` | Encapsulation, inheritance, polymorphism, SOLID principles |
| `dbms.md` | Normalization, indexing, transactions, SQL, ACID |

To use them: click **Upload Questions**, paste the file contents, or drag-and-drop the `.md` file.

---

## Project Structure

```
MockFlow/
├── index.html          # Entire application (single file, self-contained)
├── demo_subjects/      # Sample question banks
│   ├── javascript.md
│   ├── node.md
│   ├── react.md
│   ├── oop.md
│   └── dbms.md
├── .gitignore
└── README.md
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| UI Framework | React 18 (via CDN, no build step) |
| Styling | Tailwind CSS (via CDN) |
| JSX transform | Babel Standalone (in-browser) |
| Storage | Browser `localStorage` |
| Fonts | Inter, Playfair Display, JetBrains Mono (Google Fonts) |
| Backend | None |

---

## Writing Your Own Question Bank

1. Create a `.md` file.
2. Use this structure:

```markdown
### Q1. Your question here?

**A.**  
Your answer here. Can include code blocks, bullet points, and multiple paragraphs.

### Q2. Another question?

**A.**  
Another answer.
```

3. In MockFlow, click **Upload Questions**, paste the Markdown text, give the subject a name, and click **Start Session**.

**Tips:**
- You can re-upload the same subject file after editing it. MockFlow will reconcile the questions by text and preserve your learning history for unchanged questions.
- Question identifiers like `Q40` or `5.` in the source file are for your reference only — they are never shown in the interview UI.

---

## Privacy

- All data lives in your browser's `localStorage`.
- No analytics, no tracking, no network requests after initial page load.
- To reset all data: open browser DevTools → Application → Local Storage → clear all keys.

---

## License

MIT — free to use, modify, and share.
