
# Evaluation metric

references:
 - https://neptune.ai/blog/recommender-systems-metrics
 - https://towardsdatascience.com/evaluation-metrics-for-recommendation-systems-an-overview-71290690ecba

## collaborative filtering vs content-based 
collaborative filtering
- memory-based: item-item or user-user iteractions; similarities and nearest neighbor algorithms
- model-based:
  - generative model that explains user-item interactions;
  - matrix factorization algorithms decompose user-item matrix into user-factor and item factor; association rules, clustering algorithms, deep neural networks
- hybrid: content-based and collaborative filtering

content-based
- item-centered: recommend new items based on previous items
- user-centered: collect user preference (prev items and/or questionnaire),  recommend items with similar features


## similarity measure
- Cosine similarity
- Jaccard similarity
- Euclidean distance
- Pearson correlation coefficient


## Evaluation Metrics

### Classification metrics
- Precision@K
- Recall@K
- F1@K

### Ranking-based metrics
- Mean Average Precision (MAP)@K
- Normalized Discounted Cumulative Gain (NDCG)
- Mean Reciprocal Rank (MRR)

### Predictive metrics:
- Mean Absolute Error (MAE): average matinitude of differences in ratings
- Root Mean Squared Error (RMSE): 

### Recommendation-centric metrics
- Diversity
- Coverage

### User-centric metrics
- Novelty
- Trustworthiness
- Churn and responsiveness

### Business meetrics
- click-through rates
- adoption and conversion
- sales and revenue
- effects on sales distributions
- user engagement and behavior


## Things to consider
 - Popularity Bias
 - Position Bias - items placed higher more likely to be consumed irrespective of the actual relevance
 - Degenerate Feedback loop - when users are limited to interacting with suggested items, negative feedback loop can emerge



# Approach



## How YouTube does it [https://dl.acm.org/doi/10.1145/2959100.2959190]

@TODO: summarize here with diagrams and tech details

## [https://eugeneyan.com/writing/system-design-for-discovery/] 

### Common paradigm

Two steps:
1. Candidate retrieval (fast; coarse)
  - convert input into embedding for Approximate Nearest Neighbors (ANN)
  - could also be done using graphs (DoorDash) and decision trees (LinkedIn)
2. Ranking (slower; precise)
  - more precise raking with added user item, user, and contextual features
  - can be modeled as learning-to-rank or classification


#### Offline
- Retrieval:
  - train embedding model
  - build approximate NN index
  - Embed items from catalog

- Ranking:
  - build feature store (item, user)
  - train ranking model

#### Online
- Retrieval:
  - embed input item or query
  - retrieve top k candidates
- Ranking:
  - add features to candidates
  - rank top k candidates



### Alibaba [https://arxiv.org/abs/1803.02349] - item embeddings for candidate retrieval

#### Train: i2i similarity embedding
user-item interactions -> weighted bidirectional item graph
random walk on the graph -> item sequences -(word2vec skip-gram)-> item embeddings

#### Infer: retrieval -> rank
fetch latest items user interacted with
retrieve candidates from i2i similarity
rank (deep neural network)

### Facebook [https://arxiv.org/abs/2006.11632] - embedding-based retrieval for search

#### Train: two-tower network (query encoder and document encoder)
cosine similarity for each query-document pair

#### Infer: query -> query embedding -> retrieval -> ranking

### JD [https://arxiv.org/abs/2006.02282] - semantic retieval for search; similar to Facebook


### Doordash [https://doordash.engineering/2020/12/15/understanding-search-intent-with-better-recall/] - knowledge graph for query expansion and retrieval

#### Train: query understanding, query expansion, ranking models
elastic search for documents (restaurants and food items) and feature store for attribute data (ratings, price points, tags)
#### Infer: retrieval and rerank
retrieval: query standardization, synonymization, query expansion (knowledge graph), retrieval
rerank: lexical similarity between query and documents, store popularity, search context


## YouTube's video recommendation system [https://dl.acm.org/doi/pdf/10.1145/2959100.2959190]

### Overview
 - candidate generation (millions -> hundreds):
   - video corpus
   - user history and context
 - ranking (hudreds -> dozens)
   - candidates
   - user history and context
   - other candidate sources
   - video features


### Candidate generation
- Recommendataion as classification: extreme mlticlass classification

classifying a video watch Wt at time t among millions of videos i (classes) from a corpus V
based on a user U and context C:
P(wt=i|U,C) = e^(v_i*u) / sigma over j in V [ e^(v_j*u) ]

To speed up the softmax for millions of classes, several thousand negative classes are sampled (100x speed up)

Once trained, the scoring problem reduces to a nearest neighbor search in the dot product space (no softmax needed)


video vector (avg of video watches) / search vector (avg of search tokens) / geographic embedding / example age / gender /
|
V
ReLU
|
V
..
|
V
Softmax / nearest neighbor lookup (user <> video)




## Spark [https://spark.apache.org/docs/latest/ml-collaborative-filtering.html]

"Collaborative filtering is commonly used for recommender systems. These techniques aim to fill in the missing entries of a user-item association matrix. spark.ml currently supports model-based collaborative filtering, in which users and products are described by a small set of latent factors that can be used to predict missing entries. spark.ml uses the alternating least squares (ALS) algorithm to learn these latent factors."

Alternating Least Squares:
https://dl.acm.org/doi/10.1109/MC.2009.263

Yehuda Koren, Robert Bell, and Chris Volinsky. 2009. Matrix Factorization Techniques for Recommender Systems. Computer 42, 8 (August 2009), 30–37. https://doi.org/10.1109/MC.2009.263

improves upon classical nearest-neighbor techniques using additional information such as implicit feedback, temporal effects, and confidence levels


```
numBlocks is the number of blocks the users and items will be partitioned into in order to parallelize computation (defaults to 10).
rank is the number of latent factors in the model (defaults to 10).
maxIter is the maximum number of iterations to run (defaults to 10).
regParam specifies the regularization parameter in ALS (defaults to 1.0).
implicitPrefs specifies whether to use the explicit feedback ALS variant or one adapted for implicit feedback data (defaults to false which means using explicit feedback).
alpha is a parameter applicable to the implicit feedback variant of ALS that governs the baseline confidence in preference observations (defaults to 1.0).
nonnegative specifies whether or not to use nonnegative constraints for least squares (defaults to false).
```

```
from pyspark.ml.evaluation import RegressionEvaluator
from pyspark.ml.recommendation import ALS
from pyspark.sql import Row

lines = spark.read.text("data/mllib/als/sample_movielens_ratings.txt").rdd
parts = lines.map(lambda row: row.value.split("::"))
ratingsRDD = parts.map(lambda p: Row(userId=int(p[0]),
                                     movieId=int(p[1]),
                                     rating=float(p[2]),
                                     timestamp=int(p[3])))
ratings = spark.createDataFrame(ratingsRDD)
(training, test) = ratings.randomSplit([0.8, 0.2])

# Build the recommendation model using ALS on the training data
# Note we set cold start strategy to 'drop' to ensure we don't get NaN evaluation metrics
als = ALS(maxIter=5,
        regParam=0.01,
        userCol="userId",
        itemCol="movieId",
        ratingCol="rating",
        coldStartStrategy="drop")
model = als.fit(training)

# Evaluate the model by computing the RMSE on the test data
predictions = model.transform(test)
evaluator = RegressionEvaluator(metricName="rmse",
                                labelCol="rating",
                                predictionCol="prediction")
rmse = evaluator.evaluate(predictions)
print("Root-mean-square error = " + str(rmse))

# Generate top 10 movie recommendations for each user
userRecs = model.recommendForAllUsers(10)
# Generate top 10 user recommendations for each movie
movieRecs = model.recommendForAllItems(10)

# Generate top 10 movie recommendations for a specified set of users
users = ratings.select(als.getUserCol()).distinct().limit(3)
userSubsetRecs = model.recommendForUserSubset(users, 10)
# Generate top 10 user recommendations for a specified set of movies
movies = ratings.select(als.getItemCol()).distinct().limit(3)
movieSubSetRecs = model.recommendForItemSubset(movies, 10)
```
