# Privacy Permitter

Chrome extension that reviews installed extensions and scores their requested permissions for quick risk visibility.

## What it does

- Lists enabled installed extensions.
- Scores sensitive permissions and broad host access.
- Groups results into **Critical**, **Warning**, and **Safe**.
- Supports search and category filters.

The scoring model is a deterministic heuristic for awareness. It is not a definitive security verdict and does not establish that an extension is malicious or safe.

## Permission

- `management` — required to enumerate installed extensions and inspect their declared permissions.

## Install locally

1. Open `chrome://extensions/` in Chrome.
2. Enable **Developer mode**.
3. Select **Load unpacked**.
4. Choose the repository folder.

## Structure

```text
Privacy-Permitter/
├── manifest.json
├── popup/
│   ├── popup.html
│   ├── popup.css
│   └── popup.js
└── README.md
```

## Risk scoring contract

Privacy Permitter calculates a score from the permissions and host permissions reported by Chrome for each enabled installed extension. The score is the sum of the applicable weights.

### Permission weights

| Permission | Weight |
| --- | ---: |
| `all_urls` | 40 |
| `<all_urls>` | 40 |
| `browsingData` | 35 |
| `webRequest` | 30 |
| `webRequestBlocking` | 30 |
| `nativeMessaging` | 30 |
| `history` | 25 |
| `geolocation` | 25 |
| `tabs` | 20 |
| `cookies` | 20 |
| `clipboardRead` | 20 |
| `downloads` | 15 |

Permissions not listed above currently contribute a default weight of **2**. Unknown permission names are therefore handled without failing the calculation.

### Broad host access

The following host permissions are treated as broad access and add **40** points:

- `<all_urls>`
- `*://*/*`
- `https://*/*`
- `http://*/*`

The broad-host contribution is recorded as `<all_urls>` in the displayed flagged-permission list when it is not already present.

### Categories

- **Critical:** 50 or more
- **Warning:** 20–49
- **Safe:** below 20

The category is derived only from the final numeric score, so the same inputs always produce the same category.

### Representative examples

| Inputs | Score | Category |
| --- | ---: | --- |
| No permissions | 0 | Safe |
| `tabs` | 20 | Warning |
| `webRequest` | 30 | Warning |
| `<all_urls>` host access | 40 | Warning |
| `tabs` + `webRequest` | 50 | Critical |
| `cookies` + `<all_urls>` host access | 60 | Critical |

These examples form the baseline behavior for future permission-analysis regression coverage.

## Limitations

The score measures declared permission breadth using a fixed heuristic. It does not inspect an extension's source code, runtime behavior, network traffic, data handling, publisher reputation, update history, or actual use of granted permissions.

As a result:

- A legitimate extension can receive a high score because it needs broad permissions for its functionality.
- An extension can receive a low score while still presenting risks outside the permissions covered by this model.
- Permission names and browser capabilities can evolve, so the weight table must be reviewed as Chrome introduces new permission forms.
- The score should be used as a prompt for review rather than as a definitive security classification.

## Future work

Production hardening and Chrome Web Store work is tracked in [#1](../../issues/1).
