---
title: 4. RAG model for Danish court trial data (Part 2/2)
description: (Part 2/2) Hosting the model in the cloud
published: true
---

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

* ~900 kr. was spent on **Compute**, which is every time a computer is running computation. This is notebooks, job scripts, building environments and endpoint deployments etc.
* ~100 kr. was spent on **Storage**, which is for storing the files I have in BLOB storage. This is the text files, FAISS index, and meta data files.
* ~30 kr. was spent on **Networking**, which is the cost of bandwidth of data leaving Azure. This must accumulate from multiple sources, as there is not an obvious resource that consumes a ton of bandwidth.
* ~30 kr. was spent on **Containers**, which is basically compute used in (emphemeral) containers. This would be the serverless job scripts that I used for processing.



## [](#results)Results
Now that the endpoint is up an running, we can try an inspect the results of the following prompt:

> **(DK) Prompt: "Tiltalt for at besidde våben og stoffer"** \
> **(EN) Prompt: "Charged with possession of weapons and drugs"**

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
    "prompt": "Tiltalt for at besidde våben og stoffer"
}

response = requests.post(endpoint, headers=headers, json=data)
print(response.json())
```

This request retrieves the 5 best matches, from best to worst, in a JSON style format. To make it more readable here, I've omitted some numbers, and formatted it neatly:
```
RESULT 1)
'HEADLINE'     : 'Tiltale for bl.a. forsøg på manddrab ved at have planlagt at dræbe flere personer på skoler m.v. ved skyderier. Påstand om konfiskation'
'CASE_SUBJECTS': 'Strafferetlige sanktioner og andre foranstaltninger. Våben, eksplosiver og fyrværkeri. Liv og legeme'
'CHUNK_TEXT'   : 'at have opnået våbentilladelse og tilladelse til at opbevare våben'
'DISTANCE'     : '0.7364452'
``` 

```
RESULT 2)
'HEADLINE'     : 'Landsretten stadfæstede byrettens dom i sag om tiltale for overtrædelse af lov om euforiserende stoffer § 3, stk. 1, jf. § 2, stk. 4, jf. bekendtgørelse om euforiserende stoffer § 30, stk. 1, jf. § 3, stk. 2, jf. bilag 1, liste B, nr. 59 og straffelovens §279 a.'
'CASE_SUBJECTS': 'Narkotika. Formueforbrydelser'
'CHUNK_TEXT'   : 'vens § 191, stk. 1, 1. pkt., våbenbekendtgørelsen, lov om visse dopingmidler og lovgivningen om euforiserende stoffer.'
'DISTANCE'     : '0.7441238'
```

```
RESULT 3)
'HEADLINE'     : 'Landsrettens dom i sag om overtrædelse af straffelovens § 192a, stk. 1, nr. 1, jf. til dels stk. 3, jf. våbenlovens § 10, stk. 1, jf. § 2, stk. 1, jf. § 1, stk. 1, nr. 1 - 3 mv. stadfæstes med den ændring, at tiltalte straffes med fængsel i 3 år'
'CASE_SUBJECTS': 'Våben, eksplosiver og fyrværkeri. Strafferetlige sanktioner og andre foranstaltninger. Udlændinge. Narkotika'
'CHUNK_TEXT'   : 'overdragelse og i overtrædelse af våbenlovgivningen ved at have været i besiddelse af en kre-ditkortkniv, et knojern og to peberspray.'
'DISTANCE'     : '0.7442008'
```

```
RESULT 4)
'HEADLINE'     : 'Tiltale for overtrædelse af bl.a. straffelovens § 191, stk. 1, 2. pkt., jf. til dels stk. 2, jf. lov om euforiserende stoffer § 3, stk.1, jf. § 2, stk. 4, jf. bekendtgørelse om euforiserende stoffer § 30 (dagældende § 27), jf. § 3, jf. bilag 1, liste B, nr. 70 samt straffelovens § 192a, stk. 1, nr. 1, jfr. stk. 3, jfr. våbenlovens § 10, stk. 1, jfr. § 1, stk. 1, nr. 1, 2 og 3. Påstand om konfiskation'
'CASE_SUBJECTS': 'Narkotika. Våben, eksplosiver og fyrværkeri. Formueforbrydelser. Forbrydelser mod offentlig myndighed'
'CHUNK_TEXT'   : 'Lange fængselsstraffe for salg af kokain og besiddelse af skydevåben'
'DISTANCE'     : '0.79919827'
```

```
RESULT 5)
'HEADLINE'     : 'Tiltale for narko- og våbenbesiddelse. Påstand om konfiskation'
'CASE_SUBJECTS': 'Narkotika. Våben, eksplosiver og fyrværkeri. Færdsel. Strafferetlige sanktioner og andre foranstaltninger'
'CHUNK_TEXT'   : 'Om våben og ammunition (forhold 1)'
'DISTANCE'     : '0.8109229'
```

In the output, the "CHUNK_TEXT" is what is actually being compared with the prompt, and "DISTANCE" defined how close the sentences are in vector space. 

Even if the chunk-text in result 2 and 4 are the only ones that actually mentions both weapons and drugs, the "case subjects" usually include both. This is purely coincidental.
It's hard to say why only 2 and 4 seem valid, as the distance measure doesn't come with a good explanation. Result 1 does not even have "drugs" in the case subject. The embeddings might have had a focus on something different. To maybe better understand these embeddings, let's try visualizing them.


### PCA projection

To get an intuitive feel of how close the vector are, other than the distance itself, we can display all the embeddings as a 2D prjection using PCA. This means that all the information of the 384 dimensions have been cramped into 2 dimensions, or *principal components* (PC). These PCs are basically combinations of all 384 dimensions, but are supposed to "span the 2D space of most information".

It would seem that the chunks/points that are closest in the full space are also quite close in the 2 principal directions/dimensions:

![pca_projection]({{ site.baseurl }}/assets/images/domstol/pca_projection.jpeg "pca_projection")

But notice that the results 1 and 4 are the most dissimilar to the prompt, in PCA space, while 2,3 and 5 are more similar (also to each other). We should understand this as: By only considering the 2 most important "directions of information", results 2,3 and 5 have more in common with the prompt. The compression into 2 dimensions is only meant to be guiding, and used for visualization, but it can actually be useful to reduce it to something like 100 dimensions, as the remaining dimensions might be considered noise.

![explained_variance]({{ site.baseurl }}/assets/images/domstol/explained_variance.jpeg "explained_variance")

The graph above shows how much information is kept, by reducing the dataset to K components. Using 100 out of 384 PCs, we keep around 80% information. Experience tells me that this dataset can't really be compressed, as all the dimensions have a quite big impact, since the curve is not very start for low K's. This would suggest that the embeddings don't have a few latent variables that explain most of the information.

## [](#reflection)Reflection
The RAG is quite limited by the Danish embeddings, even though it did alright. I should definitely also have included the headline as a valid chunk that could be compared to the prompt embedding, but I did not think of that in time. The RAG could be improved further if I pushed the limit on how much text I would assign to each chunk. The current paragraphs/chunks don't even come near the ~400 word limit. I also imagine that doing the project on a translated version of the court data might actually have been better, instead of training an embedding model on Danish data. The language is definitely a bottleneck in making this work perfectly, but not necessarily the first bottleneck. 

### Assesment of cloud computing
The idea of having everything hosted on the cloud in a pay-as-you-go fashion is quite appealing. In theory it's great, but in practice it has been a mess to get working. It's definitely more clear to me what to do in the future, but it still requires a solid plan as you quickly lose the overview. 

First of all, you have to be careful what resources are running, which is not displayed in a clear way. Also, the Azure SDK (libraries, whatever) are very confusing, as they have 2 very different versions that tend to conflict with each other. Everything worked when I was consistent at using SDKv2 only, but it was very unclear at first that there was this distinction.

Another thing to consider is the feedback of doing changes. Azure ML does provide cloud notebooks which lets one run code directly in the cloud, but they take some time to get up and running. For almost any other thing, it would take a long time to see if it worked. This is because things behave differently on the cloud, as they need certain access rights, and code has to work with the online environment etc. . When working locally, things usually just work. The biggest reason for using the cloud is that its available almost 100% of the time. The cloud computers don't need to be turned off, which makes server hosting and automatic script execution very easy. 

### Final thoughts
This has been a very large project, and I have learned a ton. I never expected the results to be any good, as the methods used/learned were important in themselves. I have always struggled with cloud computing, but I think this experience helped me figure it out. I am still interested in working with court data, but that would be in the context of creating an AI to assist or replace the classic role of the judge.