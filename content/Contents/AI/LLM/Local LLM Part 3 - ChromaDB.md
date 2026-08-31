---
dg-publish: true
aliases:
  - ChromaDB Tutorial
  - RAG with ChromaDB
  - Mistral and ChromaDB
  - LLM with RAG part 3
tags:
  - AI
  - LLM
  - Mistral
  - Local_LLM
---


### What is ChromaDB?
ChromaDB is a open-source vector database, focused on RAG. It comes with support for LLM frameworks like LangChain and HuggingFace. It comes with embedding, text search, vector search, document storage and multimodal search built-in.  It can also run entirely in memory, in case you want very fast retrieval. 
### What is RAG?
RAG stands for Retrieval Augmented Generation, in a nutshell it's a process where a vector database is attached to a LLM client. This allows the client to convert the prompt into an vector, which it can use to look for relevant information in the vector database. The results are then added to the prompt as context. This allows the LLM to access relevant information without needing to fine tune with a new dataset, it also focuses the responses preventing hallucinations. 
### Why not [insert tool here]?
The other tools used for RAGs are Weaviate, Pinecone. 
Both of these are great solutions for enterprise solutions, like if you need a solution that can scale to billions of vectors. For our purpose that is overkill. 
There is also the small problem of them being only available as a cloud solution, you can't run the locally. 
### What are we using it for?
We are going to be adding a functionality to our local LLM, it will be able to ingest documents placed into a subfolder and add them to a local ChromaDB. We can then use to the add RAG to our local LLM.
### How to implement it?
To implement it we need to do a couple of things, we need to set up a way to ingest data into the vector db and then we need to set up a way to query the database with each query. 

#### Embedding The Data
Ingesting data into a vector database is referred to as embedding, because we need to pass the data through an embedding stage before it can properly be stored inside of a vector db. 

This is done using an embedding model which is a type of LLM as well, it takes text and outputs a vector that contains the semantic meaning of the text. More specifically, it gives what the model thinks is the semantic meaning. The accuracy depends on the model, which just like generative LLMs depends on the quality of the training data and the size of the model. 

For this example we are going to be using the default from ChromaDB, [all-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2). This is one of the smallest ones at around 300mb. It's fast, efficient and decently good at generating embeddings. In cases where we need a production level of accuracy we would switch to another more heavyweight model like [E5-base](https://huggingface.co/intfloat/e5-base).

You can download both of these directly with chroma like this. 
```python
from sentence_transformers import SentenceTransformer

# Download the model
model = SentenceTransformer("E5-base")
```

To do the embedding we need to initialize the database. 
```python
import chromadb
from chromadb.config import Settings
from chromadb.utils import embedding_functions

# Initialize ChromaDB collection with persistence
if "chroma_client" not in st.session_state:

	# Create persistence directory
	PERSISTENCE_DIRECTORY = os.path.join(os.getcwd(), "chroma_db")
	
	os.makedirs(PERSISTENCE_DIRECTORY, exist_ok=True)
	
	# Initialize the persistent client
	st.session_state.chroma_client = chromadb.PersistentClient(
			path=PERSISTENCE_DIRECTORY,
			settings=Settings(
			anonymized_telemetry=False,
			allow_reset=True,
			is_persistent=True # Ensure full persistence mode
		)
	)

# Log the persistence configuration
st.session_state.persistence_dir = PERSISTENCE_DIRECTORY
print(f"ChromaDB initialized with persistence at: {PERSISTENCE_DIRECTORY}")


if "collection" not in st.session_state:
	# Use sentence transformers for embeddings
	sentence_transformer_ef = embedding_functions.SentenceTransformerEmbeddingFunction(
		model_name="all-MiniLM-L6-v2"
	)
	
	# Create or get collection
	st.session_state.collection = st.session_state.chroma_client.get_or_create_collection(
		name="document_collection",
		embedding_function=sentence_transformer_ef
	)
```

Then we need to add an interface for uploading the documents. For now we will only be supporting pdf and text files. We can add support for docx, md and other later as needed. 
```python
# Document uploader in the main interface
if st.session_state.get("model_loaded", False):
	with st.expander("📚 Upload Documents for RAG"):
	
		# Multiple file uploader
		uploaded_files = st.file_uploader("Upload PDF or text files", type=["pdf", "txt"], accept_multiple_files=True)
	
		# Display count of selected files
		if uploaded_files:
			st.write(f"Selected {len(uploaded_files)} file(s): 
			{', '.join([f.name for f in uploaded_files])}")
	
	col1, col2 = st.columns(2)
	
	with col1:
		# Common document type for batch uploads
		doc_type = st.selectbox("Document Type",
		["General", "Technical", "Financial", "Legal", "Medical", "Other"],
		help="Category of documents for metadata")
	
	with col2:
		# Chunk size control
		chunk_size = st.slider(
			"Chunk Size",
			min_value=500,
			max_value=2000,
			value=1000,
			step=100,
			help="Size of document chunks in characters"
		)
		
		# Chunk overlap control
		chunk_overlap = st.slider(
			"Chunk Overlap",
			min_value=0,
			max_value=500,
			value=200,
			step=50,
			help="Overlap between chunks in characters"
		)
	
	# Process all files button
	if uploaded_files and st.button("Add Documents to Knowledge Base"):
		# Initialize counters for summary
		total_files = len(uploaded_files)
		processed_files = 0
		total_chunks = 0
		failed_files = []
		
		# Progress bar
		progress_bar = st.progress(0)
		status_text = st.empty()
	
		# Process each file
		for i, uploaded_file in enumerate(uploaded_files):
			status_text.text(f"Processing file {i+1}/{total_files}: {uploaded_file.name}")
		
			try:
				# Process based on file type
				if uploaded_file.name.lower().endswith('.pdf'):
					text = parse_pdf(uploaded_file)
				else: # Assume text file
					text = uploaded_file.getvalue().decode('utf-8')
				
				# Create metadata
				metadata = {
					"source": uploaded_file.name,
					"name": uploaded_file.name,
					"type": doc_type,
					"date_added": time.strftime("%Y-%m-%d %H:%M:%S")
				}
				
				# Add to collection with custom chunk settings
				num_chunks = add_document_to_collection(
					text, 
					metadata, 
					chunk_size, 
					chunk_overlap
				)
				
				total_chunks += num_chunks
				processed_files += 1
			
			except Exception as e:
				failed_files.append(f"{uploaded_file.name}: {str(e)}")
			
			# Update progress
			progress_bar.progress((i + 1) / total_files)
		
	# Show summary
	if failed_files:
		st.warning(f"Processed {processed_files} of {total_files} files. Created {total_chunks} chunks total.")
		st.error("Some files failed to process:")
		
		for fail in failed_files:
			st.write(f"- {fail}")
	
	else:
		st.success(f"Successfully processed all {total_files} files! Created {total_chunks} chunks total.")
```
The slider values represent the size and overlap between chunks, setting them to be smaller is better if you need to narrow onto small parts of a document. Larger ones are better if the info you need comes with more context. 

Now the only thing left is the code for the ingestion. 
```python
# Function to parse PDF files
def parse_pdf(uploaded_file):
	with tempfile.NamedTemporaryFile(delete=False, suffix=".pdf") as temp_file:
		temp_file.write(uploaded_file.getvalue())
		temp_path = temp_file.name
		
	reader = pypdf.PdfReader(temp_path)
	text = ""
	
	for page in reader.pages:
		text += page.extract_text() + "\n"
		
	os.unlink(temp_path)
	return text

# Function to split text into chunks
def split_text(text: str, chunk_size: int = 1000, overlap: int = 200) -> List[str]:
	chunks = []
	
	for i in range(0, len(text), chunk_size - overlap):
		chunk = text[i:i + chunk_size]
		
		if len(chunk) < 100: # Skip very small chunks
			continue
		chunks.append(chunk)
		
	return chunks
```

This is what the interface looks like when I run it on my machine. 

<iframe src="https://giphy.com/embed/hAw2lLmuRCdL0uXgEI" width="704" height="400
" style="" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
### How well does it work?
It gave decently accurate answers based on the date I provided, I have only tested using smaller files. I don't know how well it will perform with larger files. 

So let's check. I'll feed it the entirety of "The Fundamentals of Data Engineering" by Joe Reis. 

The answers weren't bad.
![LLM_CORRECT_RESPONSE](https://i.imgur.com/HmGH4ZX.png)
Mostly.
![LLM PUFFIN RESPONSE](https://i.imgur.com/ZEJXFoL.png)
### Full Code
Contains some extra feature and bug fixes. 
```python
"""
Streamlit interface for local LLM using llama.cpp with ChromaDB RAG (Persistent Storage)

Usage:
    streamlit run llm_streamlit_app.py

Requirements:
    pip install streamlit llama-cpp-python chromadb sentence-transformers pypdf
"""

import streamlit as st
from llama_cpp import Llama
import chromadb
from chromadb.config import Settings
from chromadb.utils import embedding_functions
import time
import os
import re
import tempfile
import uuid
from typing import List, Dict, Any, Optional
import pypdf

# Set page config
st.set_page_config(
    page_title="Local LLM Chat with RAG", 
    page_icon="🦙",
    layout="wide"
)

# Use custom CSS to disable KaTeX rendering
st.markdown("""
<style>
.katex { display: none !important; }
</style>
""", unsafe_allow_html=True)

# App title and description
st.title("Chat with Local LLM + RAG")
st.markdown("Powered by llama.cpp and ChromaDB")

# Initialize session state for chat history and hidden system context
if "messages" not in st.session_state:
    st.session_state.messages = []

# Initialize system context (not shown in chat history)
if "system_context" not in st.session_state:
    st.session_state.system_context = ""

# Initialize ChromaDB collection with persistence
if "chroma_client" not in st.session_state:
    # Create persistence directory
    PERSISTENCE_DIRECTORY = os.path.join(os.getcwd(), "chroma_db")
    os.makedirs(PERSISTENCE_DIRECTORY, exist_ok=True)
    
    # Initialize the persistent client
    st.session_state.chroma_client = chromadb.PersistentClient(
        path=PERSISTENCE_DIRECTORY,
        settings=Settings(
            anonymized_telemetry=False,
            allow_reset=True,
            is_persistent=True  # Ensure full persistence mode
        )
    )
    
    # Log the persistence configuration
    st.session_state.persistence_dir = PERSISTENCE_DIRECTORY
    print(f"ChromaDB initialized with persistence at: {PERSISTENCE_DIRECTORY}")

if "collection" not in st.session_state:
    # Use sentence transformers for embeddings
    sentence_transformer_ef = embedding_functions.SentenceTransformerEmbeddingFunction(
        model_name="all-MiniLM-L6-v2"
    )
    
    # Create or get collection
    st.session_state.collection = st.session_state.chroma_client.get_or_create_collection(
        name="document_collection",
        embedding_function=sentence_transformer_ef
    )

# Function for cleaning text before embedding
def clean_text(text):
    """Clean text before embedding to remove problematic formatting"""
    # Remove extra whitespace
    text = re.sub(r'\s+', ' ', text).strip()
    
    # Remove or normalize special characters
    text = re.sub(r'[^\w\s.,;:!?()\[\]{}\'""-]', ' ', text)
    
    # Normalize dashes and hyphens
    text = re.sub(r'[–—−]', '-', text)
    
    return text

# Function to parse PDF files
def parse_pdf(uploaded_file):
    with tempfile.NamedTemporaryFile(delete=False, suffix=".pdf") as temp_file:
        temp_file.write(uploaded_file.getvalue())
        temp_path = temp_file.name
    
    reader = pypdf.PdfReader(temp_path)
    text = ""
    for page in reader.pages:
        text += page.extract_text() + "\n"
    
    os.unlink(temp_path)
    return text

# Function to split text into chunks
def split_text(text: str, chunk_size: int = 1000, overlap: int = 200) -> List[str]:
    chunks = []
    for i in range(0, len(text), chunk_size - overlap):
        chunk = text[i:i + chunk_size]
        if len(chunk) < 100:  # Skip very small chunks
            continue
        chunks.append(chunk)
    return chunks

# Function to add documents to the ChromaDB collection
def add_document_to_collection(text: str, metadata: Dict[str, Any] = None, chunk_size: int = 1000, chunk_overlap: int = 200):
    # Skip empty documents
    if not text or len(text.strip()) < 100:
        return 0
        
    # Clean text before chunking
    text = clean_text(text)

    # Create chunks with the specified size and overlap
    chunks = split_text(text, chunk_size, chunk_overlap)
    
    # Generate unique IDs
    ids = [str(uuid.uuid4()) for _ in chunks]
    
    # Add chunk index to metadata for each chunk
    metadatas = []
    for i, _ in enumerate(chunks):
        # Start with base metadata
        chunk_metadata = {}
        if metadata:
            chunk_metadata = metadata.copy()
        
        # Add chunk-specific metadata
        chunk_metadata["chunk_index"] = i
        chunk_metadata["total_chunks"] = len(chunks)
        
        metadatas.append(chunk_metadata)
    
    # Add to collection
    if chunks:
        st.session_state.collection.add(
            documents=chunks,
            ids=ids,
            metadatas=metadatas
        )
        # Note: With PersistentClient, persistence happens automatically
        # No need to call .persist() explicitly
    
    return len(chunks)

# Function to retrieve relevant documents
def retrieve_context(query: str, k: int = 3) -> str:
    results = st.session_state.collection.query(
        query_texts=[query],
        n_results=k
    )
    
    if not results["documents"]:
        return ""
    
    # Combine retrieved documents into a context string
    context = "\n\n".join([doc for doc in results["documents"][0]])
    return context

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
                               "You are a helpful, concise assistant. Provide accurate and helpful information. Use the retrieved context to inform your answers when relevant.",
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
    
    # RAG Settings
    st.header("RAG Settings")
    
    use_rag = st.checkbox("Enable RAG", value=True, 
                         help="Use Retrieval-Augmented Generation for better responses")
    
    if use_rag:
        num_chunks = st.slider("Number of Retrieved Chunks", min_value=1, max_value=10, value=3, step=1,
                              help="Number of context chunks to retrieve for each query")
        
        # Make sure this is stored in session state
        show_context = st.checkbox("Show Retrieved Context", value=False,
                                 help="Display the retrieved context in the chat")
        
        # Save the setting to session state
        st.session_state.show_retrieved_context = show_context
    
    # Load/Reload model button
    load_button = st.button("Load/Reload Model")

    # Document uploader in the main interface
if st.session_state.get("model_loaded", False):
    with st.expander("📚 Upload Documents for RAG"):
        # Multiple file uploader
        uploaded_files = st.file_uploader("Upload PDF or text files", type=["pdf", "txt"], accept_multiple_files=True)
        
        # Display count of selected files
        if uploaded_files:
            st.write(f"Selected {len(uploaded_files)} file(s): {', '.join([f.name for f in uploaded_files])}")
        
        col1, col2 = st.columns(2)
        with col1:
            # Common document type for batch uploads
            doc_type = st.selectbox("Document Type", 
                                   ["General", "Technical", "Financial", "Legal", "Medical", "Other"],
                                   help="Category of documents for metadata")
        
        with col2:
            # Chunk size control
            chunk_size = st.slider("Chunk Size", 
                                 min_value=500, 
                                 max_valu
```