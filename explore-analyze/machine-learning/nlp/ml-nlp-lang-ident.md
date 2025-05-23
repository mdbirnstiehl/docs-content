---
mapped_pages:
  - https://www.elastic.co/guide/en/machine-learning/current/ml-nlp-lang-ident.html
applies_to:
  stack: ga
  serverless: ga
products:
  - id: machine-learning
---

# Language identification [ml-nlp-lang-ident]

{{lang-ident-cap}} enables you to determine the language of text.

A {{lang-ident}} model is provided in your cluster, which you can use in an {{infer}} processor of an ingest pipeline by using its model ID (`lang_ident_model_1`). For an example, refer to [*Add NLP {{infer}} to ingest pipelines*](ml-nlp-inference.md).

The longer the text passed into the {{lang-ident}} model, the more accurately the model can identify the language. It is fairly accurate on short samples (for example, 50 character-long streams) in certain languages, but languages that are similar to each other are harder to identify based on a short character stream. If there is no valid text from which the identity can be inferred, the model returns the special language code `zxx`. If you prefer to use a different default value, you can adjust your ingest pipeline to replace `zxx` predictions with your preferred value.

{{lang-ident-cap}} takes into account Unicode boundaries when the feature set is built. If the text has diacritical marks, then the model uses that information for identifying the language of the text. In certain cases, the model can detect the source language even if it is not written in the script that the language traditionally uses. These languages are marked in the supported languages table (see below) with the `Latn` subtag. {{lang-ident-cap}} supports Unicode input.

## Supported languages [ml-lang-ident-supported-languages] 

The table below contains the ISO codes and the English names of the languages that {{lang-ident}} supports. If a language has a 2-letter `ISO 639-1` code, the table contains that identifier. Otherwise, the 3-letter `ISO 639-2` code is used. The `Latn` subtag indicates that the language is transliterated into Latin script.

| Code | Language | Code | Language | Code | Language |
| --- | --- | --- | --- | --- | --- |
| af | Afrikaans | hr | Croatian | pa | Punjabi |
| am | Amharic | ht | Haitian | pl | Polish |
| ar | Arabic | hu | Hungarian | ps | Pashto |
| az | Azerbaijani | hy | Armenian | pt | Portuguese |
| be | Belarusian | id | Indonesian | ro | Romanian |
| bg | Bulgarian | ig | Igbo | ru | Russian |
| bg-Latn | Bulgarian | is | Icelandic | ru-Latn | Russian |
| bn | Bengali | it | Italian | sd | Sindhi |
| bs | Bosnian | iw | Hebrew | si | Sinhala |
| ca | Catalan | ja | Japanese | sk | Slovak |
| ceb | Cebuano | ja-Latn | Japanese | sl | Slovenian |
| co | Corsican | jv | Javanese | sm | Samoan |
| cs | Czech | ka | Georgian | sn | Shona |
| cy | Welsh | kk | Kazakh | so | Somali |
| da | Danish | km | Central Khmer | sq | Albanian |
| de | German | kn | Kannada | sr | Serbian |
| el | Greek, modern | ko | Korean | st | Southern Sotho |
| el-Latn | Greek, modern | ku | Kurdish | su | Sundanese |
| en | English | ky | Kirghiz | sv | Swedish |
| eo | Esperanto | la | Latin | sw | Swahili |
| es | Spanish, Castilian | lb | Luxembourgish | ta | Tamil |
| et | Estonian | lo | Lao | te | Telugu |
| eu | Basque | lt | Lithuanian | tg | Tajik |
| fa | Persian | lv | Latvian | th | Thai |
| fi | Finnish | mg | Malagasy | tr | Turkish |
| fil | Filipino | mi | Maori | uk | Ukrainian |
| fr | French | mk | Macedonian | ur | Urdu |
| fy | Western Frisian | ml | Malayalam | uz | Uzbek |
| ga | Irish | mn | Mongolian | vi | Vietnamese |
| gd | Gaelic | mr | Marathi | xh | Xhosa |
| gl | Galician | ms | Malay | yi | Yiddish |
| gu | Gujarati | mt | Maltese | yo | Yoruba |
| ha | Hausa | my | Burmese | zh | Chinese |
| haw | Hawaiian | ne | Nepali | zh-Latn | Chinese |
| hi | Hindi | nl | Dutch, Flemish | zu | Zulu |
| hi-Latn | Hindi | no | Norwegian |  |  |
| hmn | Hmong | ny | Chichewa |  |  |

## Example of {{lang-ident}} [ml-lang-ident-example]

In the following example, we feed the {{lang-ident}} trained model a short Hungarian text that contains diacritics and a couple of English words. The model identifies the text correctly as Hungarian with high probability.

```js
POST _ingest/pipeline/_simulate
{
   "pipeline":{
      "processors":[
         {
            "inference":{
               "model_id":"lang_ident_model_1", <1>
               "inference_config":{
                  "classification":{
                     "num_top_classes":5 <2>
                  }
               },
               "field_map":{
               }
            }
         }
      ]
   },
   "docs":[
      {
         "_source":{ <3>
            "text":"Sziasztok! Ez egy rövid magyar szöveg. Nézzük, vajon sikerül-e azonosítania a language identification funkciónak? Annak ellenére is sikerülni fog, hogy a szöveg két angol szót is tartalmaz."
         }
      }
   ]
}
```

1. ID of the {{lang-ident}} trained model.
2. Specifies the number of languages to report by descending order of probability.
3. The source object that contains the text to identify.

In the example above, the `num_top_classes` value indicates that only the top five languages (that is to say, the ones with the highest probability) are reported.

The request returns the following response:

```js
{
  "docs" : [
    {
      "doc" : {
        "_index" : "_index",
        "_type" : "_doc",
        "_id" : "_id",
        "_source" : {
          "text" : "Sziasztok! Ez egy rövid magyar szöveg. Nézzük, vajon sikerül-e azonosítania a language identification funkciónak? Annak ellenére is sikerülni fog, hogy a szöveg két angol szót is tartalmaz.",
          "ml" : {
            "inference" : {
              "top_classes" : [ <1>
                {
                  "class_name" : "hu",
                  "class_probability" : 0.9999936063740517,
                  "class_score" : 0.9999936063740517
                },
                {
                  "class_name" : "lv",
                  "class_probability" : 2.5020248433413966E-6,
                  "class_score" : 2.5020248433413966E-6
                },
                {
                  "class_name" : "is",
                  "class_probability" : 1.0150420723037688E-6,
                  "class_score" : 1.0150420723037688E-6
                },
                {
                  "class_name" : "ga",
                  "class_probability" : 6.67935962773335E-7,
                  "class_score" : 6.67935962773335E-7
                },
                {
                  "class_name" : "tr",
                  "class_probability" : 5.591166324774555E-7,
                  "class_score" : 5.591166324774555E-7
                }
              ],
              "predicted_value" : "hu", <2>
              "model_id" : "lang_ident_model_1"
            }
          }
        },
        "_ingest" : {
          "timestamp" : "2020-01-22T14:25:14.644912Z"
        }
      }
    }
  ]
}
```

1. Contains scores for the most probable languages.
2. The ISO identifier of the language with the highest probability.

## Further reading [ml-lang-ident-readings] 

* [Multilingual search using {{lang-ident}} in {{es}}](https://www.elastic.co/blog/multilingual-search-using-language-identification-in-elasticsearch)
