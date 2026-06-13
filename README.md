# 🎮 Game Glitch Investigator: The Impossible Guesser

## 🚨 The Situation

You asked an AI to build a simple "Number Guessing Game" using Streamlit.
It wrote the code, ran away, and now the game is unplayable. 

- You can't win.
- The hints lie to you.
- The secret number seems to have commitment issues.

## 🛠️ Setup

1. Install dependencies: `pip install -r requirements.txt`
2. Run the broken app: `python -m streamlit run app.py`

## 🕵️‍♂️ Your Mission

1. **Play the game.** Open the "Developer Debug Info" tab in the app to see the secret number. Try to win.
2. **Find the State Bug.** Why does the secret number change every time you click "Submit"? Ask ChatGPT: *"How do I keep a variable from resetting in Streamlit when I click a button?"*
3. **Fix the Logic.** The hints ("Higher/Lower") are wrong. Fix them.
4. **Refactor & Test.** - Move the logic into `logic_utils.py`.
   - Run `pytest` in your terminal.
   - Keep fixing until all tests pass!

## 📝 Document Your Experience

**Game Purpose:**
Glitch Guesser is a number-guessing game where the player picks a difficulty (Easy, Normal, or Hard), then tries to guess a randomly chosen secret number within a limited number of attempts. After each guess the game gives a directional hint ("Go HIGHER" or "Go LOWER") and updates a running score. The code was intentionally shipped with several logic bugs, and the goal of this project is to find, explain, and fix them.

**Bugs Found:**

| Bug | Location | Description |
|-----|----------|-------------|
| Inverted hints | `check_guess` in `app.py` | `guess > secret` returned "Go HIGHER!" when it should say "Go LOWER!", and vice versa |
| Hard range too easy | `get_range_for_difficulty` | Hard was `1–50`, a smaller range than Normal (`1–100`), making it easier not harder |
| New Game ignores difficulty | `app.py` new-game block | Always picked a secret from `randint(1, 100)` regardless of selected difficulty |
| Info box hardcoded | `app.py` info bar | Always displayed "between 1 and 100" even on Easy (1–20) or Hard |
| Even-attempt string cast | `app.py` submit block | On every even attempt the secret was cast to a string, so `"9" > "50"` = True lexicographically, flipping the hint |
| New Game didn't reset status/history | `app.py` new-game block | After a win or loss, `status` and `history` were not cleared, making the game unrestartable |

**Fixes Applied:**

1. Swapped the hint messages in `check_guess` (`"Go HIGHER!"` ↔ `"Go LOWER!"`) and moved the function to `logic_utils.py`.
2. Changed Hard difficulty range from `1, 50` to `1, 500` in `get_range_for_difficulty`.
3. Replaced `random.randint(1, 100)` in the New Game handler with `random.randint(low, high)` and added resets for `status` and `history`.
4. Replaced the hardcoded info string with `f"Guess a number between {low} and {high}."`.
5. Added `# FIX:` comments near each change to document the AI-assisted repair process.

## 📸 Demo Walkthrough

Sample game on **Normal** difficulty (secret = **63**, 8 attempts allowed):

1. Player selects **Normal** from the sidebar. The info bar shows "Guess a number between 1 and 100. Attempts left: 7."
2. Player enters **40** → 40 < 63, game returns **"📈 Go HIGHER!"** Score: −5.
3. Player enters **80** → 80 > 63, game returns **"📉 Go LOWER!"** Score: −10.
4. Player enters **60** → 60 < 63, game returns **"📈 Go HIGHER!"** Score: −15.
5. Player enters **65** → 65 > 63, game returns **"📉 Go LOWER!"** Score: −20.
6. Player enters **63** → exact match, game returns **"🎉 Correct!"** and fires balloons. Win bonus of 30 points added (100 − 10 × 7, floored at 10). Final score: **10**.
7. Status changes to `won`. The input and Submit button are locked. Player sees "You won! The secret was 63. Final score: 10."
8. Player clicks **New Game 🔁** → attempts reset to 0, a new secret is drawn from 1–100 (Normal range), `status` resets to `"playing"`, history clears. Game is fully playable again.

**Screenshot** *(optional)*: <!-- Insert a screenshot of your fixed, winning game here -->

## 🧪 Test Results

```
$ pytest tests/test_game_logic.py -v
============================= test session starts ==============================
platform darwin -- Python 3.14.4, pytest-9.0.3, pluggy-1.6.0
collected 5 items

tests/test_game_logic.py::test_winning_guess PASSED                      [ 20%]
tests/test_game_logic.py::test_guess_too_high PASSED                     [ 40%]
tests/test_game_logic.py::test_guess_too_low PASSED                      [ 60%]
tests/test_game_logic.py::test_too_high_hint_says_go_lower PASSED        [ 80%]
tests/test_game_logic.py::test_too_low_hint_says_go_higher PASSED        [100%]

============================== 5 passed in 0.01s ===============================
```

## 🚀 Stretch Features

- [ ] [If you choose to complete Challenge 4, describe the Enhanced UI changes here — a screenshot is optional]
