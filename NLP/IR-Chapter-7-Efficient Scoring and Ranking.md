### Efficient Scoring and Ranking

1. Champion Lists

2. Impact Ordering:
- A type of inexact top-K retrieval in which the postings lists are not all ordered by the same common ordering.
- Cannot concurrently process scores for all docs (NO DAAT)
- TAAT needs to be used in this ordering scheme since each term has its own ordering of documents.
- Order the documents d in postings list of term t by decreasing order of term frequency tf.
- Two ways to significantly reduce the documents to accumulate scores for:
	* Stop considering documents after some limit, like term frequency drops below some threshold or if "r" documents have been seen.
	* In the outer loop (over the query terms), we consider those terms with highest IDF first. Query terms likely to contribute the most to the final scores are considered first. This can be adaptive at query time. If the change in IDF score from q(t-1) to q(t) is not that high, we can choose to ignore checking the rest of the query terms, or we can decrease the number of documents to be processed from their respective postings lists (shorter prefixes).
- Basically, the ordering of each postings list is upto the designer. Could be ordered by term frequency, static scores, query-dependent scores, a combination of these scores, anything!
- This general setting of independent ordering schemes is known as __Impact Ordering__.

3. Cluster Pruning

- Like LOPQ in visual search.
- preprocessing step - cluster the document vectors.
	* Pick sqrt(N) documents at random from the collection (leaders)
	* For each non-leader (follower), compute its nearest leader and add it to that cluster. Approx. sqrt(N) followers for each leader. Next, process the query as follows:
	1. Compute closest leader
	2. Get cosine scores for all followers of that leader. Simple.
- Randomly selecting the leaders is good because it reflects the distribution better. Dense regions will have more leaders and hence finer sub-partitions can be formed in the document space.

- Variations of cluster pruning introduce additional parameters b1 and b2 (both positive integers.)
	* Each follower is assigned to its b1 closest leaders
	* At query time, we select b2 closest leaders to the query term initially.
	* In the simplest case above, __b1 = b2 = 1__.
	* Increasing b1 and b2 increase the recall, i.e. more likely to find the true top-K documents, but at the expense of more computation.
