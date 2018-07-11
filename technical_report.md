# Technical Report - Yelp Review Classification

## Problem Statement
[Yelp](http://www.yelp.com) is a crowd-sourced review platform that allows users to provide ratings and text-based reviews for local businesses. [With around 175 million monthly users](https://www.yelp.com/factsheet), Yelp has grown to be a powerhouse in the online community for users to pick restaurants and businesses they wish to patronize. Users can choose to 'thumbs up' reviews in three categories: 'Useful', 'Funny', and 'Cool'. In addition to helping users find reviews that appeal to them, these categories, especially the 'Useful' category, can help key businesses owners into user preferences; increased counts of useful votes for a review should suggest to owners that users have a consensus opinion about some aspect of the business.  

With these facts in mind, my goal for this project was to build a classification model that could predict if a written Yelp review wwould be considered 'Useful' by Yelp users. Providing businesses with the ability to predict usefulness of reviews would not only serve as a way to validate their business practices by identifying what aspects of their business are appealing to users, but would also allow businesses to promote their appeal by highlighting useful reviews on their Yelp page.

## Data Collection
I obtained the reviews and business data directly from Yelp (dataset found [here](https://www.yelp.com/dataset/challenge)) and stored them on a PostgreSQL server hosted on an AWS t2.micro instance. Statistical analysis was performed on either a t2.xlarge or t2.2xlarge instance.

## Feature Selection 
From the review metadata, I retained the review's stars, the number of funny votes, and the word count of each review. I also used the Yelp-derived categories for each business each review was written for.

The original review dataset contained reviews from the following metropolitan regions: Edinburgh (UK), Stuttgart (Germany), Montreal (Canada), Toronto (Canada), Pittsburgh , Charlotte, Champaign-Urbana, Phoenix, Las Vegas, Madison, and Cleveland. For this project, I chose to restrict my training data to only reviews written in English; as a result I excluded businesses in Edinburgh, Stuttgart, and Montreal in an effort to eliminate the main sources of reviews written in a non-English language.

Text features were incorporated into models using Latent Semantic Analysis (LSA). For each review, text was processed and tokenized using Spacy. Each document (a list containing each non-stop word for each review in tokenized form) was then passed into a tf-idf vectorizer, resulting in a document-term matrix containing the tf-idf frequency of the top 10,000 features across all documents. The resulting document-term matrix was then transformed into a document-topic matrix using textacy's LSA topic model (built on sk-learn's truncated SVD module). The top 325 topics were chosen to be used as features in the classification models.

High level review features and the topic features were combined into one feature matrix, resulting in XXX features in total.

## Modeling




## Conclusions
