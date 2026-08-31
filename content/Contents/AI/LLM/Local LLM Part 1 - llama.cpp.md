---
dg-publish: true
aliases:
  - llama.cpp tutorial
  - local llm tutorial
  - llama.cpp + Mistral
  - How to run Mistral locally
tags:
  - AI
  - Python
  - LLM
  - Local_LLM
---

### What is llama.cpp?
Llama.cpp is an open source library that performs inference on large language models. It's much lighter than frameworks like Ollama or LM Studio. 

### What is it used for?
For running LLM models on consumer hardware. It uses quantized models, which massively reduces the memory requirements, without significantly reducing quality. This allows people to run LLM locally, which is good for experimentation and for commercial applications where privacy is a major concern.

### Why not [insert other tool]?
Because the point of this exercise it to learn about LLMs by doing it "the hard way". The more friction we encounter the more we can learn. 

### What are we going to build?
For now the only thing we are going to do is set up llama.cpp locally and prompt it a couple of times to make sure it works. We will make further improvements in the following parts, such as a UI, sessions and RAGs.

### How to install
I use a M3 Macbook, so I will be following the instructions for that system. I will provide links to the instructions for each of the operating systems. 
	- [Windows](https://github.com/ggml-org/llama.cpp/releases)aka *just download the latest exe and run it.*
	- [Linux](https://github.com/ggml-org/llama.cpp/blob/master/docs/install.md)
	- [Mac](https://github.com/abetlen/llama-cpp-python/blob/main/docs/install/macos.md)

### Which model to use?
For local LLMs, the models used are ones big enough to be useful and small enough to fit into memory. There is a class of models called Quantized models which fit this niche perfectly. They have been modified in such a way that they use a lot less memory at the cost of some quality loss. This comes in degrees marked as precision variants, which are measured in bits. There range from 2 to 16, with 2 being the most quantized and 16 being the least. 

Other than being quantized, we also need to pay attention to how many parameters a model has. The number of parameters is a proxy for knowing how much "knowledge" the model contains, the more parameters the "smarter" the model is. Naturally we want the highest number possible, however we still need to fit all of this into memory. 

The ones I recommend are llama-2-7b and Mistral-7b, they are big enough to be somewhat useful and small enough to run on most modern computers.

There are many sizes of each, choose the biggest one you can run, the bigger the model the better the quality. 
- [Llama](https://huggingface.co/TheBloke/Llama-2-7B-GGUF)
- [Mistral](https://huggingface.co/TheBloke/Mistral-7B-v0.1-GGUF)

I chose Mistral, mostly because I liked the name more. It sounds like a DnD character. 

### How to prompt
Create a Jupyter notebook and add the following code. 

1. Install the necessary packages. 
```bash
pip install llama-cpp-python sentence-transformers numpy torch pandas
```
2. Import the libraries
```python
from llama_cpp import Llama
```
3. Connect to the model. The model file is in the same directory as the notebook.
```python
# Load the model
model_path = "mistral-7b-instruct-v0.2.Q5_K_M.gguf"

llm = Llama(
	model_path=model_path,
	n_ctx=2048,
	n_gpu_layers=30,
	verbose=False
)
```
4. Run a prompt, make sure to be very specific about what type of response you expect. Otherwise the model will ramble until it hits the token limit. 
```python
# Generate response
input = "What is artificial intelligence?"
prompt = f"Be extremely concise. Answer in one sentence only: {input}"

response = llm(
	prompt,
	max_tokens=50,
	temperature=0.5,
	top_p=0.9,
	stop=["</s>", "Human:", "User:"],
	echo=False
)
```
5. Check response
```python
# Print the response
print(response["choices"][0]["text"])
```

The result should look something like this:
```bash
Artificial intelligence is a branch of computer science that enables systems to perform tasks that typically require human intelligence, such as visual perception, speech recognition, decision-making, and language translation.
```