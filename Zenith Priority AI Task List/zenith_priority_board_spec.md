# Zenith Priority Board — Claude Code Build Spec

## Overview

Build a standalone single-page web app called **Zenith Priority Board** — a personal AI-powered task management tool for one user. It replaces scattered to-do lists (notes apps, sticky notes, calendar, Trello, reminders) with a single visual board. The user is a visual thinker who values simplicity, speed, and clarity above all else. No login, no accounts, no teams — just one person's board.

The app must work offline for viewing and editing tasks. It calls the Anthropic API only when the user presses an AI action button.

---

## Tech Stack

- **Frontend:** Single HTML file with vanilla JavaScript, or a simple React app (single component file is fine)
- **Storage:** `localStorage` for all task and archive data — tasks must persist between sessions
- **AI:** Anthropic API (`claude-sonnet-4-6`) called on-demand via fetch from the browser. The user will supply their own API key, stored in `localStorage`.
- **No backend required.** No build pipeline required if using vanilla JS. If React, use a simple Vite setup.
- **Hosting:** Runs locally, opened as a file in the browser or via `npx serve`

---

## Layout

### Top toolbar (fixed, does not scroll)
Left to right:
- App title: `🗂 Priority Board`
- Zoom controls: `−` / `100%` / `+` buttons
- Button: `🤖 AI recommendations`
- Button: `🔍 Find duplicates`
- Button: `✦ Re-prioritize`
- Button: `+ Add task` (primary/blue style)

### Category filter bar (fixed, below toolbar)
- Label: `Filter:`
- Pills: `All` (neutral), then one pill per category (color-matched to category colors)
- Clicking a pill hides all cards not matching that category. `All` resets.
- Active pill has a dark border to indicate selection.

### Board (scrollable)
- Three columns side by side, each `minmax(400px, 1fr)`, minimum board width `1230px`
- Board scrolls horizontally and vertically inside a scroll container
- Board supports zoom via `transform: scale()` on the board div, with `transform-origin: top left`
- Ctrl/Cmd + scroll wheel = zoom in/out (5% per tick, range 50%–150%)
- Toolbar zoom buttons = 10% per click

### Archive (below board, inside scroll container)
- Hidden by default, shown only when at least one task has been archived
- Collapsed toggle: `🗂 Archived tasks (N)` — click to expand/collapse
- Archived cards displayed in a matching 3-column grid
- Each archived card shows: emoji, title, date archived, and a `Restore` button

### Add task modal (overlay)
Triggered by `+ Add task` button. Fields:
- Task title (text input)
- Column selector (Professional / Personal / Reminders)
- Category selector (see categories below)
- Emoji (text input, single character, defaults to 📌)
- Notes/detail (textarea, optional)
- Cancel and Save buttons

---

## Three Columns

| Column | Background tint | Header label |
|--------|----------------|--------------|
| Professional | `rgba(23, 90, 165, 0.05)` | 💼 Professional |
| Personal | `rgba(136, 135, 128, 0.07)` | 🌿 Personal |
| Reminders | `rgba(0, 0, 0, 0.06)` | 🔔 Reminders |

Each column contains, top to bottom:
1. **Quick-capture input** — dashed border, placeholder e.g. `⚡ Quick add to Professional — press Enter`. Pressing Enter adds the task to that column with a 📌 emoji and a default category (Professional → Biz Dev, Personal → Personal, Reminders → Reminder).
2. **Column header** — title left, task count badge right
3. **Card list** — vertically stacked cards, drag-and-drop reorderable

---

## Card Design

### Visual style
- Full card background color = category color (see palette below)
- Rounded corners: `border-radius: 10px`
- Drop shadow: `box-shadow: 0 2px 7px rgba(0,0,0,0.14), 0 0.5px 2px rgba(0,0,0,0.08)`
- Elevated shadow on hover
- No border

### Card anatomy (top to bottom)

**Top row:**
- Left: emoji (static, 17px, not clickable — no emoji picker needed)
- Center: task title (13px, font-weight 500, line-height 1.45, wraps naturally, full available width)
- Right: ✓ done button (small circular button, 22×22px, top-right corner)

**Expandable notes section** (hidden by default):
- Revealed by the chevron button in the footer
- Light semi-transparent background within the card
- Click to edit inline (contenteditable)

**Footer row** (left to right):
- Priority number (e.g. `1.`) — 10px, muted opacity
- Category label button — 9px uppercase, semi-transparent background, clicking opens category-change popup
- Chevron button — toggles notes section open/closed (icon flips between down/up)
- Trash button — deletes card after confirmation

### Card interactions
- **Title:** Click to edit inline (`contenteditable`). Press Enter or blur to save.
- **Notes:** Click the expanded notes area to edit inline. Blur to save.
- **Category:** Click category label in footer → small popup appears (above the card) listing all categories with colored dots. Click one to reassign.
- **Done:** Click ✓ button (top right) → card moves to archive with today's date. No confirmation needed.
- **Delete:** Click trash icon → confirm dialog → permanent deletion.
- **Drag to reorder:** Cards are draggable within and between columns. Dropping on a card inserts above it. Dropping on empty space in a column appends to bottom. Priority numbers update automatically after any reorder.

### Card filter behavior
- When a category filter is active, cards not matching the category are hidden (`display: none`) but not deleted. Clearing the filter restores them.

---

## Category Color Palette

| Category key | Display name | Card background | Text color | Dot color |
|---|---|---|---|---|
| `sales` | Sales | `#F5C4B3` | `#712B13` | `#D85A30` |
| `marketing` | Marketing | `#B5D4F4` | `#0C447C` | `#378ADD` |
| `bizdev` | Biz Dev | `#C0DD97` | `#27500A` | `#639922` |
| `editing` | Editing | `#CECBF6` | `#3C3489` | `#7F77DD` |
| `health` | Health | `#9FE1CB` | `#085041` | `#1D9E75` |
| `personal` | Personal | `#E2E1DC` | `#3A3A38` | `#888780` |
| `reminder` | Reminder | `#2C2C2A` | `#D3D1C7` | `#888780` |

---

## Data Model

All data stored in `localStorage` under two keys:

### `zpb_tasks`
Array of task objects:
```json
{
  "id": 1,
  "title": "Explore recurring retainers for new clients",
  "emoji": "🔁",
  "col": "professional",
  "cat": "sales",
  "detail": "Optional notes text here.",
  "priority": 1
}
```
- `col`: `"professional"` | `"personal"` | `"reminders"`
- `cat`: `"sales"` | `"marketing"` | `"bizdev"` | `"editing"` | `"health"` | `"personal"` | `"reminder"`
- `priority`: integer, 1-indexed per column. Re-calculated after every drag-and-drop.

### `zpb_archive`
Array of archived task objects — same shape as above, plus:
```json
{
  "archivedAt": "6/16/2025"
}
```

---

## AI Integration

### Setup
- On first launch, prompt user to enter their Anthropic API key via a simple settings modal
- Store key in `localStorage` under `zpb_api_key`
- Show a small settings/gear icon in the toolbar to update the key later
- All AI calls use model: `claude-sonnet-4-6`, `max_tokens: 1500`

### AI button behaviors

**✦ Re-prioritize**
Sends all current tasks (title + column) and the user's goals to Claude. Asks Claude to return a re-ordered list by column. Parse the response and update `priority` values accordingly. Display Claude's response in a slide-in panel or modal — do not auto-reorder without user confirmation.

System prompt context to include:
> "You are a productivity assistant helping a video production business owner named Evans prioritize his tasks. His goals: Professionally — maximize profitability with minimal time, close retainer deals with new clients, grow his company Zenith Creative in the industrial sector. Personally — ensure the health and moral development of his two sons, keep his wife happy, build a retirement plan."

**🔍 Find duplicates**
Sends full task list (id + title) to Claude. Asks it to identify pairs or groups of tasks that represent the same objective and could be merged into one action. Display response in a modal.

**🤖 AI recommendations**
Sends full task list to Claude. Asks it to identify which tasks could be meaningfully completed or assisted by current AI tools — with specific tool names and estimated time savings per task. Display response in a modal.

---

## Zoom Behavior

- Board div gets `transform: scale(N)` and `transform-origin: top left`
- Scroll container has `overflow: auto` so scaled content is still scrollable
- Range: 50% to 150%, step 5% (scroll) or 10% (buttons)
- Zoom level displayed as percentage in toolbar

---

## Seeded Initial Data

Pre-load the following tasks on first launch (only if `localStorage` is empty). These are the user's real starting tasks:

**Professional — Sales**
1. 🔁 Explore recurring retainers for new and existing clients
2. 📞 Call old clients — pitch new services
3. 📅 Build a free 15-minute video strategy call offer
4. 🏭 Cold call companies in the onboarding/training niche
5. 🎁 Create a free offer for hesitant prospects
6. 📦 Sell the DIY client content product/service

**Professional — Marketing**
7. 📄 Create a one-page offer for onboarding/training videos
8. ⭐ Ask clients for reviews or testimonials
9. 📣 Create new Facebook ads for Zenith
10. 📧 Start an email newsletter
11. 📋 Set up a client email list
12. ✍️ Write articles for trade publications with ROI metrics
13. 📝 Create a cheap digital product / how-to sheet
14. ▶️ Start a YouTube channel
15. 📱 Create a TikTok channel about business/video tips
16. 🎞️ Start a separate short-form AI video channel
17. 🎙️ Try getting on podcasts for outreach

**Professional — Biz Dev**
18. 🤝 Reach out to agencies for overflow/freelance work
19. ⚙️ Improve your follow-up process with past clients
20. 🍽️ Invite old clients to lunch or an event
21. 🤖 Build a more automated lead-gen system
22. ✈️ Build a list of target aerospace companies
23. 💡 Start a mastermind with your creative freelancer network
24. 🗺️ Create a client-facing roadmap/strategy product
25. 🧠 Build an AI agent that writes daily blog posts
26. 🥊 Develop the 'Battle of the Podcasters' concept
27. 👤 Consider hiring a commission-based salesperson
28. 💻 Hire and onboard a virtual assistant

**Professional — Editing**
29. 🎬 Update the Zenith Creative demo reel and share socially
30. 🚀 Do spec recruiting video work for aerospace companies
31. 🎥 Send a new video concept to editor for a social post

**Personal — Personal**
32. 💛 Help wife find a new job
33. 🏡 Explore a house-sitting business for wife to run
34. 🦖 Watch Shin Godzilla
35. 🪑 Find a more comfortable office chair

**Personal — Health**
36. 🎣 Take the boys fishing in Houston

**Personal — Editing**
37. 🎵 Make the pro bono music video with Mark Bebawi

**Reminders — Reminder**
38. ⏱️ Remember: time saved = financial ROI when pitching
39. 💡 Remind Josh: need more chandelier footage

---

## Design Notes for Claude Code

- The app is for **one person on a wide desktop screen**. Optimize for 1440px+ width. No need for mobile responsiveness.
- Keep the UI clean and flat. No gradients, no heavy shadows, no decorative elements.
- Font: system default sans-serif is fine.
- All text on colored card backgrounds must use the text color from the category palette — never generic black.
- The reminder column uses a near-black background (`#2C2C2A`) with light text (`#D3D1C7`). Make sure all interactive elements within reminder cards are visible against the dark background.
- Drag-and-drop should feel smooth. Use the HTML5 Drag and Drop API. Visual indicator: a 2px blue outline on the card being dragged over, and a subtle blue tint on the column list when dragging over empty space.
- The ✓ done button should feel satisfying — consider a brief fade/scale animation when a card is marked done before it disappears.
- Quick-capture inputs use a dashed border style to visually distinguish them from task cards.
- Category change popup should appear **above** the card (not below, to avoid clipping at the bottom of long lists).
- All data writes to localStorage should happen immediately on any change (title edit blur, drag drop, done, delete, add).

---

## Future Features (do not build now, but design the data model to support them)

- Due dates on cards
- Collapsed card view (title-only toggle)
- Filter by column + category simultaneously
- Export/print view
