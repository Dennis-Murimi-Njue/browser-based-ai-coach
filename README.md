# Daily Goal History

A small, dependency-free web app that predicts whether you'll hit a daily health goal from your sleep, water intake and steps, then keeps a running log of each day and your hit rate.

Everything runs in the browser from a single HTML file. There is no build step, no framework and no server.

## Features

- Rule-based goal prediction from three inputs: sleep, water and steps
- A confidence value for each prediction
- A session history table with time, inputs, result (HIT or MISS) and confidence
- A live hit rate (for example `67% (2/3)`) shown above the table and written to the console
- Responsive layout with automatic light and dark mode

## Getting started

1. Clone the repository:
   ```bash
   git clone <your-repo-url>
   cd <your-repo-folder>
   ```
2. Open `history-log.html` in any modern browser. You can double-click the file or run:
   ```bash
   open history-log.html        # macOS
   xdg-open history-log.html    # Linux
   start history-log.html       # Windows
   ```

That's all. No installation is needed.

## Usage

1. Enter sleep (hours), water (glasses) and steps.
2. Click **Log this day**.
3. The new entry appears in the table, the hit rate updates, and the form clears.

If any field is empty or not a number, the submission is ignored.

Open the browser's developer console to see the debug output (hit rate and entry details).

## How the prediction works

Each input is compared to a daily target, capped at 100% of that target, then multiplied by its weight:

| Metric | Target | Weight (max points) |
|---|---|---|
| Sleep | 8 hours | 35 |
| Water | 10 glasses | 25 |
| Steps | 12,000 | 40 |

```
score = min(sleep/8, 1) * 35 + min(water/10, 1) * 25 + min(steps/12000, 1) * 40
```

- **Hit or miss:** a score of 60 or more counts as a HIT.
- **Confidence:** starts at 50% at the threshold and grows by 1.2 percentage points for each point of distance from 60, capped at 95%.

  ```
  confidence = min(0.50 + |score - 60| * 0.012, 0.95)
  ```

Because of the caps, exceeding a target earns no extra points. Sleeping 10 hours scores the same as 8.

### Changing the weights or threshold

The prediction lives in the `predictGoal` function inside the `<script>` block. Edit the multipliers (`35`, `25`, `40`) there. Keep them summing to 100 so the score stays on a 0 to 100 scale. The threshold is the `60` in `total >= 60`.

## Project structure

```
.
├── history-log.html   # Markup, styles and script in one file
└── README.md
```

Inside `history-log.html`:

| Part | Purpose |
|---|---|
| `sessionLog` | In-memory array holding every logged entry |
| `logEntry` | Builds an entry object, saves it, and returns it |
| `renderHistory` | Redraws the history table from `sessionLog` |
| `predictGoal` | Computes score, hit or miss, and confidence |
| Submit handler | Reads the form, runs the prediction, logs it, and updates the hit rate |

## Limitations

- **No persistence.** The log lives in memory, so refreshing or closing the page clears it. Saving across visits would need `localStorage`.
- **Minimal validation.** Empty and non-numeric input is rejected, but negative or unrealistic values (such as 100 hours of sleep) are not.
- **Simple heuristic.** The weights and threshold are hand-picked rules, not a trained model. The "confidence" reflects distance from the threshold, not a statistical probability.

## Ideas for next steps

- Persist the log with `localStorage`
- Show the newest entry first
- Add the coaching messages (a lookup table keyed on hit/miss, sleep and water flags)
- Stronger input validation with inline error messages
- Export the history as CSV

## License

Add a license of your choice (for example MIT) before publishing.