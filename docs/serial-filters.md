# Serial Monitor Filters

The serial monitor includes a filter toolbar that lets you transform, suppress, and annotate incoming serial output in real time. Filter settings are persisted to VS Code settings via the **Save** button.

---

## Toolbar Overview

```
Filters: [✓ Timestamp] | Suppress: [regex…] | Highlight: [regex…] | Dedup: [regex…] after [N] | [Save]
```

All filters operate on the **raw text** of each line (including ANSI escape codes) before it is rendered to the DOM.

---

## Timestamp

Prepends a dimmed `[HH:MM:SS.mmm]` timestamp to every completed line.

**VS Code setting:** `esp-decoder.serialFilters.timestamp` (boolean, default `false`)

**Example output:**

```
[12:34:56.789] I (1234) wifi: connected to AP
[12:34:56.812] I (1235) app_main: starting loop
```

Useful for correlating serial events with external logs or measuring time between messages.

---

## Suppress

Hides lines whose full text matches a given regular expression. The line is removed from the DOM entirely — it does not count toward the line limit.

**VS Code setting:** `esp-decoder.serialFilters.suppressPattern` (string, default `""`)

The input field turns red if the regex is invalid.

**Examples:**

| Pattern | Effect |
|---|---|
| `^\s*$` | Hide blank / whitespace-only lines |
| `^\[V\]` | Hide verbose-level ESP-IDF log lines |
| `heap_caps` | Hide any line containing `heap_caps` |
| `^D \(\d+\)` | Hide ESP-IDF debug lines |

---

## Highlight

Wraps every match of a regex in ANSI inverse-video styling, making matches visually stand out without hiding any content.

**VS Code setting:** `esp-decoder.serialFilters.highlightPattern` (string, default `""`)

The input field turns red if the regex is invalid.

**Examples:**

| Pattern | Effect |
|---|---|
| `ERROR` | Highlight every occurrence of the word ERROR |
| `0x[0-9a-fA-F]+` | Highlight hex addresses |
| `\d+\.\d+\.\d+\.\d+` | Highlight IP addresses |
| `(?i)assert` | Highlight assert (case-insensitive) |

---

## Dedup (Repeated Character Collapse)

Collapses repeated occurrences of a regex match within a single line. The first N matches are rendered normally; further matches are suppressed and a live counter badge (`×N`) is appended inline to the line.

The badge updates in real time as more matching characters arrive — no new DOM lines are created.

**VS Code settings:**

| Setting | Type | Default | Description |
|---|---|---|---|
| `esp-decoder.serialFilters.dedupPattern` | string | `""` | Regex matching the character(s) to collapse |
| `esp-decoder.serialFilters.dedupThreshold` | number | `3` | Number of matches to show before collapsing |

The input field turns red if the regex is invalid. Set pattern to empty or threshold to `0` to disable.

**Examples:**

### ArduinoOTA progress dots

ArduinoOTA prints a `.` for every kilobyte transferred, all on one line:

```
...................................................................................................
```

With pattern `\.` and threshold `3`:

```
... ×98
```

### ESP-IDF flash progress `#`

```
######## ×512
```

Pattern: `#`, threshold: `8`

### Spinner characters

```
| ×47
```

Pattern: `[|/\-\\]`, threshold: `1`

### Repeated log prefix

Collapse repeated identical words within a line:

Pattern: `OK`, threshold: `2` turns `OK OK OK OK OK` into `OK OK ×3`

---

## Saving Settings

Click **Save** in the filter toolbar to write all current filter values to your global VS Code settings (`settings.json`). They are restored automatically the next time the panel opens.

You can also edit them directly in VS Code settings under **ESP Decoder > Serial Filters**.

Settings are also re-applied immediately if you change them in `settings.json` while the panel is open.
