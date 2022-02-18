# Paragraph-Extraction

Given any set of documents (usually PDFs), and a phrase (which we term ”query”), we look to expand the query into all possible contexts it is associated with, and search for those contexts - paragraph wise, in the document(s). We return all the paragraphs from the document(s) which have the closest association with the given query.

# Implementation Details 

## Query Processing

### Word Substitution 
Let us say we have q words in our query, after cleaning the data (removing stop-words and other irrelevant information). And we are considering
the top n most-similar words for each word in the query. We have to choose
words out of the top n most-similar words to substitute the original word with.
And we have to consider all possible combinations.
<br /> <br />
This leads to (n+1)q many expanded queries that needs to be processed on
the document. Each query size is approximately equal to the original query
size, scaled to some factor close to 1. Also, we have to filter out the combinations based on the similarity factors of the results that we get.

No. of queries: (n+1)q<br />
Query size: q<br />
Time Complexity: O ((n+1)^q . q . n . log v)<br />
[Explanation: For each of the O ((n+1)^q) expanded queries, we have approximately q words in each query. For each word, we can find n similar words in O(n . log v)]<br />
Space Complexity: O ((n+1)^q. q) [Explanation: For each of the O ((n+1)^q) expanded queries, we have approximately q words in each query]<br />
[v : vocabulary of the pre-trained model i.e 3M]<br />

### Word Stacking 
Let us say we have q words in our query, after cleaning the data (removing stop-words and other irrelevant information). And we are considering
the top n most-similar words for each word in the query. We have to choose
words out of the top n most-similar words to substitute the original word with.
And we have to consider all possible combinations.

No. of queries: 1<br />
Query size: q.n<br />
Time Complexity: O (q . n . log v)<br />
[Explanation: For each of the q words, we can find n similar words in O (n . log v)]<br />
Space Complexity: O (q . n)<br />
[Explanation: The final query size determines the space complexity]<br />
[v: vocabulary of the pretrained model i.e 3M]<br />

## Document Processing
### Weighted Frequency Count
We count the frequency of the query in each paragraph by taking all the words in the query one by one, counting its no. of occurrences in that paragraph and multiplying it by the similarity factor. After that, we sort the paragraphs in non-increasing order of the frequencies and choose the top k paragraphs as our result.

Time Complexity: O (c)<br />
[Explanation: O (c) is achievable with advanced string-matching algorithms like Suffix Trees / Arrays]<br />
Space Complexity: O (p)<br />
[Explanation: We need a counter for each of the p paragraphs]<br />
[c: No of characters in the document, p: No of paragraphs in the document]<br />

### Word2Vec
We consider the expanded query as a paragraph and find its paragraph
vector using the same greedy approach. Finally, we take the top k most-similar
paragraphs with respect to above query vector. The similarities can be evaluated using the cosine similarity function. The time complexity of the cosine
similarity function is O (d), where d is the vector dimension.<br /><br />
Time Complexity: O ((w + k) . d)<br />
[Explanation: Each word in the document is mapped to a vector on which we are using simple addition operations that can be achieved in O (w . d). And the top k most-similar paragraphs can be found using the cosine similarity function in O (k . d)]<br />
Space Complexity: O (p . d)<br />
[Explanation: Each of the p paragraphs is mapped to a vector with dimension d]<br />
[w: No of words in the document, p: No of paragraphs in the document, d: Dimension of the paragraph-vectors (300)]<br />

### Doc2Vec
We feed all the paragraphs in a Doc2Vec Model (with dimensions same as
the pre-trained word2vec model) and get the corresponding paragraph vectors.
Now, we find the top k most-similar paragraphs in the model with respect to
the expanded query, which is considered as a paragraph.<br /><br />
Time Complexity: O ((w + p) . log (w + p) . d + k . log (w + p))<br />
[Explanation: The time complexity of training the doc2vec model is O ((w + p) . log(w + p) . d). And the top k most-similar paragraphs can be found using the model in O (k . log (w + p))]<br />
Space Complexity: O (p . d)<br />
[Explanation: Each paragraph is mapped to a vector with dimension d]<br />
[w: No of words in the document, p: No of paragraphs in the document, d:Dimension of the paragraph-vectors (300)]<br />

### BERT
We feed all the paragraphs from
our document one by one in the model and get the corresponding paragraph
embeddings.
Next, we consider the expanded query as a paragraph and find its paragraph
embedding by feeding it in the same model. Finally, we take the top k most-similar paragraphs with respect to the above query embedding. The similarities
can be evaluated using the cosine similarity function. The time complexity of
the cosine similarity function is O (d), where d is the vector dimension.<br /><br />
Time Complexity: O (p . log (v) . d + k . d)<br />
[Explanation: The time complexity of retrieving the sentence/paragraph embedding of p paragraphs from the Sentence Transformer model is O (p . log(v) . d). The top k most-similar paragraphs can be found using the cosine similarity function in O (k . log v).]<br />
Space Complexity: O (p . d)<br />
[Explanation: Each of the p paragraphs is mapped to a vector with dimension d]<br />
[w: No of words in the document, p: No of paragraphs in the document, v: Pre-trained model constant ( 10M), d: Dimension of the paragraph-vectors (300)]<br />
