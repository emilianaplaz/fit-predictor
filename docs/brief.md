# Fit Predictor — Brief

Decisions behind this brief are logged in [decisions.md](decisions.md) (D24–D31). The earlier board-analysis and trend-radar plans are parked there (D1–D23), not deleted.

## What it is
A tool that predicts whether a clothing item will fit a shopper. Given a customer's measurements, an item and the size they chose, it predicts small, fit or large and suggests a better size. A findings page shows where fit goes wrong most often.

## Why
Portfolio piece for fashion tech roles. Fit problems drive returns, which cost fashion retailers a lot, so this speaks directly to employers. It should demonstrate data cleaning, data analysis, a prediction model judged honestly against baselines, and a polished demo (D24, D25).

## Data
- ModCloth clothing fit dataset from a UCSD paper (RecSys 2018), available on Kaggle (D26).
- Verified on the file (D32): 82,790 rows, 47,958 users and 1,378 items (the papers reported 5,012 products).
- Fit feedback: fit 68.6%, large 15.8%, small 15.7%. Always predicting "fit" scores 68.6%, so accuracy alone is misleading.
- Usable measurements: height, bra size, cup size and hips. Waist (3% present) and bust (14%) are too sparse.
- Not usable as inputs: length, quality, review text and review summary. They are reported after wearing the item and would leak the answer.
- Personal data: user_name, user_id and review text must be dropped before anything is shared or shown.
- License: CC BY 4.0, as reported from the Kaggle listing (D35). The README must credit the source, link the license and state what was changed. The raw file is not republished.

## Analysis
- Split train and test by shopper, so no user appears on both sides (D33).
- Train on all rows. Leave "new" and "sale" out of the category charts, since they are shop sections and not clothing types (D34).
- Clean the data and explore it: how fit feedback varies by category, size and body measurements.
- Build baselines: always predict "fit", and a simple size rule.
- Build the model and compare it with the baselines. Report how well each class (small, fit, large) is caught, not just overall accuracy (D27).
- Write down the main findings: which categories and sizes fail most, and what the model can and cannot tell a retailer.

## Demo app
- **Try-it form:** enter measurements, pick an item and a size, get "likely too small / fits / too large" and a better size.
- **Findings page:** the baseline comparison and charts of which categories and sizes fail most (D28).

## Stack
- Analysis and model: Python with pandas and scikit-learn, in notebooks first.
- API: FastAPI, serving the trained model.
- Web: Next.js + TypeScript + Tailwind (D29). The existing nested folder is a Vite app, which will be replaced by a Next.js app in web/ when the web step starts (D31).
- Runs locally for now. Hosting is decided at the end (D30).

## Build order
1. Download the ModCloth file from Kaggle and check fields, missing values, class balance and license.
2. Explore and clean the data. Write the first findings.
3. Build the baselines, then the model, and compare them per class.
4. Save the trained model and serve it with FastAPI.
5. Build the Next.js try-it form and findings page.
6. Write the README with the findings and limits.

## Parked
The board-to-shopping analysis and the Google Trends radar. See D1–D23 in [decisions.md](decisions.md) for what was decided and learned, including the Fashionpedia findings (D23).

## Still open
- None. The planning questions are settled.
