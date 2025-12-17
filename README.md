# Upskills

1. NYC TLC Taxi Trips (yellow_tripdata)

Core role: large-scale tabular + time-series + join-heavy dataset

Operations you can perform

Ingestion & structure

Parquet loading

Schema inspection

Datetime parsing

Type coercion

Index inspection

Sampling / chunked reads

Data quality

Missing value detection

Duplicate row detection

Duplicate event detection

Range validation (fare, distance)

Impossible value detection

Cardinality inspection

Class imbalance analysis

Joining & integration

Left / right / inner / outer joins (with zone lookup)

One-to-many joins

Join validation

Join indicator analysis

Row explosion detection

Post-join null inflation checks

Time-aware operations

Time-based filtering

Time-based splitting

Lag feature creation

Rolling window aggregation

Temporal leakage detection

As-of joins (with weather)

EDA

Univariate statistics

Bivariate analysis

Correlation analysis

Group-by aggregation

Target-free exploratory analysis

Time-series visualization

Transformations

Scaling numeric features

Log transforms (fare)

Binning (distance, duration)

Feature engineering

Trip duration

Fare per mile

Time-of-day features

Day-of-week features

Cumulative demand features

Validation

Temporal ordering checks

Cross-field consistency checks

2. Taxi Zone Lookup

Core role: join-enrichment & relational integrity practice

Operations you can perform

Joining

Left joins

Inner joins

Outer joins

Orphan key detection

Join cardinality validation

Data quality

Categorical consistency checks

Duplicate key detection

Cardinality validation

EDA

Frequency analysis of zones

Group-by aggregation post-join

Feature engineering

Zone-level aggregations

Borough-level grouping features

3. NOAA Weather (Hourly, NYC station)

Core role: time-aware enrichment & leakage detection

Operations you can perform

Ingestion

CSV loading

Datetime parsing

Unit normalization

Joining

Exact timestamp joins (failure case)

As-of joins

Nearest-key joins

Tolerance-based joins

Forward/backward joins

Time-series operations

Lag features

Rolling averages

Smoothing

Temporal alignment

EDA

Weather variable distributions

Seasonal pattern detection

Correlation with taxi demand

Validation

Causality checks

Leakage detection

4. Titanic Dataset

Core role: missing data, categorical encoding, classification EDA

Operations you can perform

Data quality

Missing value analysis

Missingness pattern detection

Duplicate detection

Missing data handling

Column dropping

Numeric imputation

Categorical imputation

Missing indicator creation

EDA

Univariate and bivariate analysis

Target vs feature analysis

Group-by survival analysis

Encoding

Label encoding

One-hot encoding

Ordinal encoding

Feature selection

Correlation-based pruning

Univariate statistical tests

Splitting

Stratified train/test split

Target analysis

Class imbalance analysis

Leakage detection

5. Ames Housing

Core role: regression, heavy feature engineering, transformation

Operations you can perform

Data quality

Outlier detection

Range validation

Cardinality inspection

EDA

Distribution analysis

Target-feature relationships

Multivariate correlation

Transformations

Log / Box-Cox / Yeo-Johnson

Standardization

Normalization

Feature engineering

Interaction features

Polynomial features

Aggregations

Domain-specific features

Encoding

One-hot encoding

Frequency encoding

Target encoding (with care)

Feature selection

Variance thresholding

PCA / SVD

Model-based selection

Target analysis

Target transformation

Outlier handling

6. TMDB Movies (JSON-based)

Core role: nested data, many-to-many joins, text features

Operations you can perform

Ingestion

JSON loading

Schema normalization

Exploding nested fields

Joining

Many-to-many joins

Composite key joins

Join explosion analysis

EDA

Frequency analysis

Cross-tabulation

Text preprocessing

Text cleaning

Tokenization

Vectorization (TF-IDF / embeddings)

Feature engineering

Cast count

Genre flags

Runtime buckets

Validation

Duplicate detection

Cardinality checks

7. Amazon Reviews

Core role: NLP pipelines & high-dimensional data

Operations you can perform

Data quality

Missing text detection

Duplicate review detection

Text preprocessing

Cleaning

Stopword removal

Tokenization

Lemmatization

Feature engineering

TF-IDF

Embeddings

Sentiment proxies

EDA

Review length distribution

Rating distribution

Text-feature correlations

Dimensionality reduction

SVD

PCA on embeddings

Splitting

Time-aware split (review date)

Group-based split (product/user)

8. MNIST

Core role: image preprocessing & feature extraction

Operations you can perform

Ingestion

Array loading

Shape validation

Transformations

Normalization

Scaling

Reshaping

Feature engineering

Pixel flattening

PCA for compression

Feature extraction pipelines

EDA

Image visualization

Pixel intensity distribution

Splitting

Train/test split

Validation

Data leakage checks