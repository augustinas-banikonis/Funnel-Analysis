# Funnel Chart Analysis
## Task

### Objective

- Analyze the data in the `raw_events` table to identify user events.
- Eliminate duplicate events to ensure accurate funnel analysis.
- Create a sales funnel chart using selected events and compare the funnel across the top 3 countries by event volume.

### SQL Code for the Analysis

```sql
WITH events AS (
  SELECT user_pseudo_id,
         event_name AS event_type,
         MIN(event_timestamp) AS Timestampp,
         country
  FROM `tc-da-1.turing_data_analytics.raw_events`
  WHERE country IN ('United States', 'Canada', 'India')
    AND event_name IN ('user_engagement', 'view_item', 'page_view', 'add_to_cart', 'purchase', 'begin_checkout')
  GROUP BY user_pseudo_id, event_name, country
),
country_events AS (
  SELECT event_type,
         COUNT(event_type) AS Event_count,
         country
  FROM events
  GROUP BY event_type, country
),
events_ranked AS (
  SELECT event_type,
         country,
         country_events.Event_count,
         RANK() OVER(PARTITION BY country ORDER BY Event_count DESC) AS event_order
  FROM country_events
)
SELECT events_ranked.event_order,
       events_ranked.event_type,
       SUM(CASE WHEN country = 'United States' THEN Event_count ELSE 0 END) AS United_States_events,
       SUM(CASE WHEN country = 'Canada' THEN Event_count ELSE 0 END) AS Canada_events,
       SUM(CASE WHEN country = 'India' THEN Event_count ELSE 0 END) AS India_events
FROM events_ranked
GROUP BY 1, 2
ORDER BY event_order, event_type;
```
## Results Table

| event_order | event_type       | United_States_events | Canada_events | India_events | United_States_percentage | Canada_percentage | India_percentage |
|-------------|------------------|----------------------|---------------|--------------|--------------------------|-------------------|------------------|
| 1           | page_view        | 118,333              | 20,242        | 25,331       | 100%                     | 100%              | 100%             |
| 2           | user_engagement  | 93,436               | 16,112        | 20,005       | 79.0%                    | 79.6%             | 79.0%            |
| 3           | view_item        | 26,953               | 4,653         | 5,795        | 22.8%                    | 23.0%             | 22.9%            |
| 4           | add_to_cart      | 5,603                | 993           | 1,162        | 4.7%                     | 4.9%              | 4.6%             |
| 5           | begin_checkout   | 4,310                | 764           | 878          | 3.6%                     | 3.8%              | 3.5%             |
| 6           | purchase         | 1,942                | 355           | 406          | 1.6%                     | 1.8%              | 1.6%             |

## Funnel chart
![Funnel Chart](https://github.com/augustinas-banikonis/Funnel-Analysis/blob/main/funnels.PNG)

## Main Analysis Points

- All regions contribute to the overall traffic, with the United States being the dominant source.
- The drop in user engagement is marginal, suggesting that a significant portion of visitors interact with the website's content, with engagement rates hovering around 79.04%.
- There's a substantial drop in the percentage of users who proceed from user engagement to viewing specific items, with an average decrease of approximately 56.3%. This indicates that while users engage with the website, a smaller proportion are interested enough to explore individual products, with view item rates averaging around 22.8%.
- The percentage of users who add items to their cart further decreases, signaling a drop in intent to make a purchase, with an average decrease of approximately 18.8%.
- The percentage of users who proceed to begin the checkout process continues the trend of decline from the previous stages, with an average decrease of approximately 1.15%.
- Despite the drop-offs at each stage, a notable portion of users across all regions successfully convert, though the conversion rate is relatively low, with purchase rates averaging around 1.66%.

## Suggestions

- Implement personalized recommendations and product showcases based on user preferences and browsing history to increase interest and engagement with specific items.
- Utilize retargeting strategies such as abandoned cart emails or targeted ads to remind users about items left in their carts and encourage them to complete the purchase.
- Implement chatbots or virtual assistants to provide instant assistance and guidance to users navigating the website and making purchasing decisions.
- Display user-generated content, such as customer reviews, ratings, and testimonials, on product pages to provide social proof and build trust with potential buyers.
- Conduct A/B testing and user feedback surveys to gather insights and validate hypotheses for optimizing the user experience.

## Things to Consider

- Optimize website performance and loading times to provide a seamless and responsive browsing experience for users. (Check with the SEO specialist, team)
- Use high-quality images and videos that accurately represent the products to enhance user confidence and trust in their purchases.
- Double-check if promotion codes work.

### Sources for conversion rate:
- Shopify: https://www.shopify.com/blog/ecommerce-conversion-rate (States an average of 2.5% to 3%)
- Adobe Experience Cloud: https://business.adobe.com/blog/how-to/ecommerce-conversion-rate-what-is-a-good-conversion-rate-and-how-to-optimize-it (Reports a 3.65% average with breakdowns by industry)



