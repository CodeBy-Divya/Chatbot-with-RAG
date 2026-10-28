# Chatbot-with-RAG

A simple chatbot built with Streamlit that can answer questions about a PDF document. It uses a Groq-hosted language model together with Retrieval-Augmented Generation (RAG), so the answers come from the content of your PDF and not only from the model's general knowledge.

The project is built in three small steps (phases). Each phase is a separate file, so you can run them one by one and see how the chatbot grows from a basic chat window into a full RAG application.

---

## What is RAG?

A language model on its own only knows what it learned during training. It does not know what is inside your private documents.

RAG solves this in three steps:

1. **Split and store:** The PDF is cut into small pieces (chunks). Each chunk is converted into numbers (an embedding) and saved in a vector database.
2. **Retrieve:** When you ask a question, the system finds the few chunks that are most similar to your question.
3. **Generate:** Those chunks are given to the language model along with your question, and the model writes the answer using them.

---

## Project Structure

```
Chatbot-with-RAG/
├── phase_1.py      # Chat interface only (no AI model)
├── phase_2.py      # Chat interface + Groq language model
├── phase_3.py      # Chat interface + Groq model + PDF question answering (RAG)
├── Pipfile.txt     # List of required Python packages
├── Pipfile.lock    # Locked package versions
└── README.md
```

### Phase 1: Chat interface

File: `phase_1.py`

- Builds the chat screen using Streamlit with the title "Ask Chatbot!".
- Keeps all earlier messages in `st.session_state`, so the conversation stays on screen after each new message.
- Shows a text box at the bottom for your prompt.
- Replies with a fixed message ("I am your assistant"). No AI model is connected yet.

This phase proves the user interface and the message history work.

### Phase 2: Connect a language model

File: `phase_2.py`

- Keeps everything from Phase 1.
- Connects to the Groq API through LangChain (`ChatGroq`) using the `llama3-8b-8192` model.
- Uses a prompt template that asks the model to answer directly, with no small talk.
- Builds a chain: prompt, then model, then output parser, and shows the model's reply in the chat.
- Reads the API key from the `GROQ_API_KEY` environment variable.

Now the chatbot gives real answers, but only from the model's own knowledge.

### Phase 3: Add RAG over a PDF

File: `phase_3.py`

- Keeps everything from Phase 2.
- Loads a PDF named `reflexion.pdf` with `PyPDFLoader`.
- Splits it into chunks of 1000 characters with an overlap of 100 characters (`RecursiveCharacterTextSplitter`).
- Creates embeddings with the Hugging Face model `all-MiniLM-L12-v2` and stores them in a vector store.
- The vector store is created once and reused (`@st.cache_resource`), so the PDF is not processed again on every message.
- For each question, the retriever fetches the top 3 matching chunks (`k = 3`). A `RetrievalQA` chain passes them to the Groq model, which writes the final answer.
- Shows an error message in the app if something fails while loading the document or answering.

---

## Tech Stack

| Part | Tool |
|---|---|
| User interface | Streamlit |
| Language model | Groq API (`llama3-8b-8192`) |
| Orchestration | LangChain |
| PDF loading | PyPDF |
| Embeddings | Hugging Face Sentence Transformers (`all-MiniLM-L12-v2`) |
| Vector store | Created through LangChain's `VectorstoreIndexCreator` |
| Python version | 3.11 |

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/CodeBy-Divya/Chatbot-with-RAG.git
cd Chatbot-with-RAG
```

### 2. Create a virtual environment and install packages

```bash
python3.11 -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate

pip install streamlit langchain-groq langchain-community pypdf sentence-transformers
```

The `Pipfile.txt` in this repo lists these same five packages. If you use Pipenv, rename the file to `Pipfile` and run `pipenv install`.

Phase 3 also imports the main `langchain` package. If you see an import error for `langchain` or for the vector store, install them too:

```bash
pip install langchain chromadb
```

### 3. Get a Groq API key

Create a free account at [console.groq.com](https://console.groq.com), generate an API key, and set it as an environment variable.

```bash
export GROQ_API_KEY="your_api_key_here"        # macOS / Linux
setx GROQ_API_KEY "your_api_key_here"          # Windows (restart the terminal after)
```

Never write your API key directly in the code or commit it to GitHub.

### 4. Add your PDF (Phase 3 only)

Phase 3 looks for a file called `reflexion.pdf` in the project folder. This file is not included in the repository, so place your own PDF in the folder and either name it `reflexion.pdf` or change this line in `phase_3.py`:

```python
pdf_name = "./reflexion.pdf"
```

### 5. Run the app

Run any one phase:

```bash
streamlit run phase_1.py
streamlit run phase_2.py
streamlit run phase_3.py
```

Streamlit opens the app in your browser, usually at `http://localhost:8501`.

---

## Example Usage

With `phase_3.py` running and a PDF in place:

1. Type a question about the PDF, for example: "What is the main idea of this paper?"
2. The app finds the three most relevant parts of the PDF.
3. The Groq model reads those parts and writes the answer in the chat.

---

## Known Limitations

- The PDF path is fixed in the code. There is no file upload button yet.
- Phase 3 works with one PDF at a time.
- The chat does not use earlier messages when answering. Each question is handled on its own.
- Source chunks are retrieved but not shown to the user.
- Groq updates its list of available models from time to time. If `llama3-8b-8192` is no longer available, pick a current model from the Groq documentation and change the `model` line in the code.
- Some imports in `phase_3.py` (for example `langchain.embeddings` and `langchain.document_loaders`) come from older LangChain versions. In newer versions they have moved to `langchain_community`.

---

## Future Improvements

- Add a PDF upload option in the app.
- Support several documents at once.
- Add chat memory so follow-up questions work.
- Show the source pages used for each answer.
- Move to the latest LangChain imports and a currently supported Groq model.

---

