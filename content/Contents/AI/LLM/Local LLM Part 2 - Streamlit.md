---
dg-publish: true
aliases:
  - Streamlit Tutorial
  - Streamlit and LLM
tags:
  - AI
  - LLM
  - Business-Intelligence
  - Python
  - Local_LLM
---

### What is Streamlit?
Streamlit is an open source library which allows users to easily create UIs for data apps without needing to fiddle around with html, css or js. 

### What is it used for?
For creating UI interfaces for applications, the main ones being data apps and ML models. It also comes with a very simple method for sharing and deploying the interface. 

### Why not [insert other tool]?
We could use something like LM Studio if we wanted a out of the box solution for the local llm. 
This is a very valid choice if you just want to be up and running ASAP, I'm more interested in learning how to build apps.
Why not use FastAPI + react/vue? 
Because that would be overcomplicated for what I'm trying to build here. Also, I don't want to add js if not absolutely necessary. 

### What are we going to build?
A simple interface that will allow us to configure which llm we are using, change the parameters and interact with the model. 

### How to start?
Start by creating a file, streamlit can't run from inside of a notebook. 
Let's call it `llama_chat.py`. 

### Adding a simple chat window
To create a super simple chat window we first need to create a window. 
We can do that with the following code:
```python
import streamlit as st

# Set page config
st.set_page_config(
	page_title="Local LLM Chat",
	page_icon="🦙",
	layout="wide"
)

# App title and description
st.title("Chat with Local LLM")
st.markdown("Powered by llama.cpp")

# Chat input
prompt = st.chat_input("Message the LLM...", disabled=False)
```

Now if we run the script with `streamlit run llama_chat.py`we will see the following on our localhost:8501 page.
![Initial Screen](https://i.imgur.com/AFpQ6YY.png)

Of course this isn't connected to our model, or anything else, so it can't do anything right now.
### Adding the model
In order to communicate with the model via our streamlit interface we need to load it and send it the prompts. And then display the responses. 

Let's start by connecting to the model on app start.
```python
from llama_cpp import Llama

# Settings for the model
model_path = "models/mistral-7b-instruct-v0.2.Q5_K_M.gguf"
context_size = 512
max_tokens = 50
temperature = 0.5
top_p = 0.9
gpu_layers = 20

# Load model when app starts
if "llm" not in st.session_state and "model_path" in locals():
	try:
		with st.spinner("Loading model... This may take a moment."):
			# Clear existing model if it exists
			if "llm" in st.session_state:
				del st.session_state.llm
				
			# Load the model with selected parameters
			st.session_state.llm = Llama(
				model_path=model_path,
				n_ctx=context_size,
				n_gpu_layers=gpu_layers,
				verbose=False
				)
		
		st.sidebar.success(f"Model loaded successfully!")
		st.session_state.model_loaded = True
	except Exception as e:
		st.sidebar.error(f"Error loading model: {str(e)}")
		st.session_state.model_loaded = False
```
Ok, so this needs a bit of explaining.
Streamlit comes with a built in session_state object, which we can use out of the box.
It also comes with a sidebar object, which let's you add things to the left sidebar. 

We can put our new sidebar knowledge to use by adding all of the model configs to the sidebar, this way we will be able to adjust them as we go instead of having to reload every time we want to fiddle with the settings. 

Luckily streamlit makes this quite easy. 
```python
# Sidebar for model configuration
with st.sidebar:
	st.header("Model Settings")
	# Model path selection
	default_model_path = ""
	model_files = []
	
	# Check for models directory
	if os.path.exists("models"):
		model_files = [f for f in os.listdir("models") if f.endswith(".gguf")]

	if model_files:
		default_model_path = os.path.join("models", model_files[0])

	# Model selection
	if model_files:
		selected_model = st.selectbox("Select Model", model_files)
		model_path = os.path.join("models", selected_model)
	else:
		model_path = st.text_input("Model Path", default_model_path,help="Enter the path to your .gguf model file")
	
	# Create columns for settings
	col1, col2 = st.columns(2)

	with col1:
		temperature = st.slider("Temperature", min_value=0.0, max_value=2.0, value=0.7, step=0.1, help="Higher = more creative, Lower = more deterministic")

		context_size = st.slider("Context Size", min_value=512, max_value=8192, value=2048, step=512, help="Maximum context window size")

	with col2:
		max_tokens = st.slider("Max Tokens", min_value=10, max_value=4096, value=500, step=10, help="Maximum tokens to generate")

		top_p = st.slider("Top P", min_value=0.0, max_value=1.0, value=0.9, step=0.05, help="Nucleus sampling parameter")

	if use_gpu:
		gpu_layers = st.slider("GPU Layers", min_value=0, max_value=100, value=20, step=1, help="Number of layers to offload to GPU")
	else:
		gpu_layers = 0

	# Load/Reload model button
	load_button = st.button("Load/Reload Model")
```

It will look something like this.
![Chat UI with Model Settings](https://i.imgur.com/LwHgvzU.png)

At this point we are able to select from a new model, configure it and continue without having to close the app and run it again. 

The next step will be the most complicated one where we add the ability to communicate with the model. We'll make heavy use of the session_state object. 
### Adding sessions

We need to manage sessions for two simple reasons.
1. So we can display the chat history.
2. So that the model can make use of it's context window.

The context window is essentially the models short term memory, it's how it can remember the details of the conversation. 

Let's start by initializing the session.
```python
# Initialize session state for chat history and hidden system context
if "messages" not in st.session_state:
	st.session_state.messages = []

# Initialize system context (not shown in chat history)
if "system_context" not in st.session_state:
	st.session_state.system_context = ""
```

Then we need to create a function that can format the prompt and add it to the message history.
```python
# Function to format prompt with conversation history and hidden initial prompt
def format_prompt(messages, new_prompt=None):
	# Start with system instructions
	formatted_prompt = system_prompt + "\n\n"

	# Add hidden initial prompt if it exists
	if st.session_state.system_context:
		formatted_prompt += f"Human: {st.session_state.system_context}\nAssistant: I understand and will follow these instructions.\n\n"

	# Add visible message history
	for message in messages:
		if message["role"] == "user":
			formatted_prompt += f"Human: {message['content']}\n"
		else:
			formatted_prompt += f"Assistant: {message['content']}\n"

	# Add new prompt if provided
	if new_prompt:
		formatted_prompt += f"Human: {new_prompt}\nAssistant:"

	return formatted_prompt
```

Now that we have everything set up we can add the actual prompting functionality.
```python
# Display chat messages
for message in st.session_state.messages:
	with st.chat_message(message["role"]):
		st.write(message["content"])

# Chat input
prompt = st.chat_input("Message the LLM...", disabled=not st.session_state.get("model_loaded", False))

if prompt:
	# Add user message to chat history
	st.session_state.messages.append({"role": "user", "content": prompt})
	
	# Display user message
	with st.chat_message("user"):
		st.write(prompt)

	# Generate assistant response
	if "llm" in st.session_state:
		with st.chat_message("assistant"):
			# Initialize placeholder for streaming output
			response_placeholder = st.empty()
			full_response = ""
			# Format conversation history for context
			conversation = format_prompt(st.session_state.messages[:-1], prompt)
			# Generate the response with streaming display
			start_time = time.time()
			
			try:
				with st.spinner("Thinking..."):
					# Call the model
					response = st.session_state.llm(
						conversation,
						max_tokens=max_tokens,
						temperature=temperature,
						top_p=top_p,
						stop=["Human:", "\n\nHuman", "\n\nUser:"],
						stream=True,
					)
				
					# Process the streaming response
					for chunk in response:
						if chunk["choices"][0]["text"]:
							text_chunk = chunk["choices"][0]["text"]
							full_response += text_chunk
							response_placeholder.markdown(full_response)
					
					# Show completion time
					elapsed_time = time.time() - start_time
					st.caption(f"Generated in {elapsed_time:.2f} seconds")
				
			except Exception as e:
				st.error(f"Error generating response: {str(e)}")
				full_response = "I encountered an error. Please try again or adjust your settings."
				response_placeholder.markdown(full_response)

		# Add assistant response to chat history
		st.session_state.messages.append({"role": "assistant", "content": full_response})
```

The code here takes the inputed prompt, formats it, prompts the model, formats the response and then refreshes the chat history. 

The UI will now look like this, after you ask it a question or two. 
![Chat UI with session and conversation](https://i.imgur.com/lgJBjIT.png)
Thanks to the session, the model remembers the context from the previous interactions. 
### The full code
```python
import streamlit as st
from llama_cpp import Llama
import time
import os


# Set page config
st.set_page_config(
	page_title="Local LLM Chat",
	page_icon="🦙",
	layout="wide"
)

# App title and description
st.title("Chat with Local LLM")
st.markdown("Powered by llama.cpp")

  
# Initialize session state for chat history and hidden system context
if "messages" not in st.session_state:
	st.session_state.messages = []

# Initialize system context (not shown in chat history)
if "system_context" not in st.session_state:
	st.session_state.system_context = ""

# Sidebar for model configuration
with st.sidebar:

	st.header("Model Settings")
	
	# Model path selection
	default_model_path = ""
	model_files = []

	# Check for models directory
	if os.path.exists("models"):
		model_files = [f for f in os.listdir("models") if f.endswith(".gguf")]

	if model_files:
		default_model_path = os.path.join("models", model_files[0])

	# Model selection
	if model_files:
		selected_model = st.selectbox("Select Model", model_files)
		model_path = os.path.join("models", selected_model)
	else:
		model_path = st.text_input("Model Path", default_model_path,
	help="Enter the path to your .gguf model file")

	# Create columns for settings
	col1, col2 = st.columns(2)
	
	with col1:
		temperature = st.slider("Temperature", min_value=0.0, max_value=2.0, value=0.7, step=0.1,
		help="Higher = more creative, Lower = more deterministic")
		context_size = st.slider("Context Size", min_value=512, max_value=8192, value=2048, step=512,
		help="Maximum context window size")

	with col2:
		max_tokens = st.slider("Max Tokens", min_value=10, max_value=4096, value=500, step=10,
		help="Maximum tokens to generate")
		top_p = st.slider("Top P", min_value=0.0, max_value=1.0, value=0.9, step=0.05,
		help="Nucleus sampling parameter")
		
		# System prompt
		system_prompt = st.text_area("System Prompt",
		"You are a helpful, concise assistant. Provide accurate and helpful information.",
		help="Instructions for how the AI should behave")
		
		# Initial hidden prompt (not shown in chat history)
		initial_prompt = st.text_area("Initial Hidden Prompt",
		"This is a hidden prompt that will be sent at the beginning of each conversation but won't appear in the chat history.",
		help="This prompt is sent before any user messages but isn't displayed in the chat")
		
		# Apply initial prompt button
		if st.button("Apply Initial Prompt"):
			st.session_state.system_context = initial_prompt
			st.success("Initial prompt applied! It will be used in the conversation but not shown in the chat history.")
		
		# GPU acceleration
		use_gpu = st.checkbox("Use GPU Acceleration", value=False,
		help="Enable GPU acceleration if available")
		
		if use_gpu:
			gpu_layers = st.slider("GPU Layers", min_value=0, max_value=100, value=20, step=1,
		help="Number of layers to offload to GPU")
		else:
			gpu_layers = 0
		
	# Load/Reload model button
	load_button = st.button("Load/Reload Model")

  

# Function to format prompt with conversation history and hidden initial prompt
def format_prompt(messages, new_prompt=None):
	# Start with system instructions
	formatted_prompt = system_prompt + "\n\n"
	
	# Add hidden initial prompt if it exists
	if st.session_state.system_context:
		formatted_prompt += f"Human: {st.session_state.system_context}\nAssistant: I understand and will follow these instructions.\n\n"
	
	# Add visible message history
	for message in messages:
		if message["role"] == "user":
			formatted_prompt += f"Human: {message['content']}\n"
		else:
			formatted_prompt += f"Assistant: {message['content']}\n"
	
	# Add new prompt if provided
	if new_prompt:
		formatted_prompt += f"Human: {new_prompt}\nAssistant:"
	
	return formatted_prompt

  

# Load model when button is clicked or on first chat
if load_button or ("llm" not in st.session_state and "model_path" in locals()):
	try:
		with st.spinner("Loading model... This may take a moment."):

		# Clear existing model if it exists
		if "llm" in st.session_state:
		del st.session_state.llm

		# Load the model with selected parameters
		st.session_state.llm = Llama(
			model_path=model_path,
			n_ctx=context_size,
			n_gpu_layers=gpu_layers,
			verbose=False
		)

		st.sidebar.success(f"Model loaded successfully!")
		st.session_state.model_loaded = True

	except Exception as e:
		st.sidebar.error(f"Error loading model: {str(e)}")
		st.session_state.model_loaded = False

  
# Display chat messages
for message in st.session_state.messages:
	with st.chat_message(message["role"]):
		st.write(message["content"])

# Chat input
prompt = st.chat_input("Message the LLM...", disabled=not st.session_state.get("model_loaded", False))

if prompt:
	# Add user message to chat history
	st.session_state.messages.append({"role": "user", "content": prompt})

# Display user message
with st.chat_message("user"):
	st.write(prompt)

# Generate assistant response
if "llm" in st.session_state:
	with st.chat_message("assistant"):
		# Initialize placeholder for streaming output
		response_placeholder = st.empty()
		full_response = ""
		
		# Format conversation history for context
		conversation = format_prompt(st.session_state.messages[:-1], prompt)
		
		# Generate the response with streaming display
		start_time = time.time()

		try:
			with st.spinner("Thinking..."):
				# Call the model
				response = st.session_state.llm(
					conversation,
					max_tokens=max_tokens,
					temperature=temperature,
					top_p=top_p,
					stop=["Human:", "\n\nHuman", "\n\nUser:"],
					stream=True,
				)

				# Process the streaming response
				for chunk in response:
					if chunk["choices"][0]["text"]:
						text_chunk = chunk["choices"][0]["text"]
						full_response += text_chunk
						response_placeholder.markdown(full_response)

				# Show completion time
				elapsed_time = time.time() - start_time
				st.caption(f"Generated in {elapsed_time:.2f} seconds")
		except Exception as e:
			st.error(f"Error generating response: {str(e)}")
			full_response = "I encountered an error. Please try again or adjust your settings."
			response_placeholder.markdown(full_response)

	# Add assistant response to chat history
	st.session_state.messages.append({"role": "assistant", "content": full_response})

# Show instructions if model not loaded
if not st.session_state.get("model_loaded", False):
	st.info("👈 Please select a model and click 'Load/Reload Model' in the sidebar to start chatting")
```