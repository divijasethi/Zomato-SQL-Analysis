
# Zomato Reviews Data Analysis Using Scrapping

## 1. Introduction

This project involves the analysis of customer reviews for Zomato, scraped directly from the App Store and Google Play Store. I created my own dataset by collecting reviews from the Zomato app The dataset contains various features such as review text, rating, sentiment labels, timestamps, and interaction metrics like likes and replies. The goal of this analysis is to uncover insights about user sentiment, rating patterns, reviewer behavior, and trends over time.

The dataset consists of 597 entries, each representing a customer review. The key attributes include the username of the reviewer, the review text, rating (on a scale of 1 to 5), sentiment label (positive, negative, or neutral), like count, review time, and the response time (for those reviews with replies).

Here’s a section you can include in your report or GitHub README to describe the data scraping process in detail, using the snippets from your notebook:

---

## Data Scraping Process

To create the dataset for Zomato reviews, I employed web scraping techniques to gather reviews from both the Google Play Store and the App Store. Below are the steps taken and the code used for scraping:

### Libraries Used

I utilized the following Python libraries to facilitate the scraping and data processing:

- **google_play_scraper**: This library was used to scrape reviews from the Google Play Store.
- **app_store_scraper**: This library was used for scraping reviews from the App Store.
- **pandas**: For data manipulation and organization into structured DataFrames.
- **numpy**: For numerical operations and handling array-like data structures.
- **json**: To manage JSON data formats.
- **os**: To interact with the operating system for file operations.
- **uuid**: To generate unique identifiers for the data entries.

### Scraping Reviews from Google Play Store

The following code snippet demonstrates how reviews were scraped from the Google Play Store for the Zomato application:

```python
from google_play_scraper import app, Sort, reviews_all
import pandas as pd 
import numpy as np 

# Scraping reviews from Google Play Store
zom_reviews = reviews_all(
    'com.application.zomato',  # Zomato app ID on Google Play Store
    sleep_milliseconds=0,      # Sleep time between requests in milliseconds
    lang='en',                 # Language of the reviews
    country='in',              # Country code for India
    sort=Sort.NEWEST           # Sort reviews by newest
)
```

In this code, the `reviews_all` function is called with the Zomato app ID, allowing for the retrieval of reviews sorted by the newest first. The parameters allow for customization such as language and country.

### Creating a DataFrame

Once the reviews were collected, they were converted into a Pandas DataFrame for easier manipulation:

```python
df = pd.DataFrame(np.array(zom_reviews), columns=['review'])
```

### Expanding and Cleaning the DataFrame

The next steps involved expanding the review data into separate columns and cleaning up unnecessary information:

```python
df2 = df.join(pd.DataFrame(df.pop('review').tolist()))
df2.drop(columns={'userImage', 'reviewCreatedVersion', 'appVersion', 'reviewId'}, inplace=True)
```

In this snippet, the `review` column is expanded into individual columns, and irrelevant columns are removed to retain only the essential information needed for analysis.

### Conclusion of the Scraping Process

After scraping reviews from the Google Play Store, similar processes were employed for the App Store using the `AppStore` class from the `app_store_scraper` library. The final dataset was then saved to a CSV file for further analysis.


## 2. Dataset Overview

The dataset contains the following columns:

- **Username**: The name of the user who posted the review.
- **Review**: The text content of the review.
- **Rating**: The numerical rating given by the user, ranging from 1 to 5.
- **LikeCount**: The number of likes received by the review.
- **ReviewTime**: The date and time the review was posted.
- **Reply**: Any reply made by the business to the review.
- **ReplyTime**: The timestamp of the reply (if applicable).
- **Sentiment_Label**: The sentiment analysis label for the review (positive, negative, neutral).
- **Score**: A numerical score associated with the review, likely representing the confidence in the sentiment classification.

## 3. Key Analyses and Findings

### 3.1 Sentiment Analysis

**Query**:
```sql
SELECT Sentiment_Label, COUNT(*) AS Count 
FROM Zomato 
GROUP BY Sentiment_Label;
```
**Finding**:  
Performed a sentiment analysis on the reviews to classify them into three categories: Positive, Negative, and Neutral. The goal was to determine the overall distribution of sentiments in the dataset. The majority of reviews were positive, with a smaller percentage of negative and neutral reviews. This indicates that customers on Zomato tend to express favorable opinions in their reviews.

### 3.2 Rating and Sentiment Distribution

**Query**:
```sql
SELECT Rating, Sentiment_Label, COUNT(*) AS Count 
FROM Zomato 
GROUP BY Rating, Sentiment_Label;
```
**Finding**:  
The reviews were grouped by both rating (from 1 to 5) and sentiment to identify how sentiments correlate with rating scores. Unsurprisingly, reviews with higher ratings (4 and 5) tend to have a positive sentiment. Lower ratings (1 and 2) are more likely to be associated with negative sentiments. The neutral reviews are distributed across various rating levels, indicating mixed feedback even within some moderate ratings.

### 3.3 Average Rating

**Query**:
```sql
SELECT AVG(Rating) AS avg_rating 
FROM Zomato;
```
**Finding**:  
The average rating across all reviews is approximately 4. This suggests that most reviewers on Zomato tend to leave high ratings, indicating general satisfaction with the service.

### 3.4 Top Reviewers

**Query**:
```sql
SELECT Username, COUNT(*) AS review_count 
FROM Zomato 
GROUP BY Username 
ORDER BY review_count DESC;
```
**Finding**:  
Identified the most active users by counting the number of reviews each user has posted. The top reviewer has posted multiple reviews, making them one of the most engaged users on the platform. This analysis can be useful for identifying loyal customers or potential brand ambassadors.

### 3.5 Rating Distribution

**Query**:
```sql
SELECT Rating, COUNT(*) AS rating_count 
FROM Zomato 
GROUP BY Rating 
ORDER BY rating_count DESC;
```
**Finding**:  
The most common rating is 5, followed by 4, indicating that the majority of users are satisfied with the service they received. Lower ratings (1, 2, and 3) are relatively rare, which further supports the general trend of positive customer feedback on Zomato.

### 3.6 Daily Review Trends

**Query**:
```sql
SELECT CAST(ReviewTime AS DATE) AS ReviewDate, COUNT(*) AS ReviewCount 
FROM Zomato 
GROUP BY CAST(ReviewTime AS DATE);
```
**Finding**:  
By grouping reviews by the date of submission, we can track the number of reviews submitted per day. This analysis shows periodic spikes in review activity, which may correspond to promotions, events, or holidays. Monitoring such trends can help Zomato predict busy periods or adjust marketing strategies accordingly.

### 3.7 Total Likes per User

**Query**:
```sql
SELECT Username, SUM(LikeCount) AS TotalLikes 
FROM Zomato 
GROUP BY Username 
ORDER BY TotalLikes DESC;
```
**Finding**:  
The analysis of likes per user reveals which users received the most engagement on their reviews. The top users in terms of likes may have particularly helpful or influential reviews. This information is useful for understanding the community dynamics on Zomato and potentially rewarding popular contributors.

### 3.8 Average Response Time

**Query**:
```sql
SELECT AVG(CAST(DATEDIFF(MINUTE, ReviewTime, ReplyTime) AS FLOAT) / 60) AS AverageResponseTime 
FROM Zomato 
WHERE Reply IS NOT NULL;
```
**Finding**:  
For the subset of reviews that received replies, we calculated the average response time. On average, businesses respond to reviews within several hours. This metric is useful for evaluating the responsiveness of businesses on the platform and can help improve customer satisfaction.

### 3.9 Monthly Review Trends

**Query**:
```sql
SELECT YEAR(ReviewTime) AS Year, MONTH(ReviewTime) AS Month, COUNT(*) AS ReviewCount 
FROM Zomato 
GROUP BY YEAR(ReviewTime), MONTH(ReviewTime) 
ORDER BY Year, Month;
```
**Finding**:  
Tracked the number of reviews posted each month to understand seasonal or monthly trends. This information can reveal high-traffic periods and help Zomato optimize platform performance or business promotions during peak times.

### 3.10 Rating Sentiment Comparison (High vs. Low Ratings)

**Query** (with CTE):
```sql
WITH AvgRating AS (
    SELECT AVG(Rating) AS AvgRating FROM Zomato
),
RatingCounts AS (
    SELECT CASE WHEN z.Rating >= a.AvgRating THEN 'High' ELSE 'Low' END AS RatingCategory,
    z.Sentiment_Label, COUNT(*) AS SentimentCount
    FROM Zomato z
    CROSS JOIN AvgRating a
    GROUP BY CASE WHEN z.Rating >= a.AvgRating THEN 'High' ELSE 'Low' END, z.Sentiment_Label
),
TotalCounts AS (
    SELECT CASE WHEN z.Rating >= a.AvgRating THEN 'High' ELSE 'Low' END AS RatingCategory, COUNT(*) AS TotalCount
    FROM Zomato z
    CROSS JOIN AvgRating a
    GROUP BY CASE WHEN z.Rating >= a.AvgRating THEN 'High' ELSE 'Low' END
)
SELECT r.RatingCategory, r.Sentiment_Label, r.SentimentCount * 100.0 / t.TotalCount AS SentimentPercentage
FROM RatingCounts r
JOIN TotalCounts t ON r.RatingCategory = t.RatingCategory;
```
**Finding**:  
Compared sentiments across reviews categorized as "High" (ratings above the average) and "Low" (ratings below the average). Reviews with higher ratings had a much higher percentage of positive sentiments, while low ratings corresponded more frequently with negative sentiments. This analysis highlights the relationship between satisfaction levels and expressed sentiment.

## 4. Conclusion

Here’s an expanded conclusion summarizing all the findings:

---

## Conclusion

In conclusion, the analysis of Zomato reviews provides valuable insights into user behavior and sentiments on the platform. The sentiment analysis revealed that the majority of reviews were positive, with fewer negative and neutral reviews, indicating general customer satisfaction. Ratings were closely aligned with sentiment, where higher ratings were mostly associated with positive sentiments, and lower ratings often correlated with negative ones. The average rating across all reviews was 4, further reinforcing the predominance of favorable feedback.

The analysis also identified key contributors, with certain users posting multiple reviews and receiving high levels of engagement in the form of likes. By tracking review submission dates, we found that review activity fluctuates, potentially tied to promotions or events. The businesses’ average response time to reviews, particularly those with replies, was found to be efficient, reflecting a reasonable level of engagement with customer feedback.

Overall, this analysis highlights the strong relationship between customer satisfaction (as measured by ratings) and expressed sentiment, while also providing insights into user engagement, like patterns, and review trends. These findings can guide businesses in improving customer service, tailoring responses, and better managing customer relationships on Zomato.
