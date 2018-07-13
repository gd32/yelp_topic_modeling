# Technical Report - Yelp Review Classification

## Problem Statement
[Yelp](http://www.yelp.com) is a crowd-sourced review platform that allows users to provide ratings and text-based reviews for local businesses. [With around 175 million monthly users](https://www.yelp.com/factsheet), Yelp has grown to be a resource not only for users searching for businesses or restaruants to patronize, but also for businesses to actively collect and digest user ratings and reviews and use that data to improve their products or services.

Unlike other social media platforms, Yelp divides user-provided positive feedback into three categories; users can choose to 'thumbs up' reviews written by their peers in three categories: 'Useful', 'Funny', and 'Cool'. These categories, especially the 'Useful' category, can help key businesses owners into user preferences; the presence of useful votes for a review should suggest to owners that users share opinions made in that review.

With these facts in mind, my goal for this project was to build a classification model that could predict if a written Yelp review wwould be considered 'Useful' by other Yelp users. Providing businesses with the ability to predict usefulness of reviews would not only serve as a way to validate their business practices by identifying what aspects of their business are appealing to users, but would also allow businesses to promote their appeal by highlighting useful reviews on promotional materials or on other social media platforms such as Facebook or Twitter.

## Data Collection
I obtained the reviews and business data directly from Yelp, which provides a random sample of review, user, and business data for academic or research use. The dataset can be found [here](https://www.yelp.com/dataset/challenge) or on [Kaggle](https://www.kaggle.com/yelp-dataset/yelp-dataset).

Data was obtained in .csv format and added to a PostgreSQL database hosted on a t2.micro AWS EC2 instance.

## Exploratory Data Analysis

The dataset contained 5.2 million reviews from 12 metropolitan regions across 4 countries.  

The original review dataset provided by Yelp contained reviews from the following metropolitan regions: Edinburgh (UK), Stuttgart (Germany), Montreal (Canada), Toronto (Canada), Pittsburgh , Charlotte, Champaign-Urbana, Phoenix, Las Vegas, Madison, and Cleveland. For this project, I chose to restrict my training data to only reviews written in English; as a result I excluded businesses in Edinburgh, Stuttgart, and Montreal in an effort to eliminate the main sources of reviews written in a non-English language. 

The distribution of useful votes is described in the histogram below:

![Useful dist](https://github.com/gd32/DSI_capstone/blob/master/visuals/useful_dist.png)
---
### Useful Votes by Category

![Useful categories](https://github.com/gd32/DSI_capstone/blob/master/visuals/useful_stats.png)

As expected, restaurants dominate the review space on Yelp and thus restaurant reviews have a majority of the useful votes; while other business categories such as religious organizations or educational businesses had a higher average number of useful votes, the number of total votes was much lower for those categories.

In comparing reviews with and without useful votes, I found that useful reviews were overall slightly longer than not useful reviews.
In both useful and not useful reviews, positive review (4 or 5 star reviews) were in general shorter than negative reviews.

![Useful stars](https://github.com/gd32/DSI_capstone/blob/master/visuals/useful_stars.png)

![Not useful](https://github.com/gd32/DSI_capstone/blob/master/visuals/notuseful_stars.png)
---
### Comparing Vote Categories

In comparing useful and funny votes, 3 vote patterns are apparent:

1. Reviews with a high number of funny votes, but very few useful votes in comparison.
2. Votes with an equal number of funny and useful votes (suggesting there is a linear relationship between funny and useful votes up to a certain point)
3. Votes with a high number of useful votes, but very few funny votes.

![Funny useful](https://github.com/gd32/DSI_capstone/blob/master/visuals/funny_useful.png)

In contrast to the useful/funny comparison, the useful/cool comparison only shows two groups of reviews. A group with equal amounts of useful and cool votes up to around 250 votes, and reviews with a high number of useful votes but no cool votes. We see that there are no reviews with many cool votes, but no useful votes.

![Cool useful](https://github.com/gd32/DSI_capstone/blob/master/visuals/cool_useful.png)

## NLP and Topic Modeling
 
The review text was prepared for modeling using Spacy and textacy in three stages: preprocessing, tokenization, and vectorization (see below flowchart for example). In the preprocessing stage, reviews were converted to all lowercase and all numbers, URLs, and punctuation marks were removed. In the tokenization stage, reviews were passed through a token filter to remove stop words and short tokens and converted into *documents* which consist of lists of tokens that passed the filter. Lastly, each document was then passed into a term-frequency/inverse-document frequency (tf-idf) vectorizer, resulting in a document-term sparse matrix containing the tf-idf frequency of the top 10,000 features across all documents in the corpus of documents. 

![NLP Flowchart](https://github.com/gd32/DSI_capstone/blob/master/visuals/NLPflow.png)

Topic modeling was performed using Latent Semantic Analysis (LSA) as implemented in both scikit-learn and textacy; due to memory limitations only 325 topics could be generated from the business dataset and 200 topics for the restaurants dataset.

A sample of the top 20 topics and top 20 terms for the business dataset is show in the below termite plot:

![Business termite](https://github.com/gd32/DSI_capstone/blob/master/visuals/business_termite.png) 

A termite plot can be interpreted as follows: topics are on the x-axis at the top of the plot, with unique terms on the y-axis. Larger circles at a topic/term intersection mean that term had a heavier weight for that topic. Color coding shows the relative strength of terms across the highlighted topics; low numbers of highlighted terms for a topic indicate that the topic did not share many terms with the other selected topics.

## Modeling

All statistical analysis was performed on a t2.2xlarge AWS instance.

---
### Strategy

I defined the target of my classification models as follows: 

 - Reviews with no useful votes were labeled not useful (0).
 - Reviews with either 1 or 2 useful votes were held out as a validation set. 
 - Reviews with 3 or more useful votes were labeled as useful (1).
 
This approach intends to control for the unknown factor of page/click count influencing the number of votes a review could get, in addition to the proprietary algorithm Yelp uses to filter and sort reviews that are displayed on a specific businesses' page. My assumptions are that reviews with 3 or more useful votes represent some consensus among users, whereas having only one vote indicates only one opinion of usefulness and two votes could be the result of two people finidng a review useful for different reasons. Based on this reasoning, I chose to use my chosen model to predict the usefulness of the reviews with 1-2 votes and see what reviews of that subset were predicted as not useful.

Features used in modeling were a combination of review metadata and the topic weights for each document. From the review metadata, I used the review's number of stars, the number of funny votes, and the word count of each review in addition to boolean values for the business category (restaurant, active life, health and medical, etc.).

---
### Results

For the businesses data, a logistic regression model was trained that classifies Yelp reviews of non-restaurant businesses as useful or not useful with 82.1% accuracy. 

|             	| precision 	| recall 	| f1-score 	| support 	|
|-------------	|-----------	|--------	|----------	|---------	|
| Not useful  	| 0.82      	| 0.94   	| 0.88     	| 679113  	|
| Useful      	| 0.82      	| 0.55   	| 0.66     	| 308811  	|
| avg / total 	| 0.82      	| 0.82   	| 0.81     	| 987924  	|

Based on the above classification report, we can see that model has high specificity but low sensitivity, suggesting that our model is more pessimistic about reviews than users. Overall, short 4 or 5 star reviews were more likely to be classified as not useful, likely because they contain little information that can add to their topic weights.

A similar model trained on restaurant reviews classifies those reviews as useful/not useful with 89.6% accuracy.

I also trained a random forest classifier on the businesses dataset which achieved a very similar accuracy score (81.9%). However, this model took approximately 5 times longer to fit than the corresponding logistic regression model (2.5 hours vs 30 mins for the logistic regression).

## Future Steps

**Improve sensitivity** - Although the model performed well at detecting not useful reviews, it was lacking in its ability to detect truly useful reviews (sensitivity = 0.55). The source of this discrepancy is likely the presence of numerous short reviews which do not have strong topic weights due to their lack of content. It is possible that eliminating reviews shorter than 7 or 8 words could improve the sensitivity of the model. 

**Topic coherence** - While the model using LSA-generated topics as features did quite well at classifying reviews, the topics are relatively weak in terms of human coherence. Latent Dirichlet Allocation (LDA) has been known to generate topics that are both coherent and beneficial in classification problems. Topic coherence can be quantified and compared using various metrics (i.e. UMass, UCI coherence) contained in gensim's coherenceModel.

**Additional classifiers** - Additional, more computationally/time-intensive classifiers could be trained. Of note, support vector classifiers have been shown to perform well in classification tasks utilizing topic model weights.

