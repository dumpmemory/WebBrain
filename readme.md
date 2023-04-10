# WebBrain: Learning to Generate Factually Correct Articles for Queries by Grounding on Large Web Corpus


## Abstract

In this paper, we introduce a new NLP task -- generating short factual articles for queries by mining supporting evidence from the Web. In this task, called WebBrain, the ultimate goal is to generate a fluent, informative, and factually-correct short article (e.g., a Wiki article) for a factual query unseen in Wikipedia. To enable experiments on WebBrain, we construct a large-scale dataset WebBrain-Raw by extracting English Wikipedia articles and their crawlable Wiki references. WebBrain-Raw is ten times larger than the previous biggest peer dataset, which can %We believe that WebBrain-Raw would greatly benefit the research community. Besides, we empirically analyze the performances of the current state-of-the-art NLP techniques on WebBrain and introduce a new framework ReGen, which enhances the generation factualness by improved evidence retrieval and task-specific pre-training for generation. xperiment results show that ReGen outperforms all baselines in both automatic and human evaluations.

**To access to WebBrain datasets, please go to [this repo](https://github.com/qhjqhj00/WebBrain-Data).**
## Requirements
- Python == 3.7
- PyTorch == 1.8.0
- Transformers >= 4.20

## Usage

### Retriever training

```
cd retriever
sh train.sh
```

### Retriever evaluate
Modify the checkpoint path in retriever/eval.py 

```
cd retriever
python eval.py
```

### Corpus index 

```
cd retriever
sh index.sh
```

### Generator training
Step into the generator directory, and run the following script:
```
python /cache/code/run_model_train.py \
    --data_dir /cache/data/ \
    --save_ckpt_name generate.pt \
    --train_data_name data.train \
    --test_data_name data.valid \
    --per_gpu_train_batch_size 4 \
    --per_gpu_eval_batch_size 4 \
    --output_dir /cache/output/models/ \
    --result_dir /cache/output/result/ \
    --model_name_or_path /cache/pretrained_model/ \
    --config_name /cache/pretrained_model/ \
    --tokenizer_name /cache/pretrained_model/ \
    --learning_rate 5e-5 \
    --num_train_epochs 5
```
### Data format:
WebBrain-Raw contains 154 chunk files, which are in jsonline format.
Each line of data in WebBrain-Raw is in the following format:
```
{
   "url":"wiki_url",
   "title": "wiki_title"
   "text":"sentence_a <a href=\"wiki_hyperlink\">wiki_entry</a> sentence_b[1].
           <h2> section_title </h2> sentence_c.[2]"
   "text":"wiki_content",
   "references":[
      {
         "cite_id":"[1]",
         "title":"ref_title",
         "url":"ref_url",
         "text": "ref_content"
      },
      ...
   ]
}
```
For the Wiki pages, we keep necessary html tags to identify the Wiki section and the Wiki entry. The Wiki entry refers to the internal links to other Wiki page. 

WebBrain-R contains four files: train.tsv / dev.tsv / test.tsv and corpus.jsonl. 
The first three files are in the same format:
```
qid\tquery\tpositive_passage_id\tnegative_passage1_id\t...\n
```
And data in corpus.jsonl are in the fowllowing format:
```
{"id": "passage_id", "content": "passage_content"}
```

WebBrain-G contains train / dev / test files, which are in the following format:

```
[title] wiki_title [ref] [ref_id] ref_title ref_content [SPLIT] ... [SPLIT] target_text 
```
where we append the Wiki title to the front of each reference, merge all references and the target text (label) with a special token [SPLIT].
