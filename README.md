# Tower of Hanoi — powered by SWI-Prolog

## Project structure
```
hanoi-prolog/
├── hanoi.pl        <- Prolog logic engine (solver, rules, validation)
├── server.pl       <- Prolog HTTP server (API + serves the website)
├── tests.pl        <- Prolog unit tests
├── README.md       <- this guide
└── public/
    ├── index.html  <- page structure
    ├── style.css   <- styling
    └── app.js      <- drawing, animation, calls to the Prolog API
```

## How it works
The browser draws the board but decides nothing. Every start position, move check,
solution and win check is computed by Prolog:

| Endpoint | Prolog predicate | Purpose |
|---|---|---|
| `GET /api/new?disks=N` | `initial_state/2`, `min_moves/2` | new puzzle + minimum moves (2^N - 1) |
| `GET /api/solve?disks=N` | `hanoi_moves/2` | optimal move list for auto-solver |
| `POST /api/move` | `try_move/5`, `solved/2` | validate a manual move, detect win |

**Why SWI-Prolog?** Free, actively maintained, Windows installer, and it ships with a
built-in multi-threaded HTTP server and JSON libraries, so no extra dependencies are needed.

---
## Phase 1 — Install software (Windows)
1. Go to https://www.swi-prolog.org/Download.html and download the **Stable release for Windows 64-bit** (`.exe`).
2. Run the installer. Keep defaults, and **tick "Add swipl to the system PATH for all users" (or current user)**.
3. Close any open Command Prompt windows, then open a **new** one (Start → type `cmd` → Enter).
4. Verify (in **Command Prompt**):
   ```
   swipl --version
   ```
   Expected: `SWI-Prolog version 9.x.x for ...`
5. Other tools: only a web browser (Chrome, Edge, Firefox) and a text editor (Notepad, or VS Code). Nothing else.

## Phase 2 — Project setup
In **Command Prompt**:
```
cd %USERPROFILE%\Documents
mkdir hanoi-prolog
cd hanoi-prolog
mkdir public
```
Put `hanoi.pl`, `server.pl`, `tests.pl` in `hanoi-prolog`, and `index.html`, `style.css`, `app.js` in `hanoi-prolog\public`.
(If you downloaded the ZIP, just extract it there.) In Notepad choose *Save as type: All Files* so names don't end in `.txt`.

## Phase 3 — Test the Prolog logic alone
Run the unit tests (**Command Prompt**, inside the project folder):
```
swipl -g run_tests -t halt tests.pl
```
Expected: `% All 20 tests passed`

Try it by hand — start Prolog with `swipl hanoi.pl`; you will see a `?-` prompt. These lines are typed **in Prolog** (end each with a period):
```
?- hanoi_moves(3, Moves).
?- min_moves(5, X).
?- initial_state(3, S), try_move(S, 0, 2, S2, R).
?- halt.
```
Expected for the first: a list of 7 `move(Disk,From,To)` terms. Press `;` or Enter to finish.

## Phase 5 — Run the website
In **Command Prompt**, inside the project folder:
```
swipl server.pl
```
You should see `=== Tower of Hanoi server running ===`, then a `?-` prompt (leave this window open).
Open a browser at **http://localhost:8080**. The top badge should say *Prolog backend connected*.
To stop: type `halt.` at the `?-` prompt (or Ctrl+C twice).

To use another port, start with: `swipl -g "start(9000)" server.pl` (then use `localhost:9000`).

## Phase 6 — Testing the features
| Test | What to do | Expected |
|---|---|---|
| Valid move | click Source, then Destination | disk moves, "Good move!", moves = 1 |
| Invalid: big on small | move disk 1 to Destination, then try Source → Destination | red message, tower shakes, move not counted |
| Invalid: empty tower | click an empty tower first | red message |
| Drag and drop | drag the top disk onto another tower | same checks apply |
| Disk count | change the dropdown (1–10) | puzzle resets, Minimum = 2^N - 1 |
| Auto-solve | click Auto-Solve | animation, moves reach exactly the minimum |
| Pause / Resume | click Pause, then Resume | animation stops and continues |
| Restart | click Restart Solution | solution replays from the start |
| Speed | drag slider while solving | next moves use new speed |
| Win | finish a puzzle | congratulations dialog with moves and time |
| API directly | open http://localhost:8080/api/solve?disks=3 | JSON with 7 moves |

## Troubleshooting
| Problem | Fix |
|---|---|
| `'swipl' is not recognized` | Close/reopen Command Prompt; or reinstall with the PATH option ticked; or use the full path `"C:\Program Files\swipl\bin\swipl.exe" server.pl` |
| Badge says *NOT reachable* | Server isn't running. Run `swipl server.pl` and keep the window open; refresh the page |
| `Address already in use` / port error | Another program uses 8080. Start with `swipl -g "start(9000)" server.pl` and open `localhost:9000` |
| `source_sink 'hanoi' does not exist` | Run `swipl server.pl` from inside the project folder; `hanoi.pl` must sit next to `server.pl` |
| Page blank / 404 | `index.html`, `style.css`, `app.js` must be inside the `public` folder |
| File saved as `app.js.txt` | Enable *File name extensions* in Explorer's View menu and rename |
| Windows Firewall prompt | Choose *Allow* for private networks (only needed for localhost) |
| Changes to `.pl` files not applied | Stop the server (`halt.`) and start it again; for HTML/CSS/JS just refresh (Ctrl+F5) |
| Prolog prompt shows `|:` | Prolog is waiting for more input; type `.` and Enter, or Ctrl+C then `a` |
