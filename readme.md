# Reddit Data Scraping, NLP Analysis, and CSV Separation Pipeline

This repository contains a full end-to-end workflow for collecting Reddit posts, cleaning and analyzing their text, and exporting structured datasets.  
All three major components are implemented in **Jupyter notebooks (.ipynb)** for clarity and reproducibility.

Workflow overview:

1. **Scraper Notebook** → collects Reddit posts & comments  
2. **Analysis Notebook** → cleans text, runs sentiment analysis & topic modeling  
3. **Separator Notebook** → filters the combined dataset into subsets  

---

## 📁 Project Structure

- `reddit_scraper.ipynb` – Reddit scraping pipeline (PRAW)  
- `reddit_auth0.py` – Template for API credentials  
- `reddit_analysis.ipynb` – NLP pipeline (cleaning → sentiment → topics)  
- `reddit_csv_separator.ipynb` – Keyword-based CSV splitting  
- `requirements.txt` – Python dependencies  
- `data/` – (optional) sample data folder  

---

# 🔹 1. Scraper Notebook (reddit_scraper.ipynb)

This notebook handles **data collection from Reddit**.

### How it works 

1. **Provide keywords** (e.g., “Zyn”, “nicotine pouch”, “quit nicotine”).  
2. For each keyword, the notebook:
   - searches Reddit using PRAW  
   - retrieves the top N relevant posts  
   - stores their URLs and associated keyword labels  
3. It **removes duplicates**, filters out non-text posts (e.g., videos/images).  
4. For each unique URL:
   - loads the Reddit submission  
   - extracts post metadata (title, text body, author, subreddit, timestamp, upvotes)  
   - saves it to `reddit_post_log.csv`  
5. It then fetches **every top-level comment and all replies**  
   - extracts comment text, author, timestamp, upvotes  
   - labels whether a row is a “comment” or a “reply”  
   - saves to `reddit_comment_log.csv`  

This results in a complete, labeled dataset of posts + comments tied to your keywords.

**Outputs:**
- `reddit_post_log.csv`  
- `reddit_comment_log.csv`  

---

# 🔹 2. Analysis Notebook (reddit_analysis.ipynb)

This notebook performs **all NLP processing** on the scraped text.

### How it works 

1. **Merges post CSV + comment CSV** into one combined dataset.  
2. Cleans the text:
   - lowercase  
   - remove punctuation  
   - strip URLs  
   - remove stopwords  
   - tokenize words  
   - lemmatize them using spaCy  
3. **Sentiment Analysis (VADER)**:
   - calculates `pos`, `neg`, `neu`, and `compound` scores  
   - assigns each row a sentiment label (Positive / Neutral / Negative)  
4. **Topic Modeling (LDA)**:
   - builds dictionary & corpus from cleaned text  
   - trains Gensim’s LDA model  
   - identifies latent topics (e.g., “quitting”, “flavors”, “side effects”)  
   - extracts top words per topic  
   - computes topic coherence  
5. Performs **basic statistics**:
   - sentiment distribution  
   - topic frequencies  
   - keyword summary counts  

**Outputs (examples):**
- `reddit_combined_final.csv`  
- `reddit_combined_with_vader.csv`  

These files contain cleaned text, sentiment labels, and topic assignments.

---

# 🔹 3. Separator Notebook (reddit_csv_separator.ipynb)

This notebook **splits** the final combined dataset into smaller CSV files based on keyword triggers.

### How it works 

1. Loads the combined CSV (`reddit_combined_final.csv`).  
2. Searches the `post` text for specific strings such as:
   - “quit”  
   - “wedding”  
   - “try”  
   - “shop”  
3. Creates filtered subsets (e.g., all posts containing “quit”).  
4. Writes each subset as:
   - individual `.csv` files  
   - separate sheets in one `.xlsx` workbook  

This is useful for manual review or researcher annotation.

**Outputs:**
- `reddit_combined_quit_only.csv`  
- `reddit_combined_wedding_only.csv`  
- `reddit_combined_try_only.csv`  
- `reddit_combined_shop_only.csv`  
- `reddit_combined_filtered_final.xlsx`  

---

# ⚙️ Installation

### 1. Create a virtual environment
```
python3 -m venv venv
source venv/bin/activate       # macOS/Linux
venv\Scripts\activate          # Windows
```

### 2. Install dependencies
```
pip install -r requirements.txt
```

### 3. Install spaCy model (required for lemmatization)
```
python -m spacy download en_core_web_sm
```

### 4. Launch Jupyter Notebook
```
jupyter notebook
```

---

# ▶️ Usage

### Step 1 — Scrape Reddit Data
Open:
`reddit_scraper.ipynb`  
Run all cells to generate:
- `reddit_post_log.csv`
- `reddit_comment_log.csv`

### Step 2 — Analyze the Text
Open:
`reddit_analysis.ipynb`  
Run all cells to generate:
- `reddit_combined_final.csv`
- `reddit_combined_with_vader.csv`

### Step 3 — Split by Keyword
Open:
`reddit_csv_separator.ipynb`  
Run all cells to generate filtered datasets.

---

# 📦 Notes

- No real datasets are included for privacy and academic integrity.  
- `reddit_auth0.py` contains placeholder fields only — add your own credentials.  
- All output CSVs are created locally when you run the notebooks.  
- Verified on Python 3.9+ (spaCy & Gensim compatibility).  


---

#  Acknowledgements
- Reddit API (PRAW)  
- VADER Sentiment Analyzer  
- spaCy  
- Gensim  
- pandas / numpy / matplotlib  
