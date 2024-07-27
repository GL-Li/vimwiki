## Python

### LLM

#### encoding-decoding with a tokenizer
The code demonstrates how to encode a sentence into tokens and decode the tokens into a readable sentence.

```python
# Use the tokenizer from the same model 
from transformers import AutoTokenizer
model_name = 'google/flan-t5-base'
tokenizer = AutoTokenizer.from_pretrained(model_name, use_fast=True)

sentence = "What time is it, Tom?"
sentence_encoded = tokenizer(sentence, return_tensors='pt')   # pt for pythorch
    # {'input_ids': tensor([[ 363,   97,   19,   34,    6, 3059,   58,    1]]), 
    # 'attention_mask': tensor([[1, 1, 1, 1, 1, 1, 1, 1]])}

sentence_decodewd = tokenizer.decode(
        sentence_encoded["input_ids"][0], 
        skip_special_tokens=True
)
    # "What time is it, Tom?"
```

#### inference with a model
`model.generate()` takes input tokens and generate readable text.

```python
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM
model_name = 'google/flan-t5-base'
tokenizer = AutoTokenizer.from_pretrained(model_name, use_fast=True)
model = AutoModelForSeq2SeqLM.from_pretrained(model_name)

prompt = "who is the 30th president of the united states"
inputs = tokenizer(prompt, return_tensors='pt')   

output = tokenizer.decode(
    model.generate(
        inputs["input_ids"], 
        max_new_tokens=50,
    )[0], 
    skip_special_tokens=True
)
```


#### configuration for model inference
`model.generate()` has many other parameters that can be specified in `GenerationConfig` class.

- max_new_tokens: max number of new tokens that are not in the prompt, used to control output text length. 
- do_sample: choose tokens randomly based on predicted probability if True, choose the most likely token if False. 
- temperature: control the randomness of sampling process. Default to 1.0, which uses models predictions on tokens. Higher temperature makes the sampling more random.
- num_beams: number of paths to generate output. The more the better output but more costly.

```python
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM, GenerationConfig
model_name = 'google/flan-t5-base'
tokenizer = AutoTokenizer.from_pretrained(model_name, use_fast=True)
model = AutoModelForSeq2SeqLM.from_pretrained(model_name)

prompt = "who is the 30th president of the united states"
inputs = tokenizer(prompt, return_tensors='pt')   

generation_config = GenerationConfig(
    max_new_tokens=50,
    do_sample=True,
    temperature=1.0
)
output = tokenizer.decode(
    model.generate(
        inputs["input_ids"], 
        generation_config=generation_config,
    )[0], 
    skip_special_tokens=True
)
```
