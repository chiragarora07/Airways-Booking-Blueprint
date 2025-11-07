# AirWays CLI Game — README

A simple Python command‑line game that lets you **collect money by playing Rock–Paper–Scissors** and **(pretend to) book airline tickets** between a few Indian cities. The program uses text menus, tracks a single in‑game currency balance, and loops back to the main menu after actions.

> ⚠️ This README also documents **bugs and improvement ideas** found while analyzing the code, so you can turn this into a cleaner project.

---

## Table of Contents
- [Overview](#overview)
- [Game Flow](#game-flow)
- [How to Run](#how-to-run)
- [Controls & Inputs](#controls--inputs)
- [Features](#features)
- [Destinations, Distances & Fares](#destinations-distances--fares)
- [Code Structure](#code-structure)
- [Known Issues / Bugs](#known-issues--bugs)
- [Suggestions for Improvement](#suggestions-for-improvement)
- [Sample Session](#sample-session)
- [License](#license)

---

## Overview

- **Language:** Python 3
- **Entry point:** `start(s)` (called at the bottom of the script)
- **Initial balance:** ₹100000
- **Mini‑game:** Rock–Paper–Scissors vs a computer opponent
  - **Win:** +₹1000
  - **Loss:** −₹250
  - **Tie:** ₹0
- **Ticket booking:** Shows fares between certain city pairs (₹10 per km), asks for confirmation, and (currently) **does not** deduct money on purchase — see bugs below.

---

## Game Flow

1. **Main menu (`start`)**
   - `1` → Play the Rock–Paper–Scissors mini‑game to earn money
   - `2` → Check current balance
   - `3` → Book AirWays ticket (choose route, see cost, confirm)
   - `4` → View supported destinations & distances
2. Actions often return to the **main menu** using `back()`.

```
Main Menu
├─ 1: Game (Rock–Paper–Scissors)
│   └─ Earn or lose money, then choose to play again
├─ 2: Check Balance
├─ 3: Book Ticket
│   └─ Show route → fare → confirm → (back to menu)
└─ 4: Destinations list
```

---

## How to Run

1. **Install Python 3.8+**
2. Save the script as, for example, `airways.py`
3. Run:
   ```bash
   python airways.py
   ```

> The script uses only the standard library (`random`). No extra packages needed.

---

## Controls & Inputs

- **Menu selection:** Enter numbers `1`–`4`.
- **Rock–Paper–Scissors:** Type one of `stone`, `paper`, `scissor` (lowercase).
- **Booking:** Enter city names in **lowercase** (e.g., `chennai`, `bangalore`).
- **Go back:** When prompted, type `b` to return to the main menu.

> The code compares exact lowercase strings; other capitalizations will not match.

---

## Features

- Simple **earn/spend** loop with a balance shown in rupees.
- **Rock–Paper–Scissors** mini‑game for earning money.
- **Ticket booking quotes** at **₹10/km** for predefined routes.
- **Destinations screen** that lists distances (from Chennai).

---

## Destinations, Distances & Fares

> The program uses **fixed route pairs** in `booking()` and a general **price = ₹10/km`**.

| Route (boarding → destination) | Distance | Fare |
|---|---:|---:|
| chennai → bangalore | 350 km | ₹3500 |
| chennai → raipur | 1200 km | ₹12000 |
| chennai → mumbai | 1300 km | ₹13000 |
| bangalore → raipur | 850 km | ₹8500 |
| bangalore → mumbai | 900 km | ₹9000 |
| raipur → mumbai | **100 km** | **₹1000** |
| mumbai → raipur | **100 km** | **₹1000** |

**Notes:**
- The distances for **raipur ↔ mumbai** are likely incorrect (100 km seems unrealistic).
- The `places()` screen lists: `Chennai (0 km), Bangalore (350 km), Raipur (1200 km), Mumbai (1300 km)` — effectively distances **from Chennai**.

---

## Code Structure

- **Globals**
  - `money` (initial balance ₹100000)
  - Some string variables (`s`, `a`, `b`, `p`) that are placeholders and not functionally needed.
- **Functions**
  - `start(s)`: Main menu and navigation
  - `game(x)`: Rock–Paper–Scissors mini‑game
  - `win(x)`: Adds ₹1000 to `money` (parameter unused)
  - `loose(x)`: Subtracts ₹250 from `money` (parameter unused)
  - `booking(b)`: Shows fares and “confirms” ticket purchase
  - `places(p)`: Prints distances from Chennai
  - `back(a)`: Offers return to main menu on `b`

Execution begins with:
```python
start(s)
```

---

## Known Issues / Bugs

1. **Random choice can be out of range**
   ```python
   bot_choice = random.randint(1,4)
   ```
   - Maps only `1..3` to `stone/paper/scissor`. If `4` occurs, `bot_choice` stays as an **int**, breaking comparisons.
   - ✅ **Fix:** use `random.randint(1,3)` or `random.choice(['stone','paper','scissor'])`.

2. **Name typos in tie messages**
   ```python
   print(..., botchoice)  # should be bot_choice
   ```
   - Two tie branches reference `botchoice` (missing underscore) → `NameError`.

3. **Case sensitivity / inconsistent input**
   - The code compares exact lowercase strings but doesn’t `.lower()` user input.
   - ✅ **Fix:** normalize with `.strip().lower()`.

4. **`win(x)` and `loose(x)` have unused parameters**
   - Both functions ignore the `x` argument; they mutate the global `money`.
   - ✅ **Fix:** remove the parameter; also consider returning the new balance.

5. **Spelling: `loose` vs `lose`**
   - Function is named `loose` but printed text says “Lost”; both are inconsistent with common spelling “lose/lose”.
   - ✅ **Fix:** rename to `lose()` everywhere.

6. **Ticket purchase does not change balance**
   - On successful confirmation (`confirm == 'y'`), the program prints success but **does not deduct fare** from `money`.
   - ✅ **Fix:** subtract the fare and then print the new balance.

7. **Flow control: stray `back(a)` calls**
   - Some branches call `back(a)` outside the `if/elif` block, which may execute regardless of condition or lead to confusing navigation.
   - ✅ **Fix:** structure `if/elif/else` blocks so `back(a)` is called exactly once per path.

8. **Magic strings and duplicated route logic**
   - Route checks are manually repeated.
   - ✅ **Fix:** store routes in a dict and compute fare = `distance * 10`.

9. **Global state and recursion depth**
   - Repeatedly calling `start()` via recursion can grow the call stack.
   - ✅ **Fix:** use a `while True:` loop for the main menu instead of recursive calls.

10. **Validation & error handling**
    - Invalid menu options or city names aren’t handled gracefully.
    - ✅ **Fix:** add `try/except` and input validation + helpful prompts.

---

## Suggestions for Improvement

- **Refactor input handling:**
  ```python
  choice = input("Enter your choice: ").strip().lower()
  ```
- **Use data structures for routes:**
  ```python
  ROUTES = {
      ("chennai","bangalore"): 350,
      ("chennai","raipur"): 1200,
      # ...
  }
  FARE_PER_KM = 10
  ```
- **Deduct fare on successful booking** and show remaining balance.
- **Looping main menu** via `while True` and `continue/break`.
- **Separate concerns** into modules: `game.py`, `booking.py`, `menu.py` for maintainability.
- **Unit tests** for win/lose logic and fare calculations.
- **Improve UX:** accept multiple input styles (e.g., `Stone`, `STONE`), show computer’s choice every round, and confirm balances after transactions.
- **Data correctness:** fix distances for `raipur ↔ mumbai` (100 km is not realistic).

---

## Sample Session

```
1 - Collect Money
2 - Check Balance
3 - Book AirWays Ticket
4 - Check Destinations.
Price - ₹10/km
Note-Make Sure your 'CapsLock' is off.
Enter Your Choice - 1

Choose One(Stone/Paper/Scissor): stone
Its a Tie! Computer also Chose stone
Do You Want To Play Again?(y/n) n

1 - Collect Money
2 - Check Balance
3 - Book AirWays Ticket
4 - Check Destinations.
Price - ₹10/km
Note-Make Sure your 'CapsLock' is off.
Enter Your Choice - 3

Enter Boarding Place: chennai
Enter Arrival Place: bangalore
Your Journey = 350kms
Total Charges = ₹3500
Are you sure you want to continue(y/n): y
You Successfully purchased a ticket!
Enter 'b' for returning to main menu: b
```

---

## License

Add your preferred license (e.g., MIT) to this section.
