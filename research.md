# layoutLM: the Holy Grail of Understanding

the what, why and how on the path to scanned document parsing, using ML \
_by Mattia Callegari, "the Intern", 2024_

## The task

"Extracting information from documents": the twist? the documents were photos!
The usual PDFs scraping tools were now out of reach: new tools were necessary.

## The easy route

### [Azure AI Document Intelligence](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/?view=doc-intel-4.0.0&branch=release-build-cogserv-forms-recognizer)

This was the first tool: a ready-to-use, refined service that did exactly what was needed.
Send an image (in whatever format you wanted.. png, jpeg, tiff) to the [Document processing model](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-model-overview?view=doc-intel-4.0.0)
of choice, and voilà! [OCR](https://en.wikipedia.org/wiki/Optical_character_recognition) gets perfomed, the layout of the document read and the relevant information extracted: all based on the model you choose (or created!).

You could provide Azure with examples of the documents you were trying to parse and create your own [Custom model](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-custom?view=doc-intel-4.0.0).

Basically the perfect solution: the catch? It's [expensive](https://azure.microsoft.com/en-us/pricing/details/ai-document-intelligence/)!

## The hard route

After a bit of [research](https://www.microsoft.com/en-us/research/project/document-ai/), we found the backbone of Azure AI Document Intelligence service: LayoutLM. A state-of-the-art [transformer](https://en.wikipedia.org/wiki/Transformer_(machine_learning_model)) developed by Microsoft, published in this [ground-breaking paper](https://arxiv.org/abs/1912.13318).

The good news? It was licensed under the [MIT License](https://mit-license.org). We could use it! ...but how? it was _RESEARCH TIME!_

### Studying up

We had no experience in Machine learning. So we went to who created the whole thing: the infamous [_Attention is all you need_](https://arxiv.org/abs/1706.03762) paper, the birth of the trasnfomer model, developed by Google.

After lots of math and way too hard stuff, we went here: [Machine Learning Foundational Courses](https://developers.google.com/machine-learning/foundational-courses) and done'em all. After a week of learning, studying and trying to understand all the theory behind, we were (kinda) ready.

### The magic of fine-tuning

But we were fortunate! We already had a [pre-trained model](https://huggingface.co/microsoft/layoutlm-base-uncased) to use. We just needed to [fine-tune](https://en.wikipedia.org/wiki/Fine-tuning_(deep_learning)) it on our dataset!

To do it we needed a couple of things:

1. Lots of samples (aka: dataset)
2. Training script
3. Infrastructure with GPUs

#### 1. the dataset

Our dataset was a merely 150 samples, spanning over 32 different templates. We basically had less than 5 samples per template, on average. That was way too low.\

If you don't have the data... just create it!

Over a weekend, a [document generator](_INSERT LINK HERE_) was born: pick a template, set the position of the labels and generate away! \
Using the newly created tool we were able to generate hundreds or thousands of mock images filled with _realish_, consistent data.

#### 2. the script

Arguably the hardest part: gather, organize, package and send the data through the layers of the model. We used the most powerful tool ever created: google search. \
I found out that the benchmarks that the smart guys use to understand if a new model architecture is better than the old one, are actually _surprisingly_ close to the real thing.

Out task was basically this: Form Understanding in Noisy Scanned Documents [(FUNSD)](https://paperswithcode.com/dataset/funsd).
What did we find? An endless number of guides on how to benchmark the task! \
After reading dozens of [python jupyter](https://docs.jupyter.org/en/latest/start/index.html) notebooks (e.g. [this](https://github.com/NielsRogge/Transformers-Tutorials/blob/master/LayoutLM/Fine_tuning_LayoutLMForTokenClassification_on_FUNSD.ipynb)), we had our own fine-tuning [script](_INSERT LINK HERE_).

#### 3. the machine

Turns out that computing power is free if you ask kindly and visit [Google Colab](https://colab.research.google.com). Of course it wasn't suitable as a long-term solution, but coverd all of our needs for now: a ready to use python environment than run on a virtual machine with an [Nvidia T4](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/tesla-t4/t4-tensor-core-datasheet-951643.pdf) GPU.

### The roadblocks

We had the model downloaded. We had the data figured out. We had the script ready to train. We had the gpu. \
But we also had lots of problems to solve:

1. token limit
2. overfitting

After a (kinda) successful first run on a template, we noticed that the results were being generated only on the first half of the page. \
The poblem was the [tokenizer](https://huggingface.co/docs/transformers/model_doc/layoutlm#transformers.LayoutLMTokenizer) and more importantly the model limitations (here's the [issue](https://github.com/huggingface/transformers/issues/19190)).

The second problem was the quality of the data: we had too few examples all too similar. This meant the the model did not generalized well and overfitted our small dataset. Meaning: the performance were poor...

### A detour in the land of Classification

While we were trying to solve our problems, we decided to try first a simpler task: _Classification_.
We needed to know the path was right and that our efforts were being spent in the right direction.
For the classification we experimented with [LayoutLMv2](https://arxiv.org/abs/2012.14740)
Thanks to the experience accumulated trying to tackle the extraction task, it only took a couple of days to have everything ready.

And after a couple of hours of training, a working model was born: a classification LayoutLM transfomer which correctly classified 32 different templates.

We quickly wrote a [FastAPI](https://fastapi.tiangolo.com) implementation, exposed an inference endpoint, and hosted it on Azure.

A working demo was a breath of fresh air, more so after weeks of endless research.
A quick [presentation](https://www.dropbox.com/scl/fi/xbsar9a0znatikteprysx/Document-Understanding-with-Transformers-Unveiling-the-Power-of-LayoutLM.pdf?rlkey=52xz0j999dw8mr5jirv752r8y&dl=0) was written, and a [talk](https://caraservices.sharepoint.com/:v:/s/CARAServicesGmbH/Efat2OQajMNNgJ1SFKIad8wBH-kylyoaGcpxCsE4gEd3CQ?e=q6kABt) scheduled.

## The next steps

_In progress..._
