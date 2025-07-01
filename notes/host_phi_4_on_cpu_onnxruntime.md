# Host Phi-4 on CPU Using onnxruntime

## Prerequisites

1. Windows OS
2. Install Python (3.12.x)
3. Install git (latest)
4. Install git-lfs (latest)
5. `pip install numpy` (2.1.1)
6. `pip install --pre onnxruntime-genai` (0.8.2)
7. `pip install argparse`

## Install Hugging Face CLI

This step can be skipped if you want to download models manually.

1. Install `pip install huggingface-hub[cli]`
2. Download model (for phi 3): `huggingface-cli download microsoft/Phi-3-mini-4k-instruct-onnx --include cpu_and_mobile/cpu-int4-rtn-block-32-acc-level-4/* --local-dir .`

However, I got the following error

```
ssl.SSLCertVerificationError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain
```

So I had to manually download phi model.

phi-4 CPU files are available here: https://huggingface.co/microsoft/Phi-4-mini-instruct-onnx/tree/main/cpu_and_mobile/cpu-int4-rtn-block-32-acc-level-4

## Test Phi

1. Create [test_phi.py](#test_phipy)
2. Open a cmd window, run `python test_phi.py --verbose --model "cpu_and_mobile-phi-4-mini\cpu-int4-rtn-block-32-acc-level-4"`

It prints

```
Loading model...
Model loaded
Tokenizer created

Input: <type your question here>
```

Sample queries:

- `Count from 1 to 10` (basic functionality)
- `What is the capital of France?` (knowledge test)
- `Write a Python function to calculate the Fibonacci sequence` (coding)

ref: https://techcommunity.microsoft.com/blog/educatordeveloperblog/getting-started-using-phi-3-mini-4k-instruct-onnx-for-text-generation-with-nlp-t/4136676

---

## Provide customised context

1. Manually download `all-MiniLM-L6-v2` model for sentence transformer by cloing the repo `https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2/tree/main`
2. Install `pip install sentence_transformers faiss` (sentence-transformers 4.1.0, faiss-cpu 1.11.0)
3. Create [test_phi_custom.py](#test_phi_custompy)
4. Run `python test_phi_custom.py --verbose --model "cpu_and_mobile/cpu-int4-rtn-block-32-acc-level-4" --kb "C:\my\path\to\markdown\folder"`

---

## Phi-4-Reasoning

1. Download Phi-4-reasonging from huggingface (https://huggingface.co/microsoft/Phi-4-reasoning-onnx/tree/main/cpu_and_mobile/cpu-int4-rtn-block-32-acc-level-4)
2. Open a cmd window, run `python test_phi.py --verbose --model "cpu_and_mobile_phi-4-reasoning\cpu-int4-rtn-block-32-acc-level-4`

## Files

### test_phi.py

```py
import onnxruntime_genai as og
import argparse
import time

def main(args):
    # If verbose mode is on, print loading model message
    if args.verbose: print("Loading model...")
    
    # If timings mode is on, initialize timing variables
    if args.timings:
        started_timestamp = 0
        first_token_timestamp = 0

    # Load the model
    model = og.Model(f'{args.model}')
    if args.verbose: print("Model loaded")
    
    # Initialize the tokenizer with the model
    tokenizer = og.Tokenizer(model)
    tokenizer_stream = tokenizer.create_stream()
    if args.verbose: print("Tokenizer created")
    
    # Print a newline for readability if verbose mode is on
    if args.verbose: print()
    
    # Create a dictionary of search options from the command line arguments
    search_options = {name:getattr(args, name) for name in ['do_sample', 'max_length', 'min_length', 'top_p', 'top_k', 'temperature', 'repetition_penalty'] if name in args}
    
    # Set a default max length if one is not provided
    if 'max_length' not in search_options:
        search_options['max_length'] = 2048

    # Define a template for the chat input
    chat_template = '<|user|>\n{input} <|end|>\n<|assistant|>'

    # Main loop: ask for input and generate responses
    while True:
        # Get user input
        text = input("Input: ")
        
        # If the input is empty, print an error message and continue to the next iteration
        if not text:
            print("Error, input cannot be empty")
            continue

        # If timings mode is on, record the start time
        if args.timings: started_timestamp = time.time()

        # Format the input with the chat template
        prompt = f'{chat_template.format(input=text)}'

        # Tokenize the input
        input_tokens = tokenizer.encode(prompt)

        # Set up the generator parameters
        params = og.GeneratorParams(model)
        # Remove the deprecated method call
        # params.try_graph_capture_with_max_batch_size(1)
        params.set_search_options(**search_options)
        
        # Create the generator with input tokens passed directly
        generator = og.Generator(model, params)
        if args.verbose: print("Generator created")
        
        # Print a message if verbose mode is on
        if args.verbose: print("Running generation loop ...")
        
        # If timings mode is on, initialize variables for the generation loop
        if args.timings:
            first = True
            new_tokens = []

        # Print the output prompt
        print()
        print("Output: ", end='', flush=True)

        try:
            generator.append_tokens(input_tokens)
            while not generator.is_done():
                generator.generate_next_token()

                new_token = generator.get_next_tokens()[0]
                print(tokenizer_stream.decode(new_token), end='', flush=True)
        except KeyboardInterrupt:
            print("  --control+c pressed, aborting generation--")

        print()
        del generator

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Run the chatbot script")
    parser.add_argument("--model", type=str, required=True, help="Path to the ONNX model file")
    parser.add_argument("--verbose", action="store_true", help="Enable verbose mode")
    parser.add_argument("--timings", action="store_true", help="Enable timings mode")
    args = parser.parse_args()
    main(args)
```

### test_phi_custom.py

```py
import onnxruntime_genai as og
import argparse
import time
from sentence_transformers import SentenceTransformer
import numpy as np
import os
import faiss

# 1. Create embeddings for your files
def create_knowledge_base(folder_path, chunk_size=1000, chunk_overlap=200, file_extensions=('.py', '.txt')):
    """Create a knowledge base from files in the specified folder."""
    # Load embedding model
    embed_model = SentenceTransformer(r'C:\Downloads\sentence-transformer\all-MiniLM-L6-v2')
    # manually clone the repo
    # `git clone https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2`
    # error `unable to access 'https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2/': SSL certificate problem: self-signed certificate in certificate chain`
    # git config --global http.sslVerify false
    # git config --global http.sslVerify true
    
    # Process files and create chunks
    documents = []
    file_count = 0
    
    for root, _, files in os.walk(folder_path):
        for file in files:
            if file.endswith(file_extensions):
                try:
                    with open(os.path.join(root, file), 'r', encoding='utf-8') as f:
                        content = f.read()
                        # Chunk content with configurable size and overlap
                        stride = chunk_size - chunk_overlap
                        chunks = [content[i:i+chunk_size] for i in range(0, len(content), stride)]
                        
                        for chunk in chunks:
                            if len(chunk) > 50:  # Skip tiny chunks
                                documents.append({
                                    "content": chunk,
                                    "source": os.path.join(root, file)
                                })
                    file_count += 1
                    if file_count % 100 == 0:
                        print(f"Processed {file_count} files...")
                except Exception as e:
                    print(f"Error processing file {file}: {e}")
    
    print(f"Creating embeddings for {len(documents)} chunks from {file_count} files...")
    
    # Create embeddings in batches to save memory
    batch_size = 32
    all_embeddings = []
    
    for i in range(0, len(documents), batch_size):
        batch = [doc["content"] for doc in documents[i:i+batch_size]]
        embeddings = embed_model.encode(batch)
        all_embeddings.append(embeddings)
    
    # Combine all embeddings
    embeddings = np.vstack(all_embeddings)
    
    # Create FAISS index
    dimension = embeddings.shape[1]
    index = faiss.IndexFlatL2(dimension)
    index.add(np.array(embeddings).astype('float32'))
    
    return index, documents, embed_model

# 2. Query knowledge base for relevant context
def get_context(query, index, documents, embed_model, top_k=3):
    query_embedding = embed_model.encode([query])
    scores, indices = index.search(np.array(query_embedding).astype('float32'), top_k)
    
    context = ""
    for idx in indices[0]:
        context += f"From {documents[idx]['source']}:\n{documents[idx]['content']}\n\n"
    
    return context
def main(args, index=None, documents=None, embed_model=None):
    # If verbose mode is on, print loading model message
    if args.verbose: print("Loading model...")
    
    # If timings mode is on, initialize timing variables
    if args.timings:
        started_timestamp = 0
        first_token_timestamp = 0

    # Load the model
    model = og.Model(f'{args.model}')
    if args.verbose: print("Model loaded")
    
    # Initialize the tokenizer with the model
    tokenizer = og.Tokenizer(model)
    tokenizer_stream = tokenizer.create_stream()
    if args.verbose: print("Tokenizer created")
    
    # Print a newline for readability if verbose mode is on
    if args.verbose: print()
    
    # Create a dictionary of search options from the command line arguments
    search_options = {name:getattr(args, name) for name in ['do_sample', 'max_length', 'min_length', 'top_p', 'top_k', 'temperature', 'repetition_penalty'] if name in args}
    
    # Set a default max length if one is not provided
    if 'max_length' not in search_options:
        search_options['max_length'] = 2048

    # Define a template for the chat input
    chat_template = '<|user|>\n{input} <|end|>\n<|assistant|>'

    # Main loop: ask for input and generate responses
    while True:
        # Get user input
        text = input("Input: ")
        
        if not text:
            print("Error, input cannot be empty")
            continue

        # If we have a knowledge base, retrieve relevant context
        context = ""
        if index is not None and documents is not None and embed_model is not None:
            if args.verbose: print("Retrieving context from knowledge base...")
            context = get_context(text, index, documents, embed_model, top_k=args.top_k)
            if args.verbose: print(f"Found {len(context.split())} words of context")

        # If timings mode is on, record the start time
        if args.timings: started_timestamp = time.time()

        # Format prompt with or without context
        if context:
            prompt = f'<|user|>\nUsing the following information, please answer my question:\n\n{context}\n\nQuestion: {text} <|end|>\n<|assistant|>'
        else:
            prompt = f'{chat_template.format(input=text)}'

        # Tokenize the input
        input_tokens = tokenizer.encode(prompt)

        # Set up the generator parameters
        params = og.GeneratorParams(model)
        # Remove the deprecated method call
        # params.try_graph_capture_with_max_batch_size(1)
        params.set_search_options(**search_options)
        
        # Create the generator with input tokens passed directly
        generator = og.Generator(model, params)
        if args.verbose: print("Generator created")
        
        # Print a message if verbose mode is on
        if args.verbose: print("Running generation loop ...")
        
        # If timings mode is on, initialize variables for the generation loop
        if args.timings:
            first = True
            new_tokens = []

        # Print the output prompt
        print()
        print("Output: ", end='', flush=True)

        try:
            generator.append_tokens(input_tokens)
            while not generator.is_done():
                generator.generate_next_token()

                new_token = generator.get_next_tokens()[0]
                print(tokenizer_stream.decode(new_token), end='', flush=True)
        except KeyboardInterrupt:
            print("  --control+c pressed, aborting generation--")

        print()
        del generator

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Run the chatbot script")
    parser.add_argument("--model", type=str, required=True, help="Path to the ONNX model file")
    parser.add_argument("--verbose", action="store_true", help="Enable verbose mode")
    parser.add_argument("--timings", action="store_true", help="Enable timings mode")
    parser.add_argument("--kb", type=str, help="Path to knowledge base folder")
    parser.add_argument("--cache", type=str, default="kb_cache.pkl", help="Path to save/load knowledge base cache")
    parser.add_argument("--top_k", type=int, default=3, help="Number of chunks to retrieve")
    args = parser.parse_args()
    
    # Load or create knowledge base
    index = None
    documents = None
    embed_model = None
    
    if args.kb:
        import pickle
        import os
        
        # Try to load from cache first
        if os.path.exists(args.cache):
            if args.verbose: print(f"Loading knowledge base from cache: {args.cache}")
            with open(args.cache, 'rb') as f:
                cached_data = pickle.load(f)
                index = cached_data['index']
                documents = cached_data['documents']
                embed_model = cached_data['embed_model']
        else:
            if args.verbose: print(f"Creating knowledge base from: {args.kb}")
            index, documents, embed_model = create_knowledge_base(args.kb)
            
            # Save to cache
            with open(args.cache, 'wb') as f:
                pickle.dump({
                    'index': index,
                    'documents': documents,
                    'embed_model': embed_model
                }, f)
    
    main(args, index, documents, embed_model)
```
