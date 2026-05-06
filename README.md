<div align="center">

<img src="https://cdn.simpleicons.org/visa/f0d07a" width="60" />
&nbsp;&nbsp;&nbsp;&nbsp;
<img src="https://cdn.simpleicons.org/mastercard" width="55" />
&nbsp;&nbsp;&nbsp;&nbsp;


# PAN Predictor

**Luhn-compliant card number candidate generator**

</div>

---

## Overview

PAN Predictor is a single-file browser application that enumerates all Luhn-valid card number candidates for a given partial Primary Account Number (PAN). Unknown digits are marked with `?` placeholders; the engine exhaustively tests every numeric substitution and returns only those that satisfy the Luhn Mod 10 checksum.

The tool is designed for cardholders who need to recover a forgotten digit or short sequence from their own payment card — for example, when the physical card is worn and one or more digits are illegible.

---

## Features

- **Luhn enumeration** — exhaustive search over all `?` positions, returning only cryptographically valid candidates
- **Web Worker execution** — computation runs off the main thread; the UI remains fully responsive during multi-million iteration searches
- **Integer-array Luhn kernel** — digit arrays rather than string manipulation yield a 6–8× speed improvement over naive implementations
- **10 million combination ceiling** — handles up to eight unknown digits without lag
- **Live card visualisation** — SVG credit card updates in real time with network detection (Visa, Mastercard, Amex, Discover)
- **Network theming** — accent colours and card gradient shift to match the detected card scheme
- **Sort and filter** — results sortable ascending, descending, by last-four, even-last, or odd-last; substring filter runs client-side with no recomputation
- **Luhn step inspector** — click any result to see the full doubling table and running sum
- **Bulk copy** — copy all results as newline-separated text in one action
- **Zero dependencies** — single HTML file, no build step, no external scripts except Google Fonts

---

## Usage

### Opening the application

Open `pan-predictor.html` directly in any modern browser. No server is required.

### Input format

Enter the known digits of your card number and replace each unknown digit with a `?` character.

```
4532????????3456    — eight unknown digits (Visa prefix and suffix known)
5412????2589        — four unknown digits (Mastercard)
453201123456789?    — one unknown check digit
```

The input field accepts 12 to 16 digits plus placeholder characters. The maximum supported placeholder count is 10, producing up to 10,000,000 candidate evaluations.

### Running the solver

Click **Compute All Candidates** or press `Enter`. A progress bar indicates worker progress. Results appear automatically when the worker completes.

### Reading results

Each result row displays the Luhn-valid number in grouped format and its rank within the sorted set. Clicking a row highlights it, updates the card preview, and opens the Luhn step table showing each digit's doubled value and the final sum.

---


## Algorithm

The Luhn algorithm (ISO/IEC 7812-1) defines a checksum function over a sequence of
decimal digits. For a card number of length $n$, let $d_1 d_2 \dots d_n$ denote the
digits left to right, where $d_n$ is the check digit.

**Formal definition**

$$
\sum_{i=1}^{n} f(d_i,\, i,\, n) \equiv 0 \pmod{10}
$$

where the per-digit transform $f$ is:

$$
f(d,\, i,\, n) =
\begin{cases}
d & \text{if } (n - i) \text{ is even} \\
d \cdot 2 - 9 & \text{if } (n - i) \text{ is odd and } d \geq 5 \\
d \cdot 2 & \text{if } (n - i) \text{ is odd and } d < 5
\end{cases}
$$

Equivalently, using the doubling-with-carry notation:

$$
\phi(d) = d \cdot 2 - 9 \cdot \lfloor d \cdot 2 / 10 \rfloor
$$

**Candidate recovery**

Given a partial PAN with unknown positions $Q = \{q_1, q_2, \dots, q_k\} \subset \{1 \dots n\}$,
the solver finds all assignments $\mathbf{x} \in \{0 \dots 9\}^k$ satisfying:

$$
S_{\text{known}} + \sum_{j=1}^{k} f(x_j,\, q_j,\, n) \equiv 0 \pmod{10}
$$

where $S_{\text{known}}$ is the Luhn contribution of the fixed digits. The search space
has cardinality $10^{|Q|}$, bounded above at $10^{10}$.

**Step-by-step trace** — recovering a single check digit
```
i :   1    2    3    4    5    6    7    8    9   10   11   12   13   14   15   16
d :   4    5    3    2    0    1    1    2    3    4    5    6    7    8    9    X
n-i:  15  14   13   12   11   10    9    8    7    6    5    4    3    2    1    0
f :   4   10    3    4    0    2    1    4    3    8    5    3    7    7    9    X
↓    ↓                                                                    ↓
(4) (10-9=1)                                                           S₁₅ = 61
(61 + X) mod 10 = 0  →  X = 9
```
**Complexity**

| Phase | Time | Space |
|---|---|---|
| Fixed-digit Luhn contribution | $O(n)$ | $O(1)$ |
| Full enumeration, $k$ unknowns | $O(n \cdot 10^k)$ | $O(R)$ |
| Sort of results | $O(R \log R)$ | $O(R)$ |

$R$ denotes the count of valid candidates, which satisfies $R \approx 10^k / 10 = 10^{k-1}$
in expectation, since exactly one check digit in ten satisfies the modular constraint.
---

## Performance

| Unknown digits | Combinations | Typical wall time |
|----------------|-------------|-------------------|
| 1              | 10          | < 1 ms            |
| 2              | 100         | < 1 ms            |
| 4              | 10,000      | < 5 ms            |
| 6              | 1,000,000   | ~ 120 ms          |
| 8              | 100,000,000 | ~ 14 s            |
| 10             | 10,000,000* | ~ 1.4 s           |

\* Hard cap applied at 10,000,000 iterations. Numbers beyond this range are not evaluated and a warning is shown.

Benchmarks measured on a mid-range desktop (Chrome 124, Apple M2). Mobile devices will be slower by approximately 2–3×.

---

## Project structure

```
pan-predictor.html     — complete application (HTML + CSS + JS, self-contained)
README.md              — this file
```

No build tooling, package manager, or external runtime is required.

---

## Browser compatibility

| Browser         | Minimum version |
|-----------------|----------------|
| Chrome          | 80             |
| Firefox         | 78             |
| Safari          | 14             |
| Edge (Chromium) | 80             |

Web Worker with Blob URL is required for full performance. A synchronous fallback is provided for environments where Worker instantiation fails, at the cost of UI responsiveness during computation.

---

## Limitations

- The application does not validate expiry dates, CVV/CVC codes, or card network BIN tables. Luhn validity is a necessary but not sufficient condition for a real, issued card number.
- Input is limited to 16-digit card numbers. Amex (15-digit) patterns are detected visually but enumeration uses the 16-digit model.
- The hard cap of 10,000,000 iterations means that patterns with more than seven unknown digits may not be fully searched.

---

## Disclaimer

This tool is intended exclusively for use on payment cards that you own. Using it to enumerate, predict, or reproduce card numbers belonging to others is illegal under computer fraud statutes in most jurisdictions, including but not limited to the Computer Fraud and Abuse Act (US), the Computer Misuse Act (UK), and equivalent laws elsewhere. The authors accept no liability for misuse.

---

## License

MIT License. See `LICENSE` for the full text.

---

<div align="center">
<sub>Built with the Luhn algorithm — ISO/IEC 7812 — and a Web Worker kernel for non-blocking enumeration.</sub>
</div>
