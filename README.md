# SYN x MLDA <br>Contextualized Graph-based <br>Sentiment Analysis Tool 😄😡😭😀😍


## Motivation
In many contexts it is useful to understand the sentiment of text data, but current setiment analysis solutions are rubbish. 
Even the best current solutions struggle with mixed emotion phrases, sarcasm, slang, code-switchings and creoles. 

## Objective
We would like to build a Proof of Concept(POC) for a fundamentally different sentiment tool that can identify
sentiment of conversation using Human Centred Data Science principles (HCDS).The HCDS will be based on networks of keywords that demote
emotion and take into account nuances within language, double meanings and the context of language use. It will not be based on a pre-identified list of emotional keywords or modifiers/ amplifier words (‘not’, ‘extremely’).

## Methodology
We make use of network theory to analyze clusters of keywords that denote emotion /sentiment based on synonyms / words with similar dictionary meanings. 

Pipelines in the Sentiment Analysis Tool:
* Baseword Selection for Synonym Network
* Synonym Extraction
* Synonym Network Construction
* Emotion-denoting Word Prediction
* Sentiment Prediction(Emotion Distribution Score Calculation)

### Emotion Wheel
We adopt Emotion & Feeling Wheel from [The Junto Institute](https://www.thejuntoinstitute.com/emotion-wheels/)
 <img src="/emotion_wheel.png" width="50%"> \
  source:[https://www.thejuntoinstitute.com/emotion-wheels/](https://www.thejuntoinstitute.com/emotion-wheels/)

### Baseword Selection for Synonym Network
In this POC studies, we select [IMDB review data](https://ai.stanford.edu/~amaas/data/sentiment/) as our primary corpus as the reviews contain wide range of emotions, which allow us to capture as many emotions as possible. We first find the top words from IMDB and rank them according to their frequency in the review dataset. Then, we compare the ranking with the [Google n-gram dataset](https://books.google.com/ngrams/info) to select 1k top words that share the same normalized ranking. 

### Synonym Extraction
Synonyms of the base 1k words are retrieved from the free [Merriam Webster Thesaurus API](https://dictionaryapi.com/products/api-collegiate-thesaurus) using the notebook in the **tool** folder. Note that there are 1k daily API limit for free accounts. After the first iteration extraction of synonyms, we found out that some of the emotion keywords in the Emotion Wheel are neither in the basewords nor in the synonyms, therefore, second iteration is performed on emotion keywords that are not covered in the first iteration as a way of bootstrapping the network. 

### Synonym Network Construction
We construct the network using undirected graph and unweighted edges with [Gelphi](https://gephi.org/) and output network metrics for the downstream processing. The modularity_class, authority score, average weighted degree(in our case, it is the degree of a node), and betweenness centrality measure are exported from Gelphi (A sample of the output is available under the **source_data** folder ).

### Contextualized Emotion-denoting Word Prediction
This is the most challenging part of the whole project. There are two goals we want to achieve here.
* Contextualization of potential emo-denoting words in the sentence: \
BERT masked word prediction is used to provide the context as the word predictions at the masked position can be used to infer the context.

  <img src="/BERT.png" width="70%"> \
  source:[http://jalammar.github.io/illustrated-bert/](http://jalammar.github.io/illustrated-bert/)

* Emo-denoting predictioon based on context and the source word:\
Since we have a SYN network, we can use each word's information in the network to predict whether it is emo-denoting using simple machine learning model (our ablative studies suggest that Random Forest Tree achieves the best F1-score). This part requires labelled data, a sample of such model input is in **output** folder 
named `bert_final_output.csv`, ( 1 indicates emo-denoting word under `emo?` column). For those words not found in the network, -1 is used to fill up their network related information and not used for model prediction. The input to the model consists of two parts, network information of source word (self), and aggregated average network information of predicted masked words from the BERT model in the previous step (predicted).


### Sentiment Prediction
After we are able to pick up the emo-denoting words from the previous step, we can make use of the network again to do the sentiment prediction. The average shortest distances between each predicted emo-denoting word to the Top 10 words in each modularity_class are computed, 1/(average class distance) would be used as the score to the sentiment class. Softmax function is used to convert the scores to probability distribution of the predicted emo-denoting word to each pre_defined sentiment class, finally the sentiment class assignment order based on the probability(first class is the top sentiment class) ,see `emo_dist_prob` and `emo_dist_cluster_order` in `emo_assignment.csv` under **output** folder.


## Instruction
### Inputs required by the sentiment analysis tool
* edge and node csv list to generate the network
* output from Gelphi
* corpus for sentiment analysis
Refer to **source_data** folder

### Steps
<ol>
  <li>Get your input ready</li>
    <li>Execute the notebooks according to the numbering</li>
  </ol>


## Acknowledgement
We would like to thank [Synthesis Partner](https://www.linkedin.com/company/synthesispartners/) for this wonderful learning experience. It would not be possible without the consistent support and guidance from [Ankit](https://www.linkedin.com/in/kalkar/) and Synthesis strategiests. A big shoutout to them! 👏👏

