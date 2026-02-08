# FDA Warning Letters NLP Classification

Natural Language Processing system to identify regulatory code references in FDA warning letters using Word2Vec embeddings and logistic regression.

## Overview

This project develops a text classification system to automatically identify references to US regulatory code (CFR, USC) within FDA warning letters. The system uses web scraping, NLP preprocessing, and machine learning to detect regulatory citations, providing a foundation for compliance monitoring and regulatory text analysis.

## Problem Statement

The FDA publishes thousands of warning letters citing violations of US regulatory code. Manually identifying which code sections are referenced is time-consuming and error-prone. This project explores whether machine learning can automate the detection of regulatory code references in unstructured text.

## Dataset

- **Source:** FDA Warning Letters (publicly available at fda.gov)
- **Collection Method:** Web scraping using Selenium and BeautifulSoup
- **Corpus Size:** 3,400+ warning letters
- **Training Data:** 
  - 40 manually tagged letters
  - 561 positive samples (regulatory code references)
  - 969 negative samples (non-code 5-grams)
- **Code Format Examples:**
  - Shorthand: "21 CFR § 820", "(21 U.S.C. § 352)"
  - Full form: "Section 820 of the Federal Drugs & Cosmetics Act"
  - Abbreviated: "Section 820 of the FD&C Act"

## Methodology

### 1. Web Scraping
- **Stage 1:** Scraped table of all warning letters (metadata)
- **Stage 2:** Downloaded full text of 3,400+ individual letters
- **Tools:** Selenium WebDriver (dynamic content), BeautifulSoup (HTML parsing)
- **Time:** ~5.5 hours for complete corpus

### 2. Data Preprocessing
- Extracted letter body text (removed headers/footers)
- Preserved punctuation and special characters (§, brackets, etc.) due to regulatory code format
- No stemming or stop word removal (would damage code references)

### 3. NLP Model
- **Word Embeddings:** Word2Vec (100-dimensional vectors)
- **Training:** Entire corpus of 3,400+ letters
- **Feature Representation:** Row-wise average of token embeddings for each n-gram
- **N-gram Length:** 5-grams (captures typical code reference length)

### 4. Classification
- **Algorithm:** Logistic Regression (binary classification)
- **Positive Class:** Text is a regulatory code reference
- **Negative Class:** Text is not a regulatory code reference
- **Negative Sampling:** Random 5-grams from corpus

## Results

### Model Performance

| Metric | Value |
|--------|-------|
| **Overall Accuracy** | 74% |
| **Precision (negative)** | 0.74 |
| **Precision (positive)** | 0.72 |
| **Recall (negative)** | 0.88 |
| **Recall (positive)** | 0.50 |
| **F1 Score (negative)** | 0.81 |
| **F1 Score (positive)** | 0.59 |

### Key Findings

- **Strong negative class performance:** Model effectively identifies non-code text (88% recall)
- **Weak positive class recall:** Only identifies 50% of actual regulatory code references
- **Usable for filtering:** High negative recall makes it suitable for filtering out obvious non-code text
- **Not production-ready:** 50% recall insufficient for automated regulatory monitoring

### Error Analysis

Major issues identified:
1. **Bracket sensitivity:** Regulatory code often appears in brackets - removing them improved classification
2. **Format variation:** Model struggles with multiple code formats (shorthand vs full name vs abbreviated)
3. **N-gram length:** Fixed 5-gram approach doesn't capture variable-length references well

## Technologies Used

- **Language:** Python 3.11
- **Web Scraping:**
  - Selenium WebDriver 4.0
  - BeautifulSoup
- **NLP & ML:**
  - gensim (Word2Vec)
  - scikit-learn (Logistic Regression)
  - NLTK (tokenization)
- **Data Processing:**
  - pandas
  - numpy

## Files

- `MA5851_A3_Samuel_Bousfield_Code.ipynb` - Complete analysis pipeline (scraping → modeling → evaluation)
- `MA5851_A3_Samuel_Bousfield_Report.pdf` - Detailed technical report with methodology and results

## Running the Code
```python
# Install required packages
pip install selenium beautifulsoup4 gensim scikit-learn nltk pandas numpy

# Note: Selenium requires ChromeDriver
# Download from: https://chromedriver.chromium.org/

# Run Jupyter notebook
jupyter notebook MA5851_A3_Samuel_Bousfield_Code.ipynb
```

**Warning:** Re-running the web scraping will take ~5-6 hours and may trigger rate limiting on FDA servers.

## Limitations & Lessons Learned

### Data Preprocessing Choices
- **Bracket removal:** Should have been done during preprocessing (significantly improved accuracy in testing)
- **N-gram strategy:** Fixed-length 5-grams don't capture the variable length of regulatory references (some are 3 words, others 7+)
- **Manual tagging:** Only 40 documents tagged limits training data quality

### Model Architecture
- **Single model limitation:** Using one model for multiple code formats (shorthand, full name, abbreviated) diluted performance
- **Better approach:** Multiple specialized models or multi-class classification
- **Feature engineering:** Could benefit from additional features beyond Word2Vec (regex patterns, position in document, surrounding context)

### Sampling Strategy
- **Class imbalance:** Negative samples (969) vs positive samples (561) - closer to 2:1 than ideal 1:1
- **Negative sample quality:** Random 5-grams may not represent typical non-code text that might confuse the model

## Future Improvements

### Immediate Wins
1. Remove brackets during preprocessing
2. Implement separate models for different code formats
3. Variable-length n-gram approach (3-7 grams)
4. Expand manually tagged dataset (100+ documents)

### Advanced Enhancements
1. **Hybrid approach:** Combine ML with regex patterns for known code formats
2. **Contextual embeddings:** Use BERT or similar transformer models instead of Word2Vec
3. **Multi-task learning:** Predict both "is code?" and "which type of code?"
4. **Active learning:** Iteratively improve by focusing on misclassified examples
5. **Ensemble methods:** Combine multiple models for better recall

## Domain Applications

This approach could be adapted for:
- Automated compliance monitoring
- Legal document analysis
- Contract review (identifying specific clause references)
- Academic citation extraction
- Patent analysis (prior art references)

## Web Scraping Ethics

- All data is publicly available (fda.gov)
- No authentication required
- Respected robots.txt (note: actual implementation used faster scraping - see limitations)
- Data used for educational/research purposes only

## License

MIT License

---

**Note:** This was an exploratory academic project. The 74% accuracy and 50% recall indicate this approach needs significant refinement before production use. The value lies in identifying what works (negative class filtering, word embeddings) and what needs improvement (multiple models, better preprocessing, variable n-grams).
