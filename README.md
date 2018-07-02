# Predicting Useful Yelp Reviews

What makes a useful Yelp review? What characteristics do reviews have that make them more likely to be seen as useful? Can we use a user's review history to predict how useful their future reviews will be?

## Goals

The primary goals of this project are to:

1) predict the usefulness of Yelp reviews as a classification problem
2) quantify usefulness of Yelp reviews on a numeric/percentage scale and predict this 'Usefulness' score as a regression problem

I would also like to explore:

1) Application and evaluation of models trained on Yelp review text on review text from other sites 
2) Assessment of reviews written by prolific Yelp users to predict usefulness of reviews on a per-user basis.

## Technologies Used

* **NLP**: Spacy, Textacy, Linear Dirichlet Allocation (LDA), Latent Semantic Analysis (LSA), non-Negative Matrix Factorization (NMF)
    
* **Modeling**: scikit-learn, statsmodels
    
* **Data Handling** pandas, PostgreSQL
