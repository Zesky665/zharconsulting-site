
### What is Chroma db?
ChromaDB is a open-source vector database, focused on RAG. It comes with support for LLM frameworks like LangChain and HuggingFace. It comes with embedding, text search, vector search, document storage and multimodal search built-in.  It can also run entirely in memory, in case you want very fast retrieval. 
### What is RAG?
RAG stands for Retrieval Augmented Generation, in a nutshell it's a process where a vector database is attached to a LLM client. This allows the client to convert the prompt into an vector, which it can use to look for relevant information in the vector database. The results are then added to the prompt as context. This allows the LLM to access relevant information without needing to fine tune with a new dataset, it also focuses the responses preventing hallucinations. 
### Why not [insert tool here]?
The other tools used for RAGs are Weaviate, Pinecone. 
Both of these are great solutions for enterprise solutions, like if you need a solution that can scale to billions of vectors. For our purpose that is overkill. 
There is also the small problem of them being only available as a cloud solution, you can't run the locally. 
### What are we using it for?
We are going to be adding a functionality to our local LLM, it will be able to ingest documents placed into a subfolder and add them to a local chromeDB. We can then use to the add RAG to our local LLM.
### How to implement it?
To implement it we need to do a couple of things, we need to set up a way to ingest data into the vector db and then we need to set up a way to query the database with each query. 

#### Embedding The Data
Ingesting data into a vector database is referred to as embedding, because we need to pass the data through an embedding stage before it can properly be stored inside of a vector db. 

This is done using an embedding model which is a type of LLM as well, it takes text and outputs a vector that contains the semantic meaning of the text. More specifically, it gives what the model thinks is the semantic meaning. The accuracy depends on the model, which just like generative LLMs depends on the quality of the training data and the size of the model. 

For this example we are going to be using the default from chrome db, [all-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2). This is one of the smallest ones at around 300mb. It's fast, efficient and decently good at generating embeddings. In cases where we need a production level of accuracy we would switch to another more heavyweight model like [E5-base](https://huggingface.co/intfloat/e5-base).

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


### How well does it work?

### Full Code
