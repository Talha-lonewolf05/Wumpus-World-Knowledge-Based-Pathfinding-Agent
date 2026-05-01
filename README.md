# Wumpus World — Knowledge-Based Pathfinding Agent

## Project Structure
```
wumpus_agent/
├── app.py           ← Python Flask backend (AI engine)
├── index.html       ← Frontend (place in static/ or serve separately)
├── requirements.txt ← Python dependencies
├── vercel.json      ← Deployment config
└── README.md
```

## LOCAL SETUP (Step by Step)

### Step 1 — Install Python dependencies
```bash
pip install flask flask-cors
```

### Step 2 — Run the backend
```bash
python app.py
```
You'll see: `Running on http://127.0.0.1:5000`

### Step 3 — Open the frontend
Open `index.html` directly in your browser.
(The frontend calls `http://localhost:5000/api`)

### Step 4 — Play!
- Set grid size and pit count with sliders
- Click "NEW EPISODE" to start
- Use arrow keys or D-Pad to move manually
- Click "AUTO STEP" or "AUTO PLAY" for AI agent
- Click any cell to query the Knowledge Base

---

## DEPLOYMENT TO VERCEL

### Option A — Deploy Backend to Vercel (Serverless)
1. Install Vercel CLI:
   ```bash
   npm i -g vercel
   ```

2. Modify `app.py` — Vercel needs the app object exported:
   - The file is already compatible

3. Login and deploy:
   ```bash
   vercel login
   vercel --prod
   ```

4. Update `index.html` — change the API URL:
   ```javascript
   // Line near top of <script>:
   const API = 'https://YOUR-PROJECT.vercel.app/api';
   ```

5. Host frontend on Vercel too:
   - Create a `static/` folder, put `index.html` inside
   - Or deploy frontend separately on GitHub Pages

### Option B — Deploy Frontend on GitHub Pages (Free!)
1. Create a GitHub repo
2. Push `index.html` to the repo
3. Go to Settings → Pages → Deploy from main branch
4. Your frontend URL: `https://yourname.github.io/repo-name`
5. Backend stays on Vercel

---

## API ENDPOINTS

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/new_game` | POST | Start new episode |
| `/api/move` | POST | Move agent (direction: up/down/left/right) |
| `/api/auto_move` | POST | Agent picks best move using KB |
| `/api/ask_cell` | POST | Query KB about a specific cell (r, c) |
| `/api/state` | GET | Get current game state |

---

## HOW THE AI WORKS (For Your Report)

### Knowledge Base
- Maintains CNF clauses (Propositional Logic)
- TELL: adds new facts when percepts received
- ASK: queries via Resolution Refutation

### Resolution Refutation Algorithm
1. To prove `¬P_{r,c}` (cell is safe from pit):
   - Negate goal: assume `P_{r,c}` (pit exists)
   - Add to KB
   - Resolve clauses repeatedly
   - If empty clause derived → contradiction → cell IS safe ✓

2. Percept Logic:
   - No Breeze at (r,c) → ¬P for all neighbors
   - Breeze at (r,c) → P_n1 ∨ P_n2 ∨ ... (at least one neighbor has pit)
   - Same for Stench/Wumpus

### Move Strategy
1. Find unvisited safe cells (KB-proven) → move there
2. If none, try unvisited unknown cells (risky)
3. If stuck, backtrack to visited cell
