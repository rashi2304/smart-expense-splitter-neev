# 💸 SplitEase — Smart Expense Splitter

A full-stack web application to track shared expenses, split costs fairly, and settle up with friends, roommates, or teammates.

**Built for: NeevAI SuperCloud Internship Assignment**

---

## 🚀 Live Demo
> Deploy URL goes here after deployment

---

## ✨ Features

| Feature | Status |
|---|---|
| Create groups with multiple members | ✅ |
| Add shared expenses | ✅ |
| Split equally OR custom amounts | ✅ |
| Auto-calculate who owes whom | ✅ |
| Settle up and record payments | ✅ |
| Real-time updates via Socket.io | ✅ |
| AI-powered expense categorization | ✅ |
| Spending analytics by category & member | ✅ |
| Multi-currency support | ✅ |

---

## 🏗️ System Architecture

```
┌──────────────────────────────────────────────────────────┐
│                        Frontend                           │
│   React + Tailwind CSS (Vite)  ·  Port 5173              │
│   Pages: HomePage, GroupPage                             │
│   Components: Modals, Cards, Panels                      │
│   Real-time: Socket.io-client                            │
└────────────────────────┬─────────────────────────────────┘
                         │ HTTP + WebSocket
┌────────────────────────▼─────────────────────────────────┐
│                        Backend                            │
│   Node.js + Express  ·  Port 5000                        │
│   Routes: /groups  /expenses  /settlements               │
│   Real-time: Socket.io server                            │
└────────────────────────┬─────────────────────────────────┘
                         │ Mongoose ODM
┌────────────────────────▼─────────────────────────────────┐
│                       MongoDB                             │
│   Collections: groups · expenses · settlements           │
└──────────────────────────────────────────────────────────┘
```

---

## 🗄️ Database Schema

### Group
```js
{
  name: String,          // "Goa Trip"
  description: String,
  currency: String,      // "INR"
  members: [{ name, email }],
  createdAt, updatedAt
}
```

### Expense
```js
{
  groupId: ObjectId,
  description: String,   // "Lunch at Dominos"
  amount: Number,
  category: String,      // auto-detected: food/travel/...
  splitType: "equal"|"custom",
  paidBy: { memberId, memberName },
  splits: [{ memberId, memberName, amount }],
  date: Date,
  notes: String
}
```

### Settlement
```js
{
  groupId: ObjectId,
  fromMemberId: ObjectId,  // who paid
  toMemberId: ObjectId,    // who received
  amount: Number,
  settledAt: Date
}
```

---

## 🔌 API Endpoints

### Groups
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET    | `/api/groups`                        | List all groups |
| POST   | `/api/groups`                        | Create group |
| GET    | `/api/groups/:id`                    | Get group |
| PUT    | `/api/groups/:id`                    | Update group |
| DELETE | `/api/groups/:id`                    | Delete group + data |
| POST   | `/api/groups/:id/members`            | Add member |
| DELETE | `/api/groups/:id/members/:memberId`  | Remove member |
| GET    | `/api/groups/:id/balances`           | **Get balances & who owes whom** |

### Expenses
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET    | `/api/expenses/group/:groupId`           | List group expenses |
| POST   | `/api/expenses`                          | Add expense |
| DELETE | `/api/expenses/:id`                      | Delete expense |
| POST   | `/api/expenses/categorize`               | AI categorize |
| GET    | `/api/expenses/group/:groupId/analytics` | Spending analytics |

### Settlements
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET    | `/api/settlements/group/:groupId` | List settlements |
| POST   | `/api/settlements`                | Record settlement |
| DELETE | `/api/settlements/:id`            | Undo settlement |

---

## 🧮 Balance Algorithm

The app uses a **greedy debt-minimization algorithm**:

1. Calculate net balance per person: `+` means others owe them, `-` means they owe others
2. Subtract any recorded settlements
3. Separate into **creditors** (owed money) and **debtors** (owe money)
4. Sort both lists by amount descending
5. Greedily match largest debtor → largest creditor
6. This minimizes the total number of transactions needed

---

## 🤖 AI Feature: Smart Expense Categorization

When a user types an expense description, the backend automatically categorizes it using keyword matching. The frontend calls `/api/expenses/categorize` with a 600ms debounce, showing the detected category in real time.

Categories: `food · travel · accommodation · entertainment · utilities · shopping · health · other`

Examples:
- "Swiggy order" → 🍽️ Food
- "Uber to airport" → 🚗 Travel
- "OYO Rooms" → 🏠 Accommodation
- "BookMyShow tickets" → 🎬 Entertainment

---

## ⚡ Real-Time Updates (Socket.io)

When any user in a group adds an expense or settles up, all other members see the update instantly without refreshing.

Events: `expense-added` · `expense-deleted` · `settlement-added` · `group-updated`

---

## 🖥️ Local Setup

### Prerequisites
- Node.js 18+
- MongoDB running locally OR MongoDB Atlas URI

### Step 1: Clone & install

```bash
git clone https://github.com/YOUR_USERNAME/smart-expense-splitter.git
cd smart-expense-splitter
```

### Step 2: Setup Backend

```bash
cd backend
cp .env.example .env
# Edit .env — set your MONGO_URI
npm install
npm run dev
# → Running on http://localhost:5000
```

### Step 3: Setup Frontend

```bash
cd ../frontend
npm install
npm run dev
# → Running on http://localhost:5173
```

### Step 4: Open in browser

Visit: **http://localhost:5173**

---

## 🚀 Deployment on Vercel

### Part 1: Deploy Backend

> **Recommended**: Deploy backend on **Railway** (free, supports WebSockets)

1. Go to [railway.app](https://railway.app) → New Project → Deploy from GitHub
2. Select the `backend` folder
3. Set environment variables:
   ```
   MONGO_URI = mongodb+srv://...    ← MongoDB Atlas free tier
   CLIENT_URL = https://your-frontend.vercel.app
   PORT = 5000
   ```
4. Railway gives you a URL like: `https://your-backend.railway.app`

### Part 2: Deploy Frontend on Vercel

1. Create `frontend/.env.production`:
   ```
   VITE_API_URL=https://your-backend.railway.app/api
   VITE_SOCKET_URL=https://your-backend.railway.app
   ```
2. Push to GitHub
3. Go to [vercel.com](https://vercel.com) → New Project → Import repo
4. Set **Root Directory** to `frontend`
5. Deploy! ✅

### Alternative: Vercel for Backend too

Add `vercel.json` in `/backend`:
```json
{
  "version": 2,
  "builds": [{ "src": "server.js", "use": "@vercel/node" }],
  "routes": [{ "src": "/(.*)", "dest": "server.js" }]
}
```
> Note: Vercel serverless doesn't support persistent WebSocket connections. For full real-time, use Railway/Render for the backend.

---

## 📁 Project Structure

```
smart-expense-splitter/
├── backend/
│   ├── server.js                    # Entry point + Socket.io
│   ├── models/
│   │   ├── Group.js                 # Group + members schema
│   │   ├── Expense.js               # Expense + splits schema
│   │   └── Settlement.js            # Payment records schema
│   ├── routes/
│   │   ├── groups.js                # Group CRUD + balance API
│   │   ├── expenses.js              # Expense CRUD + analytics
│   │   └── settlements.js           # Settlement CRUD
│   ├── utils/
│   │   └── balanceCalculator.js     # Core algorithm + AI categorizer
│   ├── .env.example
│   └── package.json
│
└── frontend/
    ├── src/
    │   ├── main.jsx                 # React entry point
    │   ├── App.jsx                  # Router setup
    │   ├── index.css                # Tailwind + global styles
    │   ├── pages/
    │   │   ├── HomePage.jsx         # Groups list
    │   │   └── GroupPage.jsx        # Group detail (expenses/balances/members/analytics)
    │   ├── components/
    │   │   ├── Navbar.jsx
    │   │   ├── Modal.jsx            # Reusable modal wrapper
    │   │   ├── Avatar.jsx           # Member avatar
    │   │   ├── ExpenseCard.jsx      # Single expense display
    │   │   ├── BalancesPanel.jsx    # Who owes whom
    │   │   ├── AnalyticsPanel.jsx   # Spending charts
    │   │   ├── AddExpenseModal.jsx  # Add expense form
    │   │   ├── AddMemberModal.jsx   # Add member form
    │   │   ├── CreateGroupModal.jsx # Create group form
    │   │   └── SettleUpModal.jsx    # Record settlements
    │   ├── hooks/
    │   │   └── useSocket.js         # Real-time socket hook
    │   └── utils/
    │       ├── api.js               # Axios API service layer
    │       └── helpers.js           # Formatters, constants
    ├── index.html
    ├── vite.config.js
    ├── tailwind.config.js
    └── package.json
```

---

## 👨‍💻 Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, Tailwind CSS, React Router v6 |
| Backend | Node.js, Express.js |
| Database | MongoDB, Mongoose ODM |
| Real-time | Socket.io |
| HTTP Client | Axios |
| Build Tool | Vite |
| Fonts | Plus Jakarta Sans, Outfit, Fira Code |
