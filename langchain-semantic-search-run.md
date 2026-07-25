# Build a semantic search engine with LangChain

## Create raw documents dataset

Example syntax required:

```
example_documents = [
    Document(
        page_content="Dogs are great companions, known for their loyalty and friendliness.",
        metadata={"source": "mammal-pets-doc"},
    ),
    Document(
        page_content="Cats are independent pets that often enjoy their own space.",
        metadata={"source": "mammal-pets-doc"},
    ),
]
```

This search dataset will be based on Broadcom's Symantec DLP documentation, which is 2192 pages and is available online.

Convert the PDF to a json data model.

```
from langchain_community.document_loaders import PyPDFLoader

file_path = "symantec-data-loss-prevention-help-center-25-1.pdf"
loader = PyPDFLoader(file_path)

docs = loader.load()
```
Note: PyPDFLoader has recently been deprecated, it can be replaced with a PDF reader which extracts the text and constructs the JSON data as previously detailed.

Run documents though text splitter.

```
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000, chunk_overlap=200, add_start_index=True
)
all_splits = text_splitter.split_documents(docs)
```
This generates 7328 splits.

## Create sentence transformer and load into memory vector store

Download and load sentence transformer.

```
from langchain_huggingface import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-mpnet-base-v2",
    encode_kwargs={"normalize_embeddings": True},
)
```

Load data into memory based vector store

```
from langchain_core.vectorstores import InMemoryVectorStore

vector_store = InMemoryVectorStore(embeddings)

ids = vector_store.add_documents(documents=all_splits)
```

Create a function which allows easy search

```
def search(txt):
  results = vector_store.similarity_search_with_score(txt)
  print("Length of results:",len(results))
  for result in results:
    doc, score = result
    print("===================")
    print(f"Score: {score}\n")
    print(doc.page_content)
    print("PDF Array Element=",doc.metadata.get("page"))
```

You can then search

```
search("Block file web uploads using agent?")
```

Example of complete implementation:

[memoryVectorSearch](https://github.com/pauldeadman/pd-ml-langchain/blob/main/langchain-semantic-search-run-pub.ipynb)
