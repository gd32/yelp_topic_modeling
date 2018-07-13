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

![Useful histogram](https://github.com/gd32/DSI_capstone/blob/master/visuals/useful_dist.png)

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

![Funny useful](https://github.com/gd32/DSI_capstone/blob/master/visuals/funny_useful.png)

![Cool useful](https://github.com/gd32/DSI_capstone/blob/master/visuals/cool_useful.png)

## Topic Modeling

Text features were incorporated into models using Latent Semantic Analysis (LSA). For each review, text was processed and tokenized using Spacy. Each document (a list containing each non-stop word for each review in tokenized form) was then passed into a term-frequency/inverse-document frequency (tf-idf) vectorizer, resulting in a document-term matrix containing the tf-idf frequency of the top 10,000 features across all documents in the corpus of documents. The resulting document-term matrix was then transformed into a document-topic matrix using textacy's LSA topic model (built on sk-learn's truncated SVD module).

A sample of the top 20 topics and top 10 terms for the business dataset is show in the below termite plot:

![Business termite](https://github.com/gd32/DSI_capstone/blob/master/visuals/business_termite.png)

High level review features and the topic features were combined into one feature matrix, resulting in 228 features for restaurants and 378 features for businesses.

## Modeling

From the review metadata, I used the review's stars, the number of funny votes, and the word count of each review in addition to the Yelp-derived categories for each business each review was written for.\

I propose creating a validation set of our data using the following scheme:

 - Reviews with no useful votes will be classified as not useful (0).
 - Reviews with either 1 or 2 useful votes will be held out as a validation set. 
 - Reviews with at least 3 useful votes will be classified as useful (1).
 
This approach intends to control for the unknown factor of page/click count influencing the number of votes a review could get, in addition to the algorithm Yelp uses to filter and display reviews on a specific businesses' page. Following the scheme, we believe that having at least three individuals tag a review as 'Useful' is representative of a consensus, whereas reviews with only 1 or 2 votes may or may not be useful. 

For the businesses data, a logistic regression model was trained that classifies Yelp reviews of non-restaurant businesses as useful/not useful with 82.1% accuracy. 

A similar model trained on restaurant reviews classifies such reviews as useful/not useful with *** accuracy.

I also trained a random forest classifier on the businesses dataset which achieved a very similar accuracy score (81.9%). However, this model took approximately 5 times longer to fit than the corresponding logistic regression model (2.5 hours vs 30 mins for the logistic regression).

Evaluation of the logistic regression  model on the held-out non-restaurant businesses suggests that the model identifies short reviews (<4 words) that express one single topic as 'not useful'. However, simple statements that specifically refer to a businesses' products were incorrectly classified.

All statistical analysis was performed on a t2.2xlarge AWS instance.

## Future Steps

**Topic coherence** - While the model using LSA-generated topics as features did quite well at classifying reviews, the topics are relatively weak in terms of human coherence. Latent Dirichlet Allocation (LDA) has been known to generate topics that are both coherent and beneficial in classification problems. Topic coherence can be quantified and compared using various metrics (i.e. UMass, UCI coherence) contained in gensim's coherenceModel.

**Additional classifiers** - Additional, more computationally/time-intensive classifiers could be trained. Of note, support vector classifiers have been shown to perform well in classification tasks utilizing topic model-generated features.

**Generalization** - The ultimate goal for this topic-based strategy would be to implement such a model that could predict usefulness for unlabeled text reviews to aid business strategy or operation on any review-based platform. 
