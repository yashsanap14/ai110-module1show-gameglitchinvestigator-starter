# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
When I first ran `python -m streamlit run app.py`, the UI loaded fine and I could
see the Developer Debug Info expander, but the game itself behaved like it was
working against me. The hints sent me in the wrong direction, the difficulty
levels did not match their names, and the secret number seemed to change its
mind every other guess.

- List at least two concrete bugs you noticed at the start  
  (for example: "the hints were backwards").
  1. The hints are in reverse direction
  2. Difficulty levels are upside down. The hard level range 1-50 while others have range 1-100. Also the normal level has more attempts then the easy level
  3.I expected the same guess against the same secret to always return the same hint.
    Instead, on every other guess the comparison flips because the secret is
    being treated as a string ("9" > "50" is true alphabetically), so the
    hint contradicts the previous attempt.

**Bug Reproduction Log**

Document at least 3 bugs you found. Add rows as needed.

| Input | Expected Behavior | Actual Behavior | Console Output / Error |
|-------|-------------------|-----------------|------------------------|
| Difficulty = Normal, secret shown as **42** in Debug Info, guess = `80` | Hint says "Go LOWER" because 80 > 42 | Hint says "📈 Go HIGHER!" — points the wrong direction | none (silent logic bug) |
| Open the sidebar and switch between Easy / Normal / Hard | Easy = most attempts + smallest range; Hard = fewest attempts + widest range | Easy = 6 attempts (1–20), Normal = 8 attempts (1–100), Hard = 5 attempts (1–50). Normal has more attempts than Easy, and Hard's range is smaller than Normal's. | none |
| Secret = **50** (from Debug Info), guess `9` on attempt 1, then guess `9` again on attempt 2 | Same input + same secret should give the same hint both times ("Too Low") | Attempt 1 → "Too Low"; attempt 2 → "Too High" for the *same* guess, because the secret is cast to a string on even attempts and `"9" > "50"` lexicographically | none |
| Lose or win a game, then click **"New Game 🔁"** | Score resets to 0, status returns to "playing", a new secret is drawn from the current difficulty range | Score and status persist, so the next rerun hits `st.stop()` and the game cannot actually restart; the new secret is always drawn from 1–100 regardless of difficulty | none |

---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)?

I used **Claude Code** (Anthropic's CLI coding assistant) throughout this project as my primary AI collaborator.

---

### Correct AI Suggestion

**What the AI suggested:**
When I described the inverted hints, Claude Code analyzed `check_guess` in `app.py` and correctly identified that the outcome labels (`"Too High"`, `"Too Low"`) were logically right, but the direction strings in the messages (`"Go HIGHER!"` / `"Go LOWER!"`) were swapped. It suggested swapping the emoji+text so that `guess > secret` returns `"📉 Go LOWER!"` and `guess < secret` returns `"📈 Go HIGHER!"`.

**Whether the suggestion was correct:**
It was correct. The root cause was exactly a copy-paste inversion in the string literals — the comparison logic itself was fine.

**How I verified it:**
I traced through a concrete example by hand: secret = 50, guess = 60. Since 60 > 50, the branch `if guess > secret` fires and should tell the player to go lower. After the fix, that branch returns `"📉 Go LOWER!"` which matches the expected direction. I then confirmed it programmatically with the pytest test `test_too_high_hint_says_go_lower`, which asserts `"LOWER" in message` for guess=60, secret=50, and it passed.

---

### Incorrect / Misleading AI Suggestion

**What the AI suggested:**
When I asked about the `check_guess` refactor, Claude Code initially kept the `TypeError` fallback branch — the block that converts the guess to a string and does lexicographic comparison — without flagging it as a separate active bug. The suggestion treated it as a safe edge-case handler rather than identifying it as the cause of the "same guess gives different hints on even vs. odd attempts" bug.

**Whether the suggestion was correct:**
It was misleading. The `TypeError` branch is not a harmless edge case — it is reached intentionally on every even-numbered attempt because the app itself casts the secret to a string at line 159-160 (`secret = str(st.session_state.secret)`). String comparison (`"9" > "50"` is `True` alphabetically) produces wrong hints, and the refactored code carried this flaw forward unchanged.

**How I verified it:**
I opened the Debug Info expander during a live game and guessed the same number twice in a row (e.g., guess `9` against a secret of `50`). Attempt 1 correctly said "Too Low". Attempt 2 said "Too High" for the identical input — because `"9" > "50"` lexicographically evaluates to `True`. Reading the code confirmed that the `% 2 == 0` branch on line 159 deliberately triggers the string path that causes this.

---

## 3. Debugging and testing your fixes

**How I decided a bug was really fixed:**
I required two things before marking a bug closed: (1) a manual trace through the logic with a concrete example confirming the expected path, and (2) a passing automated test that would have caught the original broken behavior. If the test would not have failed against the buggy code, the test was rewritten.

**Tests I ran:**

I wrote a pytest suite in `tests/test_game_logic.py`. The two most targeted tests for the hint-inversion fix were:

```
test_too_high_hint_says_go_lower  — check_guess(60, 50) → message must contain "LOWER"
test_too_low_hint_says_go_higher  — check_guess(40, 50) → message must contain "HIGHER"
```

Running `pytest tests/test_game_logic.py -v` showed all 5 tests passing. Before fixing `check_guess`, these same assertions would have failed because the original code returned `"Go HIGHER!"` when guess > secret. The tests would have caught the bug in CI before it shipped.

I also manually verified the Hard difficulty range fix by reading the sidebar caption after selecting Hard — it now shows "Range: 1 to 500" instead of "Range: 1 to 50", and the new-game secret drawn via Debug Info falls within that wider range.

**Did AI help design or understand the tests?**
Yes. Claude Code pointed out that the existing starter tests were comparing `result == "Win"` directly against a tuple return value `("Win", "🎉 Correct!")`, which would always fail. It suggested unpacking the tuple with `outcome, _ = check_guess(...)` to isolate the outcome from the message. It also recommended writing separate tests for the *message* content (not just the outcome label) so the hint direction itself is explicitly verified — which is exactly what `test_too_high_hint_says_go_lower` does.

---

## 4. What did you learn about Streamlit and state?

- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
- What is one thing you would do differently next time you work with AI on a coding task?
- In one or two sentences, describe how this project changed the way you think about AI generated code.
