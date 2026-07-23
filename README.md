# Facilitator Guide: Build a Coffee Shop App in 3 Hours

**Course:** TEWP 1020 · React.js
**Prerequisites:** Days 1–4 (How React Works, Project Setup + JSX in TypeScript, Talking to a Server + Environment Variables, State & Events)
**Format:** In-class build session with live checkpoints
**Deliverable:** "Campus Grounds" — a mobile-ordering app that fetches a menu, lets students customize drinks, and manages a cart with a running total.

---

## Session at a Glance

| Time | Segment | Skills Exercised |
|------|---------|------------------|
| 0:00–0:30 | Scoping workshop | Component thinking (Day 1–2) |
| 0:30–1:00 | Checkpoint 1: Static UI | Vite setup, JSX in TS, props (Day 2) |
| 1:00–1:45 | Checkpoint 2: Fetch the menu | fetch, env variables, loading/error (Day 3) |
| 1:45–2:45 | Checkpoint 3: Cart, state & events | useState, handlers, lifting state (Day 4) |
| 2:45–3:00 | Demos + v2 backlog debrief | Scoping, roadmap thinking |

**Instructor prep (before class):**
- [ ] Push the starter repo with branches: `main` (starter), `checkpoint-1`, `checkpoint-2`, `checkpoint-3` (finished)
- [ ] Verify `npm install && npm run dev` works on a clean machine
- [ ] Verify `npx json-server db.json --port 3001` serves the menu
- [ ] Have the finished app running on the projector as the "north star"
- [ ] Print or link this guide's Common Pitfalls section for TAs

---

## 0:00–0:30 — Scoping Workshop

**Goal:** Students experience cutting a real product down to a 3-hour MVP.

1. **(5 min) Brainstorm the dream app.** Ask: "What does the Starbucks app do?" Capture everything on the board: ordering, payments, loyalty stars, gift cards, store locator, notifications, account login...

2. **(10 min) Cut it down live.** For each feature ask two questions:
   - *Do we know the React concepts to build this?* (routing? auth? we haven't covered those)
   - *Can we build it in 3 hours?*

   Land on this MVP and write it where everyone can see it:
   > **MVP:** Display a menu fetched from a server → customize a drink (size, milk) → add to cart → see a running total → "place" an order (confirmation screen, no real backend write).

   Everything else goes on a **v2 backlog** list. Keep the list — you'll return to it at 2:45.

3. **(10 min) Component breakdown.** Sketch the component tree together:

   ```
   App                      ← owns cart state
   ├── Header               ← shop name, cart count
   ├── MenuList             ← receives menu data
   │   └── MenuItem         ← one card per drink
   ├── DrinkCustomizer      ← size/milk pickers (modal or panel)
   └── Cart                 ← line items, total, order button
   ```

   Ask: "Where should the cart's data live?" Guide them to *App*, because both MenuItem (adds) and Cart (displays/removes) need it. This plants the "lifting state up" idea before they hit it in code.

4. **(5 min) Define the data shape.** Write the core types on the board — students copy them into `src/types.ts` in the next segment.

---

## 0:30–1:00 — Checkpoint 1: Static UI with Hardcoded Data

**Branch:** `checkpoint-1` · **Skills:** Vite scaffold, JSX in TypeScript, props, `.map()` with keys

### Setup (10 min)

```bash
npm create vite@latest campus-grounds -- --template react-ts
cd campus-grounds
npm install
npm run dev
```

Delete the boilerplate in `App.tsx` and `App.css`. (If using your starter repo, students clone instead and CSS is pre-written — recommended, so the 3 hours go to React, not styling.)

### `src/types.ts`

```ts
export type Size = "Small" | "Medium" | "Large";
export type Milk = "Whole" | "Oat" | "Almond" | "None";

export interface MenuItem {
  id: number;
  name: string;
  description: string;
  basePrice: number;   // price of a Medium
  category: "Espresso" | "Brewed" | "Tea" | "Pastry";
  customizable: boolean; // pastries aren't
}

export interface CartItem {
  cartId: string;      // unique per line item (same drink, diff milk = 2 lines)
  item: MenuItem;
  size: Size;
  milk: Milk;
  quantity: number;
  linePrice: number;   // per-unit price after size adjustment
}
```

**Teaching beat:** `CartItem` wraps a `MenuItem` rather than copying its fields. Ask why (single source of truth; menu price changes propagate).

### `src/data.ts` (temporary — deleted in Checkpoint 2)

```ts
import type { MenuItem } from "./types";

export const MENU: MenuItem[] = [
  { id: 1, name: "Latte", description: "Espresso with steamed milk", basePrice: 4.75, category: "Espresso", customizable: true },
  { id: 2, name: "Cappuccino", description: "Equal parts espresso, steamed milk, foam", basePrice: 4.5, category: "Espresso", customizable: true },
  { id: 3, name: "Cold Brew", description: "Steeped 18 hours, served over ice", basePrice: 4.25, category: "Brewed", customizable: true },
  { id: 4, name: "Drip Coffee", description: "Daily single-origin roast", basePrice: 2.95, category: "Brewed", customizable: true },
  { id: 5, name: "Chai Latte", description: "Spiced black tea with steamed milk", basePrice: 4.5, category: "Tea", customizable: true },
  { id: 6, name: "Butter Croissant", description: "Baked every morning", basePrice: 3.5, category: "Pastry", customizable: false },
];
```

### `src/components/MenuItemCard.tsx`

```tsx
import type { MenuItem } from "../types";

interface Props {
  item: MenuItem;
}

export function MenuItemCard({ item }: Props) {
  return (
    <article className="menu-card">
      <div className="menu-card-info">
        <h3>{item.name}</h3>
        <p>{item.description}</p>
      </div>
      <div className="menu-card-action">
        <span className="price">${item.basePrice.toFixed(2)}</span>
        <button>Add</button>
      </div>
    </article>
  );
}
```

### `src/App.tsx`

```tsx
import { MENU } from "./data";
import { MenuItemCard } from "./components/MenuItemCard";
import "./App.css";

export default function App() {
  return (
    <div className="app">
      <header className="header">
        <h1>Campus Grounds</h1>
        <span className="cart-badge">Cart (0)</span>
      </header>

      <main>
        <h2>Menu</h2>
        {MENU.map((item) => (
          <MenuItemCard key={item.id} item={item} />
        ))}
      </main>
    </div>
  );
}
```

### ✅ Checkpoint 1 exit criteria
- App renders six menu cards from the array
- No `key` warnings in the console
- Students can explain what `{ item }: Props` is doing (destructuring a typed props object)

**If ahead of schedule:** have students group items by `category` with headings.

---

## 1:00–1:45 — Checkpoint 2: Fetch the Menu from a Server

**Branch:** `checkpoint-2` · **Skills:** fetch, async/await, environment variables, loading & error states

### The server (5 min)

Add `db.json` to the project root — its `menu` key holds the exact array from `data.ts`:

```json
{
  "menu": [
    { "id": 1, "name": "Latte", "description": "Espresso with steamed milk", "basePrice": 4.75, "category": "Espresso", "customizable": true },
    { "id": 2, "name": "Cappuccino", "description": "Equal parts espresso, steamed milk, foam", "basePrice": 4.5, "category": "Espresso", "customizable": true },
    { "id": 3, "name": "Cold Brew", "description": "Steeped 18 hours, served over ice", "basePrice": 4.25, "category": "Brewed", "customizable": true },
    { "id": 4, "name": "Drip Coffee", "description": "Daily single-origin roast", "basePrice": 2.95, "category": "Brewed", "customizable": true },
    { "id": 5, "name": "Chai Latte", "description": "Spiced black tea with steamed milk", "basePrice": 4.5, "category": "Tea", "customizable": true },
    { "id": 6, "name": "Butter Croissant", "description": "Baked every morning", "basePrice": 3.5, "category": "Pastry", "customizable": false }
  ]
}
```

Run it in a second terminal:

```bash
npx json-server db.json --port 3001
```

Menu is now at `http://localhost:3001/menu`. Have students open that URL in the browser first — seeing raw JSON demystifies the whole exercise.

### Environment variable (5 min) — reinforces Day 3

`.env` in the project root:

```
VITE_API_URL=http://localhost:3001
```

**Teaching beats:** Vite only exposes vars prefixed with `VITE_`; restart the dev server after editing `.env`; `.env` belongs in `.gitignore` in real projects (why? point at the API-keys conversation from Day 3).

### Update `App.tsx`

```tsx
import { useEffect, useState } from "react";
import type { MenuItem } from "./types";
import { MenuItemCard } from "./components/MenuItemCard";
import "./App.css";

export default function App() {
  const [menu, setMenu] = useState<MenuItem[]>([]);
  const [status, setStatus] = useState<"loading" | "ready" | "error">("loading");

  useEffect(() => {
    async function loadMenu() {
      try {
        const res = await fetch(`${import.meta.env.VITE_API_URL}/menu`);
        if (!res.ok) throw new Error(`Server responded ${res.status}`);
        const data: MenuItem[] = await res.json();
        setMenu(data);
        setStatus("ready");
      } catch {
        setStatus("error");
      }
    }
    loadMenu();
  }, []);

  return (
    <div className="app">
      <header className="header">
        <h1>Campus Grounds</h1>
        <span className="cart-badge">Cart (0)</span>
      </header>

      <main>
        <h2>Menu</h2>
        {status === "loading" && <p className="notice">Brewing the menu…</p>}
        {status === "error" && (
          <p className="notice error">Couldn't reach the café. Is json-server running?</p>
        )}
        {status === "ready" &&
          menu.map((item) => <MenuItemCard key={item.id} item={item} />)}
      </main>
    </div>
  );
}
```

Delete `src/data.ts`. The app should look identical to Checkpoint 1 — that's the point. Say it out loud: *"We swapped the data source and no component changed. That's why we separated components from data."*

**About `useEffect`:** if you haven't formally taught it yet, present the block as a given pattern — "this is how React apps run a fetch once when the component first appears" — and note it gets full treatment in an upcoming lesson. Students only need to modify the code *inside* it. The `status` union type (`"loading" | "ready" | "error"`) is a nice TypeScript payoff moment: the compiler stops typos like `staus === "raedy"`.

### Prove the states work
- **Loading:** in DevTools → Network, set throttling to "Slow 3G" and refresh.
- **Error:** stop json-server and refresh.

### ✅ Checkpoint 2 exit criteria
- Menu loads from `http://localhost:3001/menu` via the env variable (no hardcoded URL in components)
- Loading and error states both demonstrably render
- `src/data.ts` is deleted

---

## 1:45–2:45 — Checkpoint 3: Cart, Customization, State & Events

**Branch:** `checkpoint-3` (finished app) · **Skills:** useState, event handlers, controlled inputs, lifting state up, immutable updates, derived state

This is the longest and most valuable segment. Build it in four moves.

### Move 1 (10 min): Cart state lives in App

```tsx
const [cart, setCart] = useState<CartItem[]>([]);
const [customizing, setCustomizing] = useState<MenuItem | null>(null);
```

**Teaching beat:** `customizing` holds *which* drink is being customized, or `null` for "panel closed." Modeling UI state as data is a core React idea.

### Move 2 (15 min): Price logic + adding to cart

```tsx
import type { CartItem, MenuItem, Milk, Size } from "./types";

const SIZE_ADJUST: Record<Size, number> = { Small: -0.5, Medium: 0, Large: 0.75 };
const MILK_ADJUST: Record<Milk, number> = { Whole: 0, Oat: 0.75, Almond: 0.75, None: 0 };

export function unitPrice(item: MenuItem, size: Size, milk: Milk): number {
  return item.basePrice + SIZE_ADJUST[size] + MILK_ADJUST[milk];
}
```

Put this in `src/pricing.ts`. Pure functions with no React in them are easy to reason about (and, mention in passing, easy to unit test).

The add handler in `App.tsx` — **the immutability moment**:

```tsx
function addToCart(item: MenuItem, size: Size, milk: Milk) {
  const price = unitPrice(item, size, milk);
  const cartId = `${item.id}-${size}-${milk}`;

  setCart((prev) => {
    const existing = prev.find((line) => line.cartId === cartId);
    if (existing) {
      // same drink, same options → bump quantity (new array, new object!)
      return prev.map((line) =>
        line.cartId === cartId ? { ...line, quantity: line.quantity + 1 } : line
      );
    }
    return [...prev, { cartId, item, size, milk, quantity: 1, linePrice: price }];
  });
  setCustomizing(null);
}

function removeFromCart(cartId: string) {
  setCart((prev) => prev.filter((line) => line.cartId !== cartId));
}
```

**Anticipate the classic bug:** someone will try `cart.push(...)` and wonder why nothing re-renders. Let it happen to one volunteer on the projector if you can — it's the most memorable 2 minutes of the day. Rule of thumb on the board: **never mutate state; always hand React a new array/object** (`.map`, `.filter`, spread).

### Move 3 (20 min): The customizer — controlled inputs

`src/components/DrinkCustomizer.tsx`:

```tsx
import { useState } from "react";
import type { MenuItem, Milk, Size } from "../types";
import { unitPrice } from "../pricing";

interface Props {
  item: MenuItem;
  onConfirm: (item: MenuItem, size: Size, milk: Milk) => void;
  onCancel: () => void;
}

const SIZES: Size[] = ["Small", "Medium", "Large"];
const MILKS: Milk[] = ["Whole", "Oat", "Almond", "None"];

export function DrinkCustomizer({ item, onConfirm, onCancel }: Props) {
  const [size, setSize] = useState<Size>("Medium");
  const [milk, setMilk] = useState<Milk>("Whole");

  return (
    <div className="customizer">
      <h3>Customize your {item.name}</h3>

      <label>
        Size
        <select value={size} onChange={(e) => setSize(e.target.value as Size)}>
          {SIZES.map((s) => (
            <option key={s} value={s}>{s}</option>
          ))}
        </select>
      </label>

      <label>
        Milk
        <select value={milk} onChange={(e) => setMilk(e.target.value as Milk)}>
          {MILKS.map((m) => (
            <option key={m} value={m}>{m}</option>
          ))}
        </select>
      </label>

      <p className="live-price">${unitPrice(item, size, milk).toFixed(2)}</p>

      <div className="customizer-actions">
        <button className="ghost" onClick={onCancel}>Cancel</button>
        <button onClick={() => onConfirm(item, size, milk)}>Add to cart</button>
      </div>
    </div>
  );
}
```

**Teaching beats:**
- `value` + `onChange` = controlled input. React state is the source of truth, not the DOM.
- The live price is **derived state** — computed during render from `size`/`milk`. We do *not* store it in `useState`. Ask: "What bug would we invite if we stored the price in state too?" (It can drift out of sync.)
- The child doesn't own the cart. It reports upward via `onConfirm` — callbacks-as-props are how children talk to parents.

Wire it in `MenuItemCard` (add an `onAdd: (item: MenuItem) => void` prop; the Add button calls `onAdd(item)`, and App sets `customizing`). Pastries (`customizable: false`) skip the customizer and go straight to the cart at base price.

### Move 4 (15 min): The Cart component + derived total

`src/components/Cart.tsx`:

```tsx
import type { CartItem } from "../types";

interface Props {
  cart: CartItem[];
  onRemove: (cartId: string) => void;
  onOrder: () => void;
}

export function Cart({ cart, onRemove, onOrder }: Props) {
  const total = cart.reduce((sum, line) => sum + line.linePrice * line.quantity, 0);

  if (cart.length === 0) {
    return <aside className="cart"><p className="notice">Your cart is empty. Add a drink to get started.</p></aside>;
  }

  return (
    <aside className="cart">
      <h2>Your order</h2>
      <ul>
        {cart.map((line) => (
          <li key={line.cartId}>
            <div>
              <strong>{line.quantity}× {line.item.name}</strong>
              {line.item.customizable && <small> {line.size} · {line.milk} milk</small>}
            </div>
            <span>${(line.linePrice * line.quantity).toFixed(2)}</span>
            <button className="ghost" onClick={() => onRemove(line.cartId)}>Remove</button>
          </li>
        ))}
      </ul>
      <div className="cart-total">
        <span>Total</span>
        <strong>${total.toFixed(2)}</strong>
      </div>
      <button className="order-btn" onClick={onOrder}>Place order</button>
    </aside>
  );
}
```

**Teaching beat:** the total is `reduce` over the cart at render time — derived state again, same principle as the live price. The header badge count is derived the same way in App: `cart.reduce((n, line) => n + line.quantity, 0)`.

"Place order" sets an `ordered` boolean in App and swaps the main view for a confirmation screen ("Order #4231 — ready in 10 minutes ☕") with a "Start a new order" button that clears the cart and resets. No backend write — that's on the v2 backlog, and say so.

### ✅ Checkpoint 3 exit criteria
- Adding the same drink with the same options increments quantity; different options create a new line
- Removing works; total and header badge always agree with cart contents
- No state mutation anywhere (spot-check for `.push` / direct assignment)
- Confirmation screen appears and resets cleanly

---

## 2:45–3:00 — Demos and Debrief

1. **(8 min) Two or three volunteer demos.** Ask each: "What broke, and how did you figure it out?"
2. **(7 min) Return to the v2 backlog.** Map each parked feature to a concept, so the backlog becomes a course roadmap:
   - Checkout page & order history → **routing** (React Router)
   - Real order submission → **POST requests**, backend
   - Accounts & loyalty stars → **auth**, persistence
   - Order status updates → polling / websockets
   - Payments → Stripe (and why you *never* touch card numbers yourself)

   Closing line: *"You didn't build a small app today — you built the core of a real one and made an engineering decision about where to stop."*

---

## Common Pitfalls (TA Cheat Sheet)

| Symptom | Likely cause | Fix |
|---|---|---|
| `import.meta.env.VITE_API_URL` is `undefined` | Var added after server start, or missing `VITE_` prefix | Restart `npm run dev`; check prefix |
| Menu never loads, console shows CORS or refused | json-server not running / wrong port | Check the second terminal; confirm `:3001` |
| Cart doesn't update on add | Mutating with `.push` | Rebuild with spread: `[...prev, newLine]` |
| Quantity bump changes every line | `.map` returning `{ ...line, quantity: +1 }` without the `cartId` condition | Re-check the ternary condition |
| "Each child should have a unique key" | Missing/duplicate `key` in `.map` | `key={item.id}` on menu, `key={line.cartId}` in cart |
| Select won't change | `value` set but no `onChange` (read-only controlled input) | Add the `onChange` handler |
| TS error: `string` not assignable to `Size` | `e.target.value` is typed `string` | `as Size` cast (and mention runtime validation as the "real" fix) |
| Two cart lines for identical drinks | `cartId` built inconsistently | Use the `${id}-${size}-${milk}` template exactly |

## Stretch Goals (for fast finishers, in order)

1. Group the menu by `category` with section headings
2. Quantity +/– buttons on cart lines (instead of remove-only)
3. Search/filter input above the menu (another controlled input rep)
4. Add an espresso-shots option (extend `CartItem`, pricing, and customizer — a full vertical slice)
5. Persist the cart in memory across "new order" as an order history list
