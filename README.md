# Neural Topic Modelling
Topic modelling is usually done through statistical models using expectation maximization. However, this approach has many shortcomings, including a long training time, poor generalizability, and an inability to leverage the inherit semantics of words to determine topics. The method used here remedies many of these issues, by using a learned neural model rather than a statistical model.

## Algorithm Outline
The neural network consists of the following steps:

- Each sentence is split into words and each word is vectorized
- Each vectorized sentence goes through an embedding algorithm to obtain a vector representing the semantics of the sentence as a whole
    - This involves taking a weighted average of the vectors of the words in the sentence, for details see below
- The algorithm attempts to reconstruct a vector similar to the embedding vector only using the topic vectors
    - In this neural topic modelling algorithm, the topics are represented as vectors just as words
- The loss is calculated by comparing the sentence embedding, reconstruction, and the pseudo-embeddings of a small sample of other sentences in the corpus
    - The algorithm learns to combine the topics in a way to closely match the actual embedding, as well as being distinct from embeddings of other sentences

## Sentence Embedding

- The algorithm attempts to get the meaning of a sentence by getting a weighted average of the word vectors in the sentence
- First, a simple average of the word vectors is calculated
    - This is used as a rough estimate of the general context of the sentence
- Then, each word vector is multiplied by the average vector and a learned attention matrix M
    - The output is a single number, representing the prominence the word is judged to have within the sentence
- After a prominence value is calculated for every word, they are passed through a softmax function to get the final proportions of each word in the overall sentence vector

### Performance

- Due to the softmax step, this process has a strong tendency to delegate nearly all the attention to a single word in the sentence
    - This is because even small differences (~1) in the prominance values means the higher word has multiple times more prominance in the final combination
    - Often, this is a good thing, since in many sentences only one word is relevant to the topic
    - However, in many cases multiple words are relevant to the topic/combination of topics
        - In most of these cases the algorithm fails to reflect this reality
    - It is possible to replace softmax with sparsemax (details below), but this does not seem to help, and in fact if anything makes the problem worse

## Sentence Reconstruction

- The algorithm attempts to stratify the sentence into topics by approximating its embedding with topic vectors
- This is done through what is equivalent to a neural network with one softmax layer
- The ultimate goal is to approximate the embedding vector with a combination of topic vectors determined by the neural network, called the sentence reconstruction
    - The loss is calculated over all embeddings, as well as over 20 negative samples for each embedding
    - For each embedding and negative sample, the loss is 1-zr+zn, where:
        - z is the embedding vector
        - r is the reconstruction vector
        - n is the negative sample
        - The dot product is taken between vectors
    - Negative samples are chosen randomly from the corpus
        - The vectors are just simple means of the words vectors in the sentence, rather than true embeddings
        - This done to reduce computation time, as calculating embedding is relatively costly and using embeddings for negative sample would increase the number of calculated embeddings by twentyfold
    - If the loss value is negative, the loss is 0 instead