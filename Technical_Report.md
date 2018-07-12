# Technical Report - Yelp Review Classification

## Problem Statement
[Yelp](http://www.yelp.com) is a crowd-sourced review platform that allows users to provide ratings and text-based reviews for local businesses. [With around 175 million monthly users](https://www.yelp.com/factsheet), Yelp has grown to be a powerhouse in the online community for users to pick restaurants and businesses they wish to patronize. Users can choose to 'thumbs up' reviews in three categories: 'Useful', 'Funny', and 'Cool'. In addition to helping users find reviews that appeal to them, these categories, especially the 'Useful' category, can help key businesses owners into user preferences; increased counts of useful votes for a review should suggest to owners that users have a consensus opinion about some aspect of the business.  

With these facts in mind, my goal for this project was to build a classification model that could predict if a written Yelp review wwould be considered 'Useful' by Yelp users. Providing businesses with the ability to predict usefulness of reviews would not only serve as a way to validate their business practices by identifying what aspects of their business are appealing to users, but would also allow businesses to promote their appeal by highlighting useful reviews on their Yelp page.

## Data Collection
I obtained the reviews and business data directly from Yelp (dataset found [here](https://www.yelp.com/dataset/challenge) or on [Kaggle](https://www.kaggle.com/yelp-dataset/yelp-dataset) and stored them on a PostgreSQL server hosted on an AWS t2.micro instance.

The original review dataset provided by Yelp contained reviews from the following metropolitan regions: Edinburgh (UK), Stuttgart (Germany), Montreal (Canada), Toronto (Canada), Pittsburgh , Charlotte, Champaign-Urbana, Phoenix, Las Vegas, Madison, and Cleveland. For this project, I chose to restrict my training data to only reviews written in English; as a result I excluded businesses in Edinburgh, Stuttgart, and Montreal in an effort to eliminate the main sources of reviews written in a non-English language. This resulted in a review dataset of approximately **5 million** text reviews and their accompanying metadata.

All statistical analysis was performed on a t2.2xlarge AWS instance.

## Feature Selection 
From the review metadata, I used the review's stars, the number of funny votes, and the word count of each review in addition to the Yelp-derived categories for each business each review was written for.

Text features were incorporated into models using Latent Semantic Analysis (LSA). For each review, text was processed and tokenized using Spacy. Each document (a list containing each non-stop word for each review in tokenized form) was then passed into a term-frequency/inverse-document frequency (tf-idf) vectorizer, resulting in a document-term matrix containing the tf-idf frequency of the top 10,000 features across all documents in the corpus of documents. The resulting document-term matrix was then transformed into a document-topic matrix using textacy's LSA topic model (built on sk-learn's truncated SVD module).

High level review features and the topic features were combined into one feature matrix, resulting in 228 features for restaurants and 378 features for businesses.

## Modeling

For the businesses data, a logistic regression model was trained that classifies Yelp reviews of non-restaurant businesses as useful/not useful with 82.1% accuracy. 

A similar model trained on restaurant reviews classifies such reviews as useful/not useful with *** accuracy.

I also trained a random forest classifier on the businesses dataset which achieved a very similar accuracy score (81.9%). However, this model took approximately 5 times longer to fit than the corresponding logistic regression model (2.5 hours vs 30 mins for the logistic regression).

Evaluation of the logistic regression  model on the held-out non-restaurant businesses suggests that the model identifies short reviews (<4 words) that express one single topic as 'not useful'. However, simple statements that specifically refer to a businesses' products were incorrectly classified.

## Future Steps

**Topic coherence** - While the model using LSA-generated topics as features did quite well at classifying reviews, the topics are relatively weak in terms of human coherence. Latent Dirichlet Allocation (LDA) has been known to generate topics that are both coherent and beneficial in classification problems. Topic coherence can be quantified and compared using various metrics (i.e. UMass, UCI coherence) contained in gensim's coherenceModel.

**Additional classifiers** - Additional, more computationally/time-intensive classifiers could be trained. Of note, support vector classifiers have been shown to perform well in classification tasks utilizing topic model-generated features.

**Generalization** - The ultimate goal for this topic-based strategy would be to implement such a model that could predict usefulness for unlabeled text reviews to aid business strategy or operation on any review-based platform. 
