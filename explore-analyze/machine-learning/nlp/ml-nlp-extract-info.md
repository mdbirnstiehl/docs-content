---
mapped_pages:
  - https://www.elastic.co/guide/en/machine-learning/current/ml-nlp-extract-info.html
applies_to:
  stack: ga
  serverless: ga
products:
  - id: machine-learning
---

# Extract information [ml-nlp-extract-info]

These NLP tasks enable you to extract information from your unstructured text:

* [Named entity recognition](#ml-nlp-ner)
* [Fill-mask](#ml-nlp-mask)
* [Question answering](#ml-nlp-question-answering)

## Named entity recognition [ml-nlp-ner] 

The named entity recognition (NER) task can identify and categorize certain entities - typically proper nouns - in your unstructured text. Named entities usually refer to objects in the real world such as persons, locations, organizations, and other miscellaneous entities that are consistently referenced by a proper name.

NER is a useful tool to identify key information, add structure and gain insight into your content. It’s particularly useful while processing and exploring large collections of text such as news articles, wiki pages or websites. It makes it easier to understand the subject of a text and group similar pieces of content together.

In the following example, the short text is analyzed for any named entity and the model extracts not only the individual words that make up the entities, but also phrases, consisting of multiple words.

```js
{
    "docs": [{"text_field": "Elastic is headquartered in Mountain View, California."}]
}
...
```

The task returns the following result:

```js
{
  "inference_results": [{
    ...
      entities: [
        {
          "entity": "Elastic",
          "class": "organization"
        },
        {
          "entity": "Mountain View",
          "class": "location"
        },
        {
          "entity": "California",
          "class": "location"
        }
      ]
    }
  ]
}
...
```

## Fill-mask [ml-nlp-mask] 

The objective of the fill-mask task is to predict a missing word from a text sequence. The model uses the context of the masked word to predict the most likely word to complete the text.

The fill-mask task can be used to quickly and easily test your model.

In the following example, the special word “[MASK]” is used as a placeholder to tell the model which word to predict.

```js
{
    docs: [{"text_field": "The capital city of France is [MASK]."}]
}
...
```

The task returns the following result:

```js
...
{
  "predicted_value": "Paris"
  ...
}
...
```

## Question answering [ml-nlp-question-answering] 

The question answering (or extractive question answering) task makes it possible to get answers to certain questions by extracting information from the provided text.

The model tokenizes the string of – usually long – unstructured text, then it attempts to pull an answer for your question from the text. The logic is shown by the following examples:

```js
{
    "docs": [{"text_field": "The Amazon rainforest (Portuguese: Floresta Amazônica or Amazônia; Spanish: Selva Amazónica, Amazonía or usually Amazonia; French: Forêt amazonienne; Dutch: Amazoneregenwoud), also known in English as Amazonia or the Amazon Jungle, is a moist broadleaf forest that covers most of the Amazon basin of South America. This basin encompasses 7,000,000 square kilometres (2,700,000 sq mi), of which 5,500,000 square kilometres (2,100,000 sq mi) are covered by the rainforest. This region includes territory belonging to nine nations. The majority of the forest is contained within Brazil, with 60% of the rainforest, followed by Peru with 13%, Colombia with 10%, and with minor amounts in Venezuela, Ecuador, Bolivia, Guyana, Suriname and French Guiana. States or departments in four nations contain "Amazonas" in their names. The Amazon represents over half of the planet's remaining rainforests, and comprises the largest and most biodiverse tract of tropical rainforest in the world, with an estimated 390 billion individual trees divided into 16,000 species."}],
    "inference_config": {"question_answering": {"question": "Which name is also used to describe the Amazon rainforest in English?"}}
}
...
```

The answer is shown by the object below:

```js
...
{
  "predicted_value": "Amazonia"
  ...
}
...
```
