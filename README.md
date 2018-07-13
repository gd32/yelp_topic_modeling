# Predicting the Usefulness of Yelp Reviews

What makes a useful Yelp review? Can we predict if a review will be useful based on its text content?

## Goals

The primary goals of this project are to:

1) predict the usefulness of Yelp reviews as a classification problem using machine learning models
2) use topic modeling/decomposition to improve the accuracy of those models
3) evaluate the effectiveness of the models by assessing the validity of the models' predictions

## Technical Report

An indepth discussion of this project is found in the [technical report](https://github.com/gd32/DSI_capstone/blob/master/Technical_Report.md).

## Technologies Used

All statistical analysis was done a t2.2xlarge AWS EC2 instance.

* **NLP**: [Spacy](https://spacy.io), [Textacy](https://github.com/chartbeat-labs/textacy), [scikit-learn](http://scikit-learn.org/stable/)
    
* **Modeling**: [scikit-learn](http://scikit-learn.org/stable/) - Logistic Regression, Random Forest Classifier
    
* **Data Management**: [numpy](http://www.numpy.org/), [pandas](https://pandas.pydata.org), [PostgreSQL](https://www.postgresql.org/)
