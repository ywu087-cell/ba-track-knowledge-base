# Lesson 01 - Search Engines, SEO, SEM, and Auctions

Date: 2026-07-05  
Source: First BA track class PDF, about search engines  
Status: Organized notes, not a page-by-page transcript

## One-Line Summary

This lesson explains how search engines find, rank, and monetize information: organic ranking is handled through crawling, indexing, retrieval, and ranking models, while paid search uses SEM, CPC/PPC, Ad Rank, Quality Score, and auction rules.

## Concept Index

- Search Engine: software system for finding information on the web.
- SERP: Search Engine Results Page, the page where results and ads are shown.
- Crawling: process of discovering web pages.
- Index: searchable database built from documents and tokens.
- Top-k Retrieval: retrieving the most relevant candidate documents before ranking.
- Ranking Model: algorithm that orders results.
- Tokenization: splitting text into searchable terms.
- Term Frequency (TF): how often a query term appears in a document.
- Inverse Document Frequency (IDF): lower weight for common terms, higher weight for rarer terms.
- SEO: Search Engine Optimization, improving organic visibility.
- SEM: Search Engine Marketing, increasing visibility mainly through paid search.
- CPC / PPC: Cost Per Click / Pay Per Click.
- Ad Rank: paid-search ranking score based on bid, ad quality, thresholds, context, and extensions.
- Quality Score: estimate of ad and landing-page relevance and usefulness.
- First-Price Auction: winner pays their own bid.
- Vickrey Auction: highest bidder wins but pays the second-highest bid.

## 1. Search Engine Basics

A search engine is not just a website with a search box. It is a system that discovers information, organizes it, and returns ranked answers to a user query. The result page is called a SERP, and it can contain organic links, images, files, shopping results, and ads.

The course starts with the market context: search is dominated globally by a few major platforms, and digital advertising has become a large revenue channel. This matters for BA work because search is both a user-acquisition channel and a data-rich business process.

Key BA angle:

- Search engines convert user intent into ranked options.
- Ranking affects visibility, traffic, and revenue.
- Search behavior creates measurable signals, such as clicks, impressions, and conversion paths.

## 2. Why Ranking Matters

Users do not inspect search results evenly. Heatmap examples show that attention concentrates near the upper-left area and top results. If a listing is outside this high-attention zone, the chance of being seen drops sharply.

This creates a business problem:

- A good product or page can underperform if it is not visible.
- Ranking position changes traffic volume.
- Ranking quality affects user satisfaction because poor results reduce trust.

For BA analysis, ranking should be treated as a funnel problem:

1. Query entered.
2. Result shown.
3. Result noticed.
4. Result clicked.
5. User completes the desired action.

## 3. Search Engine Optimization Workflow

The lesson describes a simplified search engine pipeline:

1. The web contains many documents.
2. A crawler discovers pages.
3. An indexer turns pages into searchable data.
4. A user query is matched against the index.
5. Candidate documents are retrieved.
6. A ranking model orders the results.
7. The results page is shown to the user.
8. User behavior and evaluation data can feed learning and improvement.

The training loop is important:

- Define what makes users happy.
- Create guidelines.
- Use judges to produce training and test sets.
- Train or tune the ranking algorithm.
- Measure user behavior with metrics.
- Feed results back into the next improvement cycle.

## 4. Ranking Algorithms: From Boolean Search to TF-IDF

Early search can be understood as Boolean matching: a document either contains a term or it does not. Operators like AND, OR, and NOT help include or exclude terms, but this approach is too rigid because it does not measure degree of relevance.

The lesson uses food examples such as "croquets" and "bitterballen" to show why simple matching is not enough. A document can contain the query words many times, but that does not automatically mean it is the most relevant result.

### Term Frequency (TF)

TF counts how often a term appears in a document. It is useful because repeated query terms often indicate relevance.

Problem:

- Common or low-value words can distort the score.
- A stop word such as "and" may appear often but should not dominate ranking.

### Inverse Document Frequency (IDF)

IDF reduces the weight of terms that appear in many documents and gives more weight to terms that are rarer across the corpus.

Simplified idea:

```text
TF-IDF weight = term frequency x inverse document frequency
```

In the course example, "and" appears in many documents, so its IDF is low. More specific terms like "croquets" and "bitterballen" receive more useful weight. This corrects the naive TF score and makes the ranking more aligned with actual relevance.

## 5. SEO Ranking Factors

The course emphasizes that search ranking algorithms are not fully public and change constantly. The web itself also changes constantly. SEO professionals infer ranking factors through experiments, observation, and reverse engineering.

### On-Page Factors

On-page factors are signals controlled inside the page:

- Title
- Header
- Content
- Hyperlinks
- Words and term usage

BA interpretation:

- These are requirements and content-design levers.
- They should align page structure with user intent.
- Keyword usage should support meaning, not become keyword stuffing.

### Off-Page Factors

Off-page factors come from outside the page:

- Anchor text from other pages.
- Third-party reviews such as Yelp, Google reviews, or TripAdvisor.
- Link popularity and reputation signals.

BA interpretation:

- Off-page factors reflect trust, authority, and external validation.
- They are harder to control directly, so they often require partnership, reputation, and customer experience work.

## 6. SEM and Paid Search

Search Engine Marketing (SEM) is the paid-search side of traffic acquisition. In this lesson, SEM is distinguished from display ads, print, TV, and radio because it is tied directly to search behavior and SERP visibility.

### CPC / PPC

CPC means Cost Per Click, and PPC means Pay Per Click. The advertiser pays when a user clicks the ad. This makes search advertising measurable because cost can be connected to clicks, conversions, and downstream revenue.

Important BA metrics:

- Impressions
- Clicks
- Click-through rate
- CPC
- Conversion rate
- Cost per acquisition
- Return on ad spend

## 7. Google Ad Rank

Ad Rank determines where ads appear and whether they appear at all. The simplified teaching formula is:

```text
Ad Rank = Max CPC Bid x Quality Score
```

But the lesson also notes broader factors:

- Bid amount
- Auction-time ad quality
- Expected click-through rate
- Ad relevance
- Landing page experience
- Ad Rank thresholds
- User search context, such as location, device, search terms, and other signals
- Expected impact of ad extensions and formats

Key insight:

- A higher bid alone does not guarantee the top ad position.
- A more relevant ad with a stronger landing page can win a better position at a lower effective price.

This matters for BA because paid search performance is not only a budgeting problem. It is also a product, content, targeting, and landing-page quality problem.

## 8. Ad Position and Thresholds

The course gives an example where advertisers have Ad Rank scores such as 80, 50, 30, 10, and 5. If the threshold for showing above organic results is 40, only the advertisers with 80 and 50 can appear above the results.

If the threshold for showing below results is 8, advertisers with 30 and 10 may still appear below. The advertiser with 5 does not show at all.

Key takeaway:

- Position is an auction order, not a fixed visual location.
- Thresholds decide whether an ad is eligible for a placement area.
- Low Ad Rank can mean no impression, even if the advertiser is willing to pay something.

## 9. Auction Concepts

The lesson closes by connecting SEM to auction design.

### First-Price Auction

In a first-price auction, the winner pays exactly what they bid. This encourages strategic shading: bidders avoid revealing their true maximum value because overbidding can force them to pay too much.

BA takeaway:

- First-price rules create incentive problems.
- Participants think about competitors' bids, not only their own value.

### Vickrey Auction

A Vickrey auction is a sealed-bid auction where the highest bidder wins but pays the second-highest bid.

Example:

```text
A bids 1000
B bids 800
C bids 600

A wins and pays 800
```

The lesson uses this to show why second-price logic can reduce the penalty for truthful bidding. If you bid too high, you still pay the second price; if you bid too low, you may lose an item you actually value.

This connects to paid search because ad auctions need to balance advertiser incentives, platform revenue, ad relevance, and user experience.

## 10. BA Takeaways

- Search is a matching and ranking system, not just a keyword lookup.
- Organic ranking and paid ranking both depend on relevance.
- SEO requires both content structure and external authority.
- SEM requires budget, targeting, ad quality, and landing-page quality.
- Auction rules shape participant behavior.
- Metrics are essential: impressions, clicks, CTR, CPC, conversion, and revenue connect search behavior to business outcomes.

## Review Questions

1. What is the difference between crawling, indexing, retrieval, and ranking?
2. Why does a high term frequency not always mean a document is relevant?
3. How does IDF reduce the impact of common terms?
4. What is the difference between on-page and off-page SEO factors?
5. Why can a lower bid sometimes beat a higher bid in paid search?
6. What does Quality Score represent?
7. How do Ad Rank thresholds affect whether an ad appears?
8. Why does a first-price auction encourage strategic bidding?
9. Why does a Vickrey auction make truthful bidding more attractive?
10. As a BA, which metrics would you monitor to evaluate search marketing performance?

## Quick Glossary

| Term | Meaning |
| --- | --- |
| SERP | Search Engine Results Page |
| SEO | Organic optimization for better search visibility |
| SEM | Paid search marketing |
| TF | Count of query-term appearances in a document |
| IDF | Weight that downplays terms appearing in many documents |
| CPC | Cost paid per ad click |
| PPC | Pay-per-click advertising model |
| Ad Rank | Score used to determine paid ad position and eligibility |
| Quality Score | Estimate of ad and landing-page relevance and usefulness |
| Vickrey Auction | Auction where highest bidder wins and pays the second-highest bid |
