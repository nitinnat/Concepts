# Chapter 7

## 7.1. Efficient Scoring and Ranking

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

## 7.2. Components of an IR system

### 7.2.1. Tiered Indexes
- Sometimes the list of __A__ contenders used in champion lists have fewer than K documents. Here, we can use tiered indexes which are a generalization of champion lists.

- Set different tf thresholds for each tier and use the index below as backup if K documents are not retrieved.

- For example, tf_tier1 = 20 => tier 1 index consists of postings entries that have a tf  > 20, tier 2 index has postings entries which have tf > 10, and so on.

### 7.2.2. Query-term proximity
- In free-text queries that we commonly use on Google, we would like to see documents in which the query terms we entered are close to each other. 
- Define a window of size __w__ which is the smallest window in a document __d__ that contains all the query terms, measured in the number of words in that window. 
- For example, in a document that just contains the sentence "The quality of mercy is not strained", the query "strained mercy" would have __w = 4__.
- Smaller the size of __w__ the better.
- If document does not contain all the words in the query, then __w__ can be set to something arbitrarily large.
- Variants include removing stop words and considering only the non-stop words to compute __w__.
- Soft conjunctive semantics possibly used by Google and such.


### 7.2.3. Designing parsing and scoring functions

- Common search interfaces employ multiple queries against multiple indexes for a single user-defined query.
- Example: "rising interest rates". Run the user-generated query as a phrase query -> obtain docs with highest cosine scores. If less than 10 docs obtained, run bigram queries and rank these using vector-space scoring as well. If we still have fewer than 10 results, run unigram queries.
- Each of the above steps will yield a list of documents whose scores have been obtained from multiple factors ( vector space, static quality, proximity weighting, etc).
- Need an score accumulating function, e.g. Machine Learning scoring.

### 7.2.4. Putting it all together
- Parsing and linguistic processing. Resulting stream of tokens are fed into two blocks.
* First, retain a copy of the parsed document in a document cache. Useful to generate snippets to help the user understand why this document is relevant.
* Second, feed the tokens into a bank of indexers that create multiple indexes including zone and field indexes to store metadata for each document, tiered positional indexes, indexes for spelling correction, inexact top-K retrieval indexes.
- A free text query is sent to the indexes both directly and via a spelling correction module (optional and invoked only if original query fails to retrieve results)
- Retrieved docs are passed through the scoring module that uses ML to compute scores for each document.
- Finally, these documents are rendered on the result page.

## 7.3. Vector space scoring and query operator interaction
- In building a search engine, we may opt to provide multiple query operators for the end user (wildcard, bigram, etc). How do we handle multiple query operators at once?

- __Boolean Retrieval__
* A vector space index can handle a boolean query and __not__ vice-versa.
* Boolean retrieval needs the user to specify a formula to select documents with the presence (or absence) of query terms with no ordering specified.
* It is mathematically possible to use p-norms to combine boolean and vector space queries, but this is rare.

- __Wildcard Retrieval__
* Wildcard index is different from V.S. index (vector space)
* First step for both is common - both require postings and a dictionary, wildcard dict can have trigrams, for example.
* If wildcards are allowed as part of free text, then we can interpret the wildcard part as spawning several terms in the vector space. For example, "rom* restuarant" could spawn the query vector "rome roman restaurant".

- __Phrase queries__
* V.S. index cannot be used for phrase queries. For example, the phrase "German Shephard" gets encoded in the _german shephard_ axis but also has non zero scores in the _german_ and _shephard_ axes in the vector space. Also, IDF notions need to be extended to such situations to include these phrases.
* We only have weights for the individual terms and don't really have a weight for the whole phrase. 
* They can be combined like in the 3 step process described in point 4 above.

