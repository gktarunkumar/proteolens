# ProteoLens

A simple web app that analyzes and compares protein sequences.
Paste protein sequences in FASTA format, and it calculates:
- Molecular weight
- Isoelectric point (pI)
- Instability index (is the protein likely stable or unstable?)
- Aromaticity
- GRAVY score (does it like water or avoid it?)
- Amino acid composition (%)

## How to run this (step by step)

1. **Open a terminal** in this folder (the one with `app.py` in it).

2. **Install the required packages** (only need to do this once):
   ```
   pip install -r requirements.txt
   ```
   If that gives a permissions error, try:
   ```
   pip install -r requirements.txt --break-system-packages
   ```

3. **Start the app**:
   ```
   python app.py
   ```

4. You'll see a message like `Running on http://127.0.0.1:5000`.
   Open that link in your web browser.

5. Click **"Load example"** to try it with sample data, or paste your own
   sequence(s) in FASTA format, then click **Analyze**.

## How the code is organized

- `app.py` — the backend: reads your pasted text, checks it's valid,
  runs the calculations, sends the results to the webpage.
- `templates/index.html` — the webpage itself: the form, the results table,
  and the charts.
- `static/style.css` — just makes it look clean instead of plain HTML.

## If something goes wrong during your demo

- **"Contains invalid character(s)"** error means your sequence has a
  letter that isn't one of the 20 standard amino acids (A, R, N, D, C, Q,
  E, G, H, I, L, K, M, F, P, S, T, W, Y, V). Double check for typos,
  numbers, or stray symbols.
- If the page looks unstyled, make sure the `static` folder is in the
  same place as `app.py` (don't move files around separately).
- If port 5000 is already in use, change the last line of `app.py` to
  use a different port, e.g. `port=5050`.

## Ideas if you have extra time before your demo

- Add a "download report" button (export the results table as a text/CSV file).
- Highlight the amino acid with the highest % in each sequence.
- Add a short written interpretation under each chart, e.g. "This protein is
  likely stable" based on the instability index.
