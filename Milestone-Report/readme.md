# How many stars will I give? Predicting ratings of Amazon reviews

	A Capstone project: 
          	Tonia Chu
	Under the mentorship: 
   		Srdjan Santic (Consulting Data Scientist at the Research Center for Cheminformatics)
	For the course: 
 		Data Science Career Track (Springboard)

## I. INTRODUCTION
Nowadays, a massive amount of reviews is available online. Besides offering a valuable source of information, these informational contents generated by users, strongly impact the purchase decision of customers. Many consumers are effectively influenced by online reviews when making their purchase decisions. Relying on online reviews has thus become a second nature for consumers.

In their research process, consumers want to find useful information as quickly as possible. However, searching and comparing text reviews can be frustrating for users. Indeed, the massive amount of text reviews as well as its unstructured text format prevent the user from choosing a product with ease. The star-rating, i.e. stars from 1 to 5 on Amazon, rather than its text content gives a quick overview of the product quality. This numerical information is the No. 1 factor used in an early phase by consumers to compare products before making their purchase decision.

However, many product reviews (from other platforms than Amazon) are not accompanied by a scale rating system, consisting only of a textual evaluation. In this case, it becomes daunting and time-consuming to compare different products in order to eventually make a choice between them. Therefore, models able to predict the user rating from the text review are critically important. Getting an overall sense of a textual review could in turn improve consumer experience. Also, it can help business to increase sales, and improve the product by understanding customers' needs and pain-points.

The purpose of this project is to develop models that are able to predict the user rating from the text review. While our model is built to work with any kind of product, the review dataset provided by Amazon only includes Clothing and Shoes reviews.
 
## II. Deeper dive into the data set
We get dataset from Amazon product data, which contains product reviews and metadata from Amazon, including 142.8 million reviews spanning May 1996 ~ July 2014. The dataset includes reviews (ratings, text, helpfulness votes), product metadata (descriptions, category information, price, brand, and image features), and links (also viewed/also bought graphs).

In this project, we use 5-core dataset of Clothing and Shoes, which is subset of the data in which all users and items have at least 5 reviews.  

Sample review is as following:

    "reviewerID": "A2SUAM1J3GNN3B",  
	"asin": "0000013714",  
	"reviewerName": "J. McDonald",  
	"helpful": [2, 3],  
	"reviewText": "I bought this for my husband who plays the piano.  He is having a wonderful time playing these old hymns.  The music  is at times hard to read because we think the book was published for singing from more than playing from.  Great purchase though!",  
	"overall": 5.0,  
	"summary": "Heavenly Highway Hymns",  
	"unixReviewTime": 1252800000,  
	"reviewTime": "09 13, 2009"  

### 1. Preparing Amazon dataset
First we load dataset from json file and rename column 'overall' to 'Rating'. 
```python
review_df = pd.read_json('Amazon_reviews/Clothing_Shoes_and_Jewelry_5.json', orient='records', lines=True)

# change column name 
review_df = review_df.rename(columns={'overall': 'Rating'})
```
There are 278677 records in the Clothing and Shoes dataset. 

### 2. Preliminary Analysis
* __Summary of the dataset:__
```
Number of reviews:  278677  

Number of unique reviewers:  39387  
Prop of unique reviewers:  0.141  

Number of unique products:  23033  
Prop of unique products:  0.083  

Average rating score:  4.245  
```
* __Distribution of rating score__

![rating-fr](https://github.com/anxin16/Capstone-Project-3/blob/master/Figures/rating-fr.png)

* __Distribution of rating propotion__

![rating-pro](https://github.com/anxin16/Capstone-Project-3/blob/master/Figures/rating-pro1.png)

* __Subset data__

Because text analysis is very time cosumming, we just use subset of the dataset to go through our process of text normalization, feature engineering and machine learning. When all are set, we can use whole dataset or other bigger dataset to get better model.

So we select reviews before 2012 as our sub dataset. There are 16434 records in it. 

Distribution of the ratings is as following:
```
Rating
1      690
2      861
3     1518
4     3363
5    10002
```

## III. Pre-processing —— Text Normalization
For natural language processing (NLP) and text analytics, pre-processing is very important. To carry out different operations and analyze text, we need to process and parse textual data into more easy-to-interpret formats. All machine learning (ML) algorithms usually work with input features that are numeric in nature. To get to that, we need to clean, normalize, and pre-process the initial textual data. 

Usually text corpora and other textual data in their native raw format are not well formatted and standardized. And text data is highly unstructured! Text pre-processing, involves using a variety of techniques to convert raw text into well-defined sequences of linguistic components that have standard structure and notation.

Text normalization is defined as a process that consists of a series of steps that should be followed to wrangle, clean, and standardize textual data into a form that could be consumed by other NLP and analytics systems and applications as input. Besides tokenization, various other techniques include cleaning text, case conversion, correcting spellings, removing stopwords and other unnecessary terms, stemming, and lemmatization. Text normalization is also often called text cleansing or wrangling.

Below are various techniques used in the process of text normalization:

* Cleaning Text
* Tokenizing Text
* Removing Special Characters
* Expanding Contractions
* Case Conversions
* Removing Stopwords
* Correcting Words
* Stemming
* Lemmatization

We will use most of the techniques in this project.

### 1. Expanding Contractions
Contractions are shortened version of words or syllables. They exist in either written or spoken forms. Shortened versions of existing words are created by removing specific letters and sounds. In case of English contractions, they are often created by removing one of the vowels from the word.

By nature, contractions do pose a problem for NLP and text analytics because, to start with, we have a special apostrophe character in the word. Ideally, we can have a proper mapping for contractions and their corresponding expansions and then use it to expand all the contractions in our text. 
```python
from contractions import CONTRACTION_MAP

# Define function to expand contractions
def expand_contractions(text):
    contractions_pattern = re.compile('({})'.format('|'.join(CONTRACTION_MAP.keys())),flags=re.IGNORECASE|re.DOTALL)
    def expand_match(contraction):
        match = contraction.group(0)
        first_char = match[0]
        expanded_contraction = CONTRACTION_MAP.get(match)\
                        if CONTRACTION_MAP.get(match)\
                        else CONTRACTION_MAP.get(match.lower())
        expanded_contraction = first_char+expanded_contraction[1:]
        return expanded_contraction
    
    expanded_text = contractions_pattern.sub(expand_match, text)
    expanded_text = re.sub("'", "", expanded_text)
    return expanded_text
```

### 2. Removing Special Characters
One important task in text normalization involves removing unnecessary and special characters. These may be special symbols or even punctuation that occurs in sentences. This step is often performed before or after tokenization. The main reason for doing so is because often punctuation or special characters do not have much significance when we analyze the text and utilize it for extracting features or information based on NLP and ML.
```python
# Define the function to remove special characters
def remove_characters(text):
    text = text.strip()
    PATTERN = '[^a-zA-Z0-9 ]' # only extract alpha-numeric characters
    filtered_text = re.sub(PATTERN, '', text)
    return filtered_text
```

### 3. Tokenizing Text
Tokenization can be defined as the process of breaking down or splitting textual data into smaller meaningful components called tokens.

**Sentence tokenization** is the process of splitting a text corpus into sentences that act as the first level of tokens which the corpus is comprised of. This is also known as sentence segmentation , because we try to segment the text into meaningful sentences.

**Word tokenization** is the process of splitting or segmenting sentences into their constituent words. A sentence is a collection of words, and with tokenization we essentially split a sentence into a list of words that can be used to reconstruct the sentence.
```python
# Define the tokenization function
def tokenize_text(text):
    word_tokens = nltk.word_tokenize(text)
    tokens = [token.strip() for token in word_tokens]
    return tokens
```

### 4. Removing Stopwords
Stopwords are words that have little or no significance. They are usually removed from text during processing so as to retain words having maximum significance and context. Stopwords are usually words that end up occurring the most if you aggregated any corpus of text based on singular tokens and checked their frequencies. Words like a, the , me , and so on are stopwords.
```python
from nltk.corpus import stopwords
# In Python, searching a set is much faster than searching a list, 
# so convert the stop words to a set
stopword_list = set(stopwords.words("english"))

# Define function to remove stopwords
def remove_stopwords(tokens):
    filtered_tokens = [token for token in tokens if token not in stopword_list]
    return filtered_tokens
```

### 5. Correcting Words
One of the main challenges faced in text normalization is the presence of incorrect words in the text. The definition of incorrect here covers words that have spelling mistakes as well as words with several letters repeated that do not contribute much to its overall significance.

**5.1 Correcting Repeating Characters**
```python
from nltk.corpus import wordnet

# Define function to remove repeated characters
def remove_repeated_characters(tokens):
    repeat_pattern = re.compile(r'(\w*)(\w)\2(\w*)')
    match_substitution = r'\1\2\3'
    def replace(old_word):
        if wordnet.synsets(old_word):
            return old_word
        new_word = repeat_pattern.sub(match_substitution, old_word)
        return replace(new_word) if new_word != old_word else new_word

    correct_tokens = [replace(word) for word in tokens]
    return correct_tokens
```

**5.2 Correcting Spellings**
```python
from collections import Counter

# Generate a map of frequently occurring words in English and their counts
"""
The input corpus we use is a file containing several books from the Gutenberg corpus and also 
a list of most frequent words from Wiktionary and the British National Corpus. You can find 
the file under the name big.txt or download it from http://norvig.com/big.txt and use it.
"""
def tokens(text):
    """
    Get all words from the corpus
    """
    return re.findall('[a-z]+', text.lower())

WORDS = tokens(open('big.txt').read())
WORD_COUNTS = Counter(WORDS)
```
```python
# Define functions that compute sets of words that are one and two edits away from input word.
def edits1(word):
    "All edits that are one edit away from `word`."
    letters    = 'abcdefghijklmnopqrstuvwxyz'
    splits     = [(word[:i], word[i:])    for i in range(len(word) + 1)]
    deletes    = [L + R[1:]               for L, R in splits if R]
    transposes = [L + R[1] + R[0] + R[2:] for L, R in splits if len(R)>1]
    replaces   = [L + c + R[1:]           for L, R in splits if R for c in letters]
    inserts    = [L + c + R               for L, R in splits for c in letters]
    return set(deletes + transposes + replaces + inserts)

def edits2(word): 
    "All edits that are two edits away from `word`."
    return (e2 for e1 in edits1(word) for e2 in edits1(e1))
```
```python
# Define function that returns a subset of words from our candidate set of words obtained from 
# the edit functions, based on whether they occur in our vocabulary dictionary WORD_COUNTS.
# This gives us a list of valid words from our set of candidate words.
def known(words): 
    "The subset of `words` that appear in the dictionary of WORD_COUNTS."
    return set(w for w in words if w in WORD_COUNTS)
```
```python
# Define function to correct words
def correct(words):
    # Get the best correct spellings for the input words
    def candidates(word): 
        # Generate possible spelling corrections for word.
        # Priority is for edit distance 0, then 1, then 2, else defaults to the input word itself.
        candidates = known([word]) or known(edits1(word)) or known(edits2(word)) or [word]
        return candidates
    
    corrected_words = [max(candidates(word), key=WORD_COUNTS.get) for word in words]
    return corrected_words
```

### 6. Lemmatization
The process of lemmatization is to remove word affixes to get to a base form of the word. The base form is also known as the root word, or the lemma, will always be present in the dictionary.
```python
import spacy
nlp = spacy.load("en")

# Define function for Lemmatization
def Lemmatize_tokens(tokens):
    doc = ' '.join(tokens)
    Lemmatized_tokens = [token.lemma_ for token in nlp(doc)]
    return Lemmatized_tokens
```

### 7. Text Normalization
```python
def normalize_corpus(corpus):
    normalized_corpus = []    
    for text in corpus:
        text = text.lower()
        text = expand_contractions(text)
        text = remove_characters(text)
        tokens = tokenize_text(text)
        tokens = remove_stopwords(tokens)
        tokens = remove_repeated_characters(tokens)
        tokens = correct(tokens)
        tokens = Lemmatize_tokens(tokens)
        text = ' '.join(tokens)
        normalized_corpus.append(text)
                    
    return normalized_corpus
```

After above steps, we get normalized texts of Amazon reviews. 

## IV. Feature Engineering —— Feature Extraction 
**Feature engineering** is the process of using domain knowledge of the data to create features that make machine learning algorithms work. Feature engineering is fundamental to the application of machine learning, and is both difficult and expensive. 

In ML terminology, features are unique, measurable attributes or properties for each observation or data point in a dataset. Features are usually numeric in nature and can be absolute numeric values or categorical features that can be encoded as binary features for each category in the list using a process called one-hot encoding. The process of extracting and selecting features is both art and science, and this process is called *feature extraction* or *feature engineering*.

The *Vector Space Model* is a concept and model that is very useful in case we are dealing with textual data and is very popular in information retrieval and document ranking. The Vector Space Model, also known as the *Term Vector Model*, is defined as a mathematical and algebraic model for transforming and representing text documents as numeric vectors of specific terms that form the vector dimensions.

We will be implementing the following feature-extraction techniques in this project:  
* Bag of Words model  
* TF-IDF model  
* Averaged Word Vectors  
* TF-IDF Weighted Averaged Word Vectors   

### 1. Bag of Words Model
The Bag of Words model is perhaps one of the simplest yet most powerful techniques to extract features from text documents. The essence of this model is to convert text documents into vectors such that each document is converted into a vector that
represents the frequency of all the distinct words that are present in the document vector space for that specific document. 
```python
from sklearn.feature_extraction.text import CountVectorizer

def bow_extractor(corpus, ngram_range=(1,1)):
    vectorizer = CountVectorizer(min_df=1, ngram_range=ngram_range, max_features = 5000)
    features = vectorizer.fit_transform(corpus)
    return vectorizer, features
```

### 2. TF-IDF Model
TF-IDF stands for Term Frequency-Inverse Document Frequency, a combination of two metrics: *term frequency* and *inverse document frequency*.

Mathematically, TF-IDF is the product of two metrics and can be represented as $tfidf = tfxidf$, where *term frequency*(tf) and *inverse-document frequency*(idf) represent the two metrics.
```python
from sklearn.feature_extraction.text import TfidfVectorizer

# Define function to directly compute the tfidf-based feature vectors for documents from the raw documents.
def tfidf_extractor(corpus, ngram_range=(1,1)):
    vectorizer = TfidfVectorizer(min_df=1,
                                 norm='l2',
                                 smooth_idf=True,
                                 use_idf=True,
                                 ngram_range=ngram_range,
                                 max_features = 5000)
    features = vectorizer.fit_transform(corpus)
    return vectorizer, features
```

### 3. Averaged Word Vectors
In this technique, we will use an average weighted word vectorization scheme, where for each text document we will extract all the tokens of the text document, and for each token in the document we will capture the subsequent word vector if present in the vocabulary. We will sum up all the word vectors and divide the result by the total number of words matched in the vocabulary to get a final resulting averaged word vector representation for the text document.
```python
import numpy as np    

# Define function to average word vectors for a text document
def average_word_vectors(words, model, vocabulary, num_features):
    
    feature_vector = np.zeros((num_features,),dtype="float64")
    nwords = 0.
    
    for word in words:
        if word in vocabulary: 
            nwords = nwords + 1.
            feature_vector = np.add(feature_vector, model[word])
    
    if nwords:
        feature_vector = np.divide(feature_vector, nwords)
        
    return feature_vector
```
```python
# Generalize above function for a corpus of documents  
def averaged_word_vectorizer(corpus, model, num_features):
    vocabulary = set(model.wv.index2word)
    features = [average_word_vectors(sentence, model, vocabulary, num_features) for sentence in corpus]
    return np.array(features)
```

### 4. TF-IDF Weighted Averaged Word Vectors
Our previous vectorizer simply sums up all the word vectors pertaining to any document based on the words in the model vocabulary and calculates a simple average by dividing with the count of matched words. Now we use a new and novel technique of weighing each matched word vector with the word TF-TDF score and summing up all the word vectors for a document and dividing it by the sum of all the TF-IDF weights of the matched words in the document. This would basically give us a TF-IDF weighted averaged word vector for each document.
```python
# Define function to compute tfidf weighted averaged word vector for a document
def tfidf_wtd_avg_word_vectors(words, tfidf_vector, tfidf_vocabulary, model, num_features):
    
    word_tfidfs = [tfidf_vector[0, tfidf_vocabulary.get(word)] 
                   if tfidf_vocabulary.get(word) 
                   else 0 for word in words]    
    word_tfidf_map = {word:tfidf_val for word, tfidf_val in zip(words, word_tfidfs)}
    
    feature_vector = np.zeros((num_features,),dtype="float64")
    vocabulary = set(model.wv.index2word)
    wts = 0.
    for word in words:
        if word in vocabulary: 
            word_vector = model[word]
            weighted_word_vector = word_tfidf_map[word] * word_vector
            wts = wts + word_tfidf_map[word]
            feature_vector = np.add(feature_vector, weighted_word_vector)
    if wts:
        feature_vector = np.divide(feature_vector, wts)
        
    return feature_vector
```
```python
# Generalize above function for a corpus of documents
def tfidf_weighted_averaged_word_vectorizer(corpus, tfidf_vectors, 
                                   tfidf_vocabulary, model, num_features):
                                       
    docs_tfidfs = [(doc, doc_tfidf) 
                   for doc, doc_tfidf 
                   in zip(corpus, tfidf_vectors)]
    features = [tfidf_wtd_avg_word_vectors(tokenized_sentence, tfidf, tfidf_vocabulary,
                                   model, num_features)
                    for tokenized_sentence, tfidf in docs_tfidfs]
    return np.array(features) 
```

```python

```

## V. APPROACH
1. Data preparation
2. Data Wrangling
3. Descriptive analysis
4. Feature engineering
5. Modeling and Machine learning
6. Data Story

## VI. DELIVERABLES
Code, paper, slides
