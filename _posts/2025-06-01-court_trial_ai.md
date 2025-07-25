---
title: 3. RAG model for Danish court trial data
description: A model for retrieving relevant historic trials, given short text description
published: true
---

## [](#prologue)Prologue
Ethics regarding the use of AI for decision making in court, has been one of my favorite topics to discuss, as the philosophy of intelligence, conciousness, and emotions has had a big impact on my world view. 

As an engineer, I would also like to see things in practice, rather than just theorize about things. This is what drove me to start a project about AI with public Danish court trial data. Ideally, the available data would be structuered in a way similar to datasets in supervised learning, where we have a text description of the case, and a label corresponding to the punishment. Unfortunately, this was not the case, as the data consists of text documents that are not defined in a specific template. Therefore, the goal became to make a RAG model for retrieving these court documents based on a search prompt. I.e. to do natural text search on these documents. I want to do this using local models, and without LLM-pipeline libraries like LangChain.

As the ambition level of the project fell, I wanted to add a layer of complexity and utilize cloud computing for every step of the project, including testing, data processing, storage, and even endpoint hosting, primarily using Azure Machine Learning.

## [](#data-gathering)Data Gathering
In Denmark, we recently acquired a public database of court case documents, which are accesible through the [website](https://domsdatabasen.dk/) or the [API](https://domsdatabasen.dk/spoergsmaal-og-svar/api-adgang-til-domsdatabasen/). Although not exhaustive, it contains many cases from recent years. We will use the API to collect a local copy of all the data.

We first need to authenticate:
```python
import requests
from dotenv import dotenv_values
config = dotenv_values(".env")

# Authenticate with the API
url = "https://domsdatabasen.dk/webapi/kapi/v1/autoriser"
headers = {"Content-Type": "application/json"}
body = {"Email": config["username"], "Password": config["password"]}
response = requests.post(url, json=body, headers=headers)

# Check if the response is successful
if response.status_code == 200:
    data = response.json()
    print("Authorization successful!")
    print("User ID:        ", data["userId"])
    print("User Name:      ", data["username"])
    print("Issued UTC time:", data["issuedUtcTime"])
    token = data["tokenString"]
    print("Token:          ", token)
else:
    print("Error:", response.status_code, response.text)
```

And then we can fetch the data with a series of GET requests:
```python
headers = {'Authorization': f'Bearer {token}'}
sideNr = 1
while True:
  
    # Go through each page
    params = {
        'sideNr': sideNr,
        'perSide': 25
    }
    response = requests.get(url, headers=headers, params=params)

    if response.status_code == 200:
        data = response.json()
    ...
```
```python
# For each page, extract data
for item in data:

    # For each page, extract documents
    for doc in item['documents']:
        ...
```
```python
# Extract the HTML content of the document
document = doc['contentHtml']
if not document: continue
all_documents.append(document)

bs = BeautifulSoup(document, 'html.parser')
text = ''
for page in bs.find_all('div', class_='page'):
    text += page.get_text(separator="\n").strip() + '\n'

text_to_txt(text,     f'{texts_dir}/{doc["id"]}')
text_to_txt(document, f'{texts_dir+'_html'}/{doc["id"]}')
```

```python
# Extract meta data
all_meta.append([item['headline'],
                '?'.join([subject['displayText'] for subject in item['caseSubjects']]),
                int(item['id']),
                int(doc['id']),
                doc['displayTitle'],
                doc['verdictDateTime'],
                item['closedCourtroom'],
                item['profession']['displayText'],
                item['instance']['displayText'],
                item['caseType']['displayText']
                ])
```
The filters available in the request header is quite limited. We can extract 25 documents per page, which is reasonable, but we are not able to filter by time, or any other relevant attributes. This is also why we need to extract every document, since we can't guarantee which document we get otherwise.

I have added a small overview of all the important attributes. Some of them are attributes for the *case*, while some are attributes of the actual *documents* associated with a case.

| **Attribute**   | **Type**   | **Description**                                              |
|-----------------|------------|--------------------------------------------------------------|
| headline        | _case_     | Headline of the case                                         |
| caseSubjects    | _case_     | Themes/topics of a case (violence, drugs, taxes etc.)        |
| id              | _case_     | Unique ID for a case                                         |
| id              | _document_ | Unique ID for a document                                     |
| displayTitle    | _document_ | Title of document                                            |
| verdictDateTime | _document_ | Time of verdict (as defined in the document)                 |
| closedCourtroom | _case_     | Whether the courtroom has been closed from public or not     |
| profession      | _case_     | Type of case (criminal case, foreclosure etc.)               |
| instance        | _case_     | Stage of court: District court, High court, or Supreme court |
| caseType        | _case_     | More specicailly what category the case is                   |



## [](#data-processing)Data Processing
So, we split the response data into the meta data and the document data. The meta data can easily be described in a table, and will therefore be saved as a parquet-file (instead of csv) in order to save disk space. To further optimize space/memory usage, the column data types are selected manually.

```python
# Convert data types for efficiency
df_meta['id']           = df_meta['id'].astype('uint32')
df_meta['doc_id']       = df_meta['doc_id'].astype('uint32')
df_meta['verdict_date'] = pd.to_datetime(df_meta['verdict_date'])
df_meta['doc_type']     = df_meta['doc_type'].astype('category')
df_meta['profession']   = df_meta['profession'].astype('category')
df_meta['instance']     = df_meta['instance'].astype('category')
df_meta['case_type']    = df_meta['case_type'].astype('category')

df_meta.to_parquet('meta.parquet', index=False)
```

This is a somewhat neglible addition, as the table is ~5000x10. But it's still a good principle, and I also just learned these tricks from my "Python and High-peformance Computing" course, so I wanted to see it in practice.

As for the actual documents, they are stored in HTML format. The text itself can easily be extracted using the Beautiful Soup (bs4) library. It is also beneficial to split the text into small paragraphs, such that they can later be used for "chunking" (more on that later). It is easy to make these splits, as each actual paragraph is given by the HTML paragraph- or 'p' tag (see previous section for examples). 

Now, the most computationally demanding processing step is to convert each of these paragraphs into an embedding vector, which can take multiple hours. We explain the embedding vectors next.

### Sentence Embeddings
Building a RAG model requires that we can convert our prompt, as well as the reference material, into a vector representation, such that they can be compared. This representation is formally called an embedding. We need ML embedding models to perform this transformation.

As we are working with a Danish dataset, it is not possible to simply use the default models, as they are usually in English. Even so-called "multi-lingual" models can still struggle with Danish, as the Danish language is quite underrepresented on the internet. My best attempt was to use a model from Huggingface that was explicitly fine-tuned for the Danish language ([link](https://huggingface.co/KennethTM/MiniLM-L6-danish-encoder)). It's definitely not perfect, but it's alright.

A limitation I have encountered is that the tokenization of words seems a bit wrong. In terms of vector similarity, the word "Kølle" ("club", as in the weapon) and "Køleskab" ("refrigerator") are way too similar due to the *køl* -token. Let's consider this table of distances between embeddings (meaning that lower values indicate more similarity)

![embedding_matrix]({{ site.baseurl }}/assets/images/domstol/embedding_matrix.png "embedding_matrix")

Vector similarity is somewhat complicated, since embedding might actually be similar in one way, but not the way *I* want. In the matrix above we see that "bat" (bat), "stav" (rod/stick), and "hammer" (hammer) are somewhat similar to each other, but not similar to "Kølle".

### (Theory) Training an embedding model
Let's get into a bit more details regarding this embedding model. For basically any NLP (Natural Language Processing) -related task, we should use a transformer-based model. The most popular one would be the GPT-models that are used for the famous chatbots. But GPT-models are inherently text prediction models which are good for *generating* text, but not necessarily *classifying* text. On the other hand, BERT-models are good at classification as it takes an entire sequence and analyses each part of the sequence from both left-to-right, but also right-to-left, making them *bi-directional* (the B in BERT). Our embedding model is therefore a fine-tuned BERT model for (Danish) sentence embeddings, aka. Sentence-BERT.

To optimzie the weights of a deep neural network, we need a *loss-function* that defines how well the task is being solved during training, in order to nudge the weights in the right direction. This model has been trained using a *contrastive* loss function, which we need to minimize (but the fraction should be maximzied, so to speak):

![contrastive_loss]({{ site.baseurl }}/assets/images/domstol/contrastive_loss.png "contrastive_loss")

Let's understand this. The training dataset consist of N pairs of sentences, where each element (A and B) in a pair is somehow related to each other. Given the current state of the model, we try to embed A, B and many other sentences. The goal is the make the embedding of A and B more similar (numerator), while letting A and the other sentences become more dissimilar (denominator). Note that the vector operation between embeddings is the *inner product*

![inner_product]({{ site.baseurl }}/assets/images/domstol/inner_product.png "inner_product")

which is also called the *cosine similarity*

This loss is then propagated backwards through the network, and the weights are updated accordingly. Whether sentences are similar or not in the final model is hugely dependent on what the dataset has defined as similar. 

The model we are using is quite light-weight, which was probably to make fine-tuning to the Danish dataset easier. This means that our model embedding-dimension is (only) of size 384, but the maximum sequence (input) length is 512 tokens, which is actually a good amount (~400 words). This should make sure that all of our paragraphs fit individually, but not entire court trial documents.

Recall that our task is to find the trial document that best matches our prompt. We are quite limited by this token size, as we can't simply embed an entire document into a single vector (that would likely also require a larger embedding size). Instead, we will try and embed each paragraph of the documents, and track which paragraph belongs to which document. This is called "chunking", for which there are multiple strategies for how to embed these subtexts. We are lucky to have these beautifully split paragraphs, but for long texts, another strategy could be to embed overlapping subtexts of a fixed window-size, and a fixed overlap-size.

## [](#rag-model)RAG Model

Let's make it completely clear what we want to achieve using this model:

> **A list of the k most relevant documents wrt. a prompt, given the similarity between the prompt and a single paragraph in these documents. (I have allowed duplicate documents to be shown)**

It is, however, quite common to extend a chat-bot with RAG capabilities, but not necessary. To do this, the RAG model should return some text, which can then be injected into the "system prompt" (not the "user prompt"). A simplified example could be:

```
SYSTEM            : "You are a helpful chatbot that summarizes court trial documents" 
USER PROMPT       : "Man charged for violent behavior whlie under the influence of drugs"
<RAG>             : *search for relevant documents*
<RAG>             : *returns [doc_A, doc_B, doc_C]*
SYSTEM            : "The court trial in question this: {content of doc_A, doc_B, doc_C}
ASSITANT RESPONSE : "The trials are <...>"
```


If we were to do this ourselves, we should be cautious of:
1. The quality of the LLM used, due to challenges with the Danish language
2. Which information the RAG returns, as the entire reference document is too large to fit within the system prompt
Anyway, we are keeping it simple.

We have not talked about the challenge of searching for similar documents, which is very interesting. The straight forward way to find the most similar vectors wrt. the prompt vector, would be to simply check all options by brute force. In itself, this is slow, but using the FAISS (Facebook AI Similarity Search) library, many optimizations have been applied to speedup even the brute-force approach. It's very memory efficient, and it utilizes multiprocessing in order to parallelize the computation. They also offer approximate solutions for large datasets, where brute force in not feasible.

In the end, the flow looks like this:
![workflow_local]({{ site.baseurl }}/assets/images/domstol/workflow_local.png "workflow_local")




## [](#cloud)Cloud
At first, the reason for using the cloud was primarily just to learn more about it, despite thinking that it won't be necessary, but it was actually quite beneficial. First of all, it enabled me to host an endpoint for the model, which makes it possible to interact with it from anywhere in the world. But most importantly, it was actually quite nice to offload the data processing to a cloud computer. Not only would my local PC have been unavailable for proper use, the computation also happened much faster, since the cheaper VMs ran the processing way faster than my own PC, and also did not encounter any crashes. It took Azure 3½ hours using some of the cheapest compute, but mine had not even finished after 5+ hours locally.

![job_script]({{ site.baseurl }}/assets/images/domstol/job_script.png "job_script")


I had initially started working on Machine Learning Operations (MLOps) at a DTU course of the same name, where we tried doing some simple ML workflows on Google Cloud Platform, but did not utilize the ML-specific services. Later on, I got some experience in Azure, working in the ML team at GN. At GN, I went through Microsoft's official tutorials, and finally got to test it out during this project. 

Azure has everything we can ask for. 
1. It provides a **storage account** for *blob storage*
2. **environments** that VMs such as endpoints and job scripts can utilize
3. In Azure ML Studio we get many features such as **notebooks**, **job scripts**, **endpoint hosting** etc. It tries to encapsulate the entire ML workflow from start to finish.

This is my attempt at creating the equivalent workflow, from before, in the cloud:
![workflow_cloud]({{ site.baseurl }}/assets/images/domstol/workflow_cloud.png "workflow_cloud")

### Cost management
In Azure, you create an endpoint, but you also need to actually *deploy* a VM that acts as our API server. It should be noted that they run until you turn them off. I had falsely assumed that it was stationary until someone did an API call (cold start), which in turn cost me around 900 DKK.

For the overall expendeture of the project, I present the full overview:

![azure_costs]({{ site.baseurl }}/assets/images/domstol/azure_costs.png "azure_costs")

* ~900 kr. was spent on **Compute**, which is everytime a computer is running computation. This is notebooks, job scripts, building environments and endpoint deployments etc.
* ~100 kr. was spent on **Storage**, which is for storing the files I have in BLOB storage. This is the text files, FAISS index, and meta data files.
* ~30 kr. was spent on **Networking**, which is the cost of bandwidth of data leaving Azure. This must accumulate from multiple sources, and there is not an obvious resource that consumes a ton of bandwidth.
* ~30 kr. was spent on **Containers**, which is basically compute used in (emphemeral) containers. This would be the serverless job scripts that I used for processing.



## [](#results)Results
Now that the endpoint is up an running, we can try an inspect the results. 
```python
import requests
import json

endpoint = "<ENDPOINT-URL>" 
api_key  = "<API-KEY>"  

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {api_key}"
}

data = {
    "prompt": "Tilsalt for at besidde våben og stoffer"
}

response = requests.post(endpoint, headers=headers, json=data)
print(response.json())
```

Output...
```
'headline'     : 'Tiltale for bl.a. forsøg på manddrab ved at have planlagt at dræbe flere personer på skoler m.v. ved skyderier. Påstand om konfiskation'
  
'case_subjects': 'Strafferetlige sanktioner og andre foranstaltninger?Våben, eksplosiver og fyrværkeri?Liv og legeme'

'chunk_text'   : 'at have opnået våbentilladelse og tilladelse til at opbevare våben'

'distance'     : '0.7364452'
``` 

```
'headline'    : 'Landsretten stadfæstede byrettens dom i sag om tiltale for overtrædelse af lov om euforiserende stoffer § 3, stk. 1, jf. § 2, stk. 4, jf. bekendtgørelse om euforiserende stoffer § 30, stk. 1, jf. § 3, stk. 2, jf. bilag 1, liste B, nr. 59 og straffelovens §279 a.'
 
'case_subjects': 'Narkotika?Formueforbrydelser'
  
'chunk_text'   : 'vens § 191, stk. 1, 1. pkt., våbenbekendtgørelsen, lov om visse dopingmidler og lovgivningen om euforiserende stoffer.'

'distance'     : '0.7441238'
```

```
'headline'     : 'Landsrettens dom i sag om overtrædelse af straffelovens § 192a, stk. 1, nr. 1, jf. til dels stk. 3, jf. våbenlovens § 10, stk. 1, jf. § 2, stk. 1, jf. § 1, stk. 1, nr. 1 - 3 mv. stadfæstes med den ændring, at tiltalte straffes med fængsel i 3 år'

'case_subjects': 'Våben, eksplosiver og fyrværkeri?Strafferetlige sanktioner og andre foranstaltninger?Udlændinge?Narkotika'

'chunk_text'   : 'overdragelse og i overtrædelse af våbenlovgivningen ved at have været i besiddelse af en kre-ditkortkniv, et knojern og to peberspray.'

'distance'     : '0.7442008'
 ```

 ```
 {'headline': 'Tiltale for overtrædelse af bl.a. straffelovens § 191, stk. 1, 2. pkt., jf. til dels stk. 2, jf. lov om euforiserende stoffer § 3, stk.1, jf. § 2, stk. 4, jf. bekendtgørelse om euforiserende stoffer § 30 (dagældende § 27), jf. § 3, jf. bilag 1, liste B, nr. 70 samt straffelovens § 192a, stk. 1, nr. 1, jfr. stk. 3, jfr. våbenlovens § 10, stk. 1, jfr. § 1, stk. 1, nr. 1, 2 og 3. Påstand om konfiskation',
  'case_subjects': 'Narkotika?Våben, eksplosiver og fyrværkeri?Formueforbrydelser?Forbrydelser mod offentlig myndighed',
  'chunk_text': 'Lange fængselsstraffe for salg af kokain og besiddelse af skydevåben',
  'distance': '0.79919827'},
 ```

 ```
 {'headline': 'Tiltale for narko- og våbenbesiddelse. Påstand om konfiskation',
  'case_subjects': 'Narkotika?Våben, eksplosiver og fyrværkeri?Færdsel?Strafferetlige sanktioner og andre foranstaltninger',
  'chunk_text': 'Om våben og ammunition (forhold 1)',
  'distance': '0.8109229'}
```




- [call endpoint - code]
- 2 examples:
  - *define prompt*
  - *Top 5 chunks*
  - *Best headline*
  - *Qualitative description of most similar document(s)*

To get an intuitive feel of how close the vector are, other than the distance itself, we can display all the embeddings as a 2D prjection using PCA. It would seem that the chunks/points that are closest in the full space are also quite close in the 2 principal directions/dimensions
* Points in 2D space
![pca_projection]({{ site.baseurl }}/assets/images/domstol/pca_projection.jpeg "pca_projection")



...explained variance...

![explained_variance]({{ site.baseurl }}/assets/images/domstol/explained_variance.jpeg "explained_variance")




## [](#reflection)Reflection
* RAG quality
* RAG limitation
* Improvement ideas
* Cloud usage and accessibility
  * Azure in the future? SDKv1 vs SDKv2 troubles
  * Local dev vs online (slow feedback because of containers being built etc.)
  * Many things to consider: 
    * define environments properly
    * assign roles/permission to resources

