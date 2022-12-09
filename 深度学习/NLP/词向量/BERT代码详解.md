# BERT详解

[参考地址](https://zhuanlan.zhihu.com/p/112655246)

```python3
bert_path = "bert_model/"    # 该文件夹下存放三个文件（'vocab.txt', 'pytorch_model.bin', 'config.json'）
tokenizer = BertTokenizer.from_pretrained(bert_path)   # 初始化分词器
```

```python3
		# encode_plus会输出一个字典，分别为'input_ids', 'token_type_ids', 'attention_mask'对应的编码
        # 根据参数会短则补齐，长则切断
        encode_dict = tokenizer.encode_plus(text=title, max_length=maxlen, 
                                            padding='max_length', truncation=True)
```