## BLEU Score

1. What is it?
- Stands for Bilingual Evaluation Understudy Score
- Proposed by Papineni et al. (2002)
- Metric for evaluating a generated sentence with respect to a ground truth sentence.
- Ranges between 0.0-1.0 with 1.0 being the best possible BLEU score.

2. Some interesting properties of the BLEU score
- Quick to compute
- Independent of language
- Widely adopted

3. How to calculate it?
- A BLEU-5 score finds all the matching five-grams in both the generated and reference sentences.
- Once a matching pair is found, this word is removed from the reference sentence, because a generative model can generate common words repeatedly and this can skew the score.

4. Cumulative N-gram scores:
- Usually, the BLEU score is calculated for all n-gram orders from 1 to n and then a geometric mean is found between these scores, with a weight assigned to each order.
- BLEU-1 to BLEU-4 scores are commonly reported for text generation systems.

5. More information can be found [a "https://machinelearningmastery.com/calculate-bleu-score-for-text-python](here).