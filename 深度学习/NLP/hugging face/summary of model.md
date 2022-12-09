# Summary of the model

该库中的所有模型属于以下几个分类：

- 自回归模型（[Autoregressive models](https://huggingface.co/transformers/model_summary.html#autoregressive-models)）
- 自编码模型（[Autoencoding models](https://huggingface.co/transformers/model_summary.html#autoencoding-models)）
- 序列到序列模型（[Sequence-to-sequence models](https://huggingface.co/transformers/model_summary.html#seq-to-seq-models)）
- 多模态模型（[Multimodal models](https://huggingface.co/transformers/model_summary.html#multimodal-models)）
- 基于检索的模型（[Retrieval-based models](https://huggingface.co/transformers/model_summary.html#retrieval-based-models)）

自回归模型在经典语言建模任务上进行了预训练：猜测下一个已读完所有先前标记的标记。常用文本生成。

通过以某种方式破坏输入令牌并尝试重建原始句子来对自动编码模型进行预训练。常用文本生成、文本分类、命名实体。

自动回归模型和自动编码模型之间的唯一区别在于模型的预训练方式。

序列到序列模型同时使用原始转换器的编码器和解码器进行翻译任务或通过将其他任务转换为序列到序列问题。常用于翻译、摘要、问答。

多峰模型将文本输入与其他类型的输入（例如图像）混合在一起，并且更特定于给定任务。

## 1. Autoregressive models

1. [Original GPT](https://huggingface.co/transformers/model_summary.html#original-gpt)
2. [GPT-2](https://huggingface.co/transformers/model_summary.html#gpt-2)
3. [CTRL](https://huggingface.co/transformers/model_summary.html#ctrl)
4. [Transformer-XL](https://huggingface.co/transformers/model_summary.html#transformer-xl)
5. [Reformer](https://huggingface.co/transformers/model_summary.html#reformer)
6. [XLNet](https://huggingface.co/transformers/model_summary.html#xlnet)

## 2.Autoencoding models

1. [BERT](https://huggingface.co/transformers/model_summary.html#bert)
2. [ALBERT](https://huggingface.co/transformers/model_summary.html#albert)
3. [RoBERT](https://huggingface.co/transformers/model_summary.html#roberta)
4. [DistilBERT](https://huggingface.co/transformers/model_summary.html#distilbert)
5. [ConvBERT](https://huggingface.co/transformers/model_summary.html#convbert)
6. [XLM](https://huggingface.co/transformers/model_summary.html#xlm)
7. [XLM-RoBERTa](https://huggingface.co/transformers/model_summary.html#xlm-roberta)
8. [FlauBERT](https://huggingface.co/transformers/model_summary.html#flaubert)
9. [ELECTRA](https://huggingface.co/transformers/model_summary.html#electra)
10. [Funnel Transformer](https://huggingface.co/transformers/model_summary.html#funnel-transformer)
11. [Longformer](https://huggingface.co/transformers/model_summary.html#longformer)

## 3. Sequence-to-sequence models

1. [BART](https://huggingface.co/transformers/model_summary.html#bart)
2. [Pegasus](https://huggingface.co/transformers/model_summary.html#pegasus)
3. [MarianMT](https://huggingface.co/transformers/model_summary.html#marianmt)
4. [T5](https://huggingface.co/transformers/model_summary.html#t5)
5. [MT5](https://huggingface.co/transformers/model_summary.html#mt5)
6. [MBart](https://huggingface.co/transformers/model_summary.html#mbart)
7. [ProphetNet](https://huggingface.co/transformers/model_summary.html#prophetnet)
8. [XLM-ProphetNet](https://huggingface.co/transformers/model_summary.html#xlm-prophetnet)

## 4. Multimodal models

[MMBT](https://huggingface.co/transformers/model_summary.html#mmbt)

## 5.Retrieval-based models

[DPR](https://huggingface.co/transformers/model_summary.html#dpr)

[RAG](https://huggingface.co/transformers/model_summary.html#rag)

## 6.More technical aspects

