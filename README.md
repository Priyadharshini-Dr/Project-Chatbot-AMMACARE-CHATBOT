# Project-Chatbot-AMMACARE-CHATBOT

Ammacare chatbot is an AI-driven virtual assistant that provides guidance for pregnant women, new mothers, and postpartum care.
README.md file for the Ammacare Bot project, including sections for project summary, setup instructions, key components, usage of the Gradio UI, evaluation process, and LangSmith tracing.

Summarize the project

Subtask:

Write a brief overview of the Ammacare Bot and its purpose.
Reasoning: Write the project summary and purpose for the README.md file.


readme_content = """# Ammacare Bot

## Project Summary

The Ammacare Bot is an AI-powered assistant designed to provide information and support related to maternal and newborn care. Its primary goal is to offer accessible and reliable guidance to expectant and new parents.

Key capabilities include:
- Answering questions based on a knowledge base.
- Generating pregnancy checklists based on the week of gestation.
- Analyzing images related to maternal or newborn health and answering questions about them.
- Handling voice input for queries.
"""

with open("README.md", "w") as f:
    f.write(readme_content)
print("README.md created with project summary.")

Explain the setup

Subtask:
Detail the necessary installations and dependencies for the Ammacare Bot project.
Reasoning: Append the setup instructions to the README.md file, detailing installations, API keys, Google Drive requirements, and the sources.txt file format.

setup_content = """
## Setup

To run the Ammacare Bot, you need to install the necessary Python libraries and configure API keys.

### Installation

Install the required packages using pip:

```bash
!pip install openai-whisper
!pip install --upgrade git+https://github.com/pytube/pytube
!pip install -q langchain langchain-community openai pypdf yt_dlp youtube-transcript-api chromadb gradio huggingface_hub transformers accelerate whisper gtts Pillow langsmith
```

### API Keys

The project requires API keys for OpenAI and LangChain. Set these as environment variables:

```python
import os
from google.colab import userdata # If running in Google Colab

os.environ["OPENAI_API_KEY"]    = userdata.get("OPENAI_API_KEY") # Or replace with your key
os.environ["LANGCHAIN_API_KEY"] = userdata.get("LANGCHAIN_API_KEY") # Or replace with your key
os.environ["LANGCHAIN_PROJECT"] = "AmmacareBot" # Replace with your project name
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["HF_TOKEN"]          = userdata.get("HF_TOKEN") # For Hugging Face models
```

If not running in Google Colab, you can load these from a `.env` file using a library like `python-dotenv`.

### Google Drive Access

The notebook is configured to use Google Drive for persistent storage of the Chroma vector database. Ensure your environment has access to Google Drive and mount it at `/content/drive` or update the `persist_dir` variable accordingly.

### Data Ingestion (`sources.txt`)

The bot ingests knowledge from external sources listed in a file named `sources.txt`. This file should be located at `/content/sources.txt` or the path should be updated in the ingestion code. The format for each line in `sources.txt` should be `Title | URL`, where `Title` is the name you want to assign to the source and `URL` is the link to a PDF document or a YouTube video. Lines starting with `#` are treated as comments.

Example `sources.txt`:

```
# Example sources file
Official WHO Guidelines | https://www.who.int/docs/default-source/coronaviruse/breastfeeding-and-covid-19.pdf
Understanding Postpartum Depression | https://www.youtube.com/watch?v=some_video_id
```
"""

with open("README.md", "a") as f:
    f.write(setup_content)
print("Setup section added to README.md.")

Describe the key components

Subtask:

Explain the different steps in the notebook, such as data loading, model initialization, data ingestion, and the different tools (chat, image QA, voice).
Reasoning: Define the markdown string for the "Key Components" section and append it to the README.md file.

key_components_content = """
## Key Components

The Ammacare Bot is built using several key components, each handling a specific part of the process, from data ingestion to user interaction.

### Data Loading and Setup (Steps 2, 5)

The project initializes by mounting Google Drive for persistent storage (`chroma_store`) and loading necessary API keys from environment variables. It then reads the `sources.txt` file to identify URLs for PDF documents and YouTube videos that will be used to build the knowledge base.

### Model Initialization (Step 3)

LangChain is used to set up the core components for the Retrieval Augmented Generation (RAG) system:
- **Embeddings:** `OpenAIEmbeddings` are used to convert text data into numerical vectors.
- **Vector Store:** `Chroma` is used as the vector database to store the embedded documents and enable efficient similarity search. The vector store is configured to persist data to the mounted Google Drive.
- **Text Splitter:** `RecursiveCharacterTextSplitter` breaks down large documents into smaller, manageable chunks.
- **Language Model (LLM):** `ChatOpenAI` (specifically `gpt-4o-mini`) is used as the conversational model to generate answers based on retrieved information.
- **Retriever:** Configured from the vector store to fetch relevant document chunks based on user queries.
- **RetrievalQA Chain:** Combines the LLM and retriever to answer questions using the retrieved context.

### Data Ingestion (Step 6)

This step processes the sources identified in `sources.txt`:
- **PDF Download:** PDFs are downloaded to a local directory (`/content/pdfs`).
- **YouTube Download:** YouTube videos are downloaded as MP4 files to `/content/youtube_videos` using `yt-dlp`.
- **Transcription:** The downloaded YouTube videos are transcribed into text files (`/content/ingested_texts`) using the Whisper model (`small`).
- **Ingestion into Vector Store:** The text content from PDFs (loaded via `PyPDFLoader`) and transcriptions from YouTube are processed by the `RecursiveCharacterTextSplitter` and then added as `Document` objects to the `Chroma` vector database. This process builds the searchable knowledge base for the bot.

### Core Chat Functionality (`ask_bot` function, part of Step 11)

The `ask_bot` function handles basic text-based questions. It uses the `RetrievalQA` chain to find relevant information in the vector store based on the user's query and generates an answer. It also returns the source documents used.

### Image QA Tool (`analyze_image_with_question` function, Step 9)

This tool allows users to upload an image and ask a question about it.
- It uses a Hugging Face `image-to-text` pipeline (`Salesforce/blip-image-captioning-base`) to generate a caption for the uploaded image.
- The caption is combined with the user's question.
- This combined query is then passed to the `RetrievalQA` chain (`qa_chain`) to generate an answer based on the image context and the knowledge base.

### Voice Tool (`transcribe_audio` and `tts_audio` functions, Step 10)

This tool provides voice input and output capabilities:
- **Transcription:** The `transcribe_audio` function uses the Whisper model to convert spoken audio input (from a microphone or uploaded file) into text.
- **Processing:** The transcribed text is then passed to the `ask_bot` function to get an answer from the RAG system.
- **Text-to-Speech (TTS):** The `tts_audio` function uses `gTTS` to convert the bot's text answer back into an audio file, which is then played back to the user in the UI.

### Pregnancy Checklist Tool (`pregnancy_checklist` function, Step 7)

This simple tool generates a basic checklist of recommended actions or considerations based on the provided week of pregnancy. This provides structured advice for different stages of gestation.

### Gradio UI (Step 11)

A web-based user interface is created using Gradio, providing interactive components for:
- Text chat with the bot.
- Uploading images and asking questions about them.
- Recording or uploading audio for voice input.
- Playing back audio responses from the bot.
- Selecting a pregnancy week to generate a checklist.

### Evaluation (Step 8)

A simple evaluation mechanism is implemented using `evaluate_answer` function. It uses a separate LLM instance (`gpt-4o-mini`) acting as a judge to assess if the bot's answer is 'Correct', 'Partial', or 'Hallucinated' based on the original question and the retrieved source documents. This helps in qualitatively assessing the RAG system's performance.

### LangSmith Tracing (Steps 7, 9, 10)

LangSmith tracing is enabled throughout the application. Key functions like `ask_bot`, `analyze_image_with_question`, and `transcribe_audio` are decorated with `@traceable` to send detailed execution traces to the LangSmith platform. This allows for monitoring, debugging, and analyzing the performance and behavior of the LangChain components and custom tools.
"""

with open("README.md", "a") as f:
    f.write(key_components_content)
print("Key Components section added to README.md.")

Explain how to use the gradio ui

Subtask:

Provide instructions on how to interact with the chatbot, image QA, and voice input features using the Gradio UI.
Reasoning: Create a markdown string explaining the Gradio UI usage and append it to the README.md file.

gradio_usage_content = """
## Gradio UI Usage

The Ammacare Bot provides a user-friendly web interface built with Gradio.

1.  **Launching the UI:**
    *   Execute the final code cell in the notebook (`demo.launch()`).
    *   A public URL will be provided (e.g., `https://<your-domain>.gradio.live`). Open this URL in your web browser.

2.  **Main Chat Interface:**
    *   Type your question or message into the textbox labeled "Ask me anything...".
    *   Click the "Ask" button or press Enter to send your message.
    *   The bot's response will appear in the chat history area above the input box.

3.  **Clearing Chat History:**
    *   Click the "Clear" button to remove all previous messages from the chat history.

4.  **Image QA:**
    *   In the "Image QA" section, click the "Upload image" box to select and upload an image file.
    *   Type your question about the image into the textbox labeled "Ask about the image".
    *   Click the "Analyze Image + Question" button.
    *   The bot will analyze the image, generate a caption, and then answer your question based on the image content and its knowledge base. The result, including the caption, question, answer, sources, and evaluation result, will be added to the main chat history.

5.  **Voice Input:**
    *   In the "Voice Input" section, use the "Record or Upload Audio" component to either record audio using your microphone or upload an audio file.
    *   Click the "Ask by voice" button.
    *   The audio will be transcribed into text, and the bot will answer your question based on the transcribed text. The transcription and the bot's response will appear in the main chat history.
    *   The bot's answer will also be converted to speech and automatically played back through the "Audio answer" player.

6.  **Pregnancy Checklist:**
    *   In the "Pregnancy Checklist" section, move the slider to select the desired week of pregnancy (from 1 to 40).
    *   Click the "Generate" button.
    *   A relevant checklist for that week will be displayed in the textbox below the button.

Interactions with the bot, image analysis, and voice transcription/response will be logged to LangSmith if tracing is enabled, allowing you to monitor the system's performance and debug issues.
"""

with open("README.md", "a") as f:
    f.write(gradio_usage_content)
print("Gradio UI Usage section added to README.md.")

Explain the evaluation process

Subtask:

Describe how the model's answers are evaluated based on the provided context.
Reasoning: Define the markdown content for the evaluation section and append it to the README.md file.

evaluation_content = """
## Evaluation Process

To assess the quality of the bot's answers, a simple evaluation process is implemented using the `evaluate_answer` function (Step 8).

This function takes the following inputs:
- The original user `question`.
- The bot's generated `answer`.
- A list of `docs` (source documents) retrieved from the vector database that were used to generate the answer.

A separate instance of the `ChatOpenAI` model (`gpt-4o-mini`), acting as a judge, is given these inputs along with a specific prompt (`judge_prompt`). The judge model then provides a qualitative assessment based *solely* on whether the bot's `answer` is supported by the `Context` provided by the retrieved `docs`.

The possible evaluation outcomes returned by the judge are:
- **Correct:** The answer is fully supported by the provided context.
- **Partial:** The answer is partially supported by the provided context, or is incomplete but not incorrect.
- **Hallucinated:** The answer is not supported by the provided context.

This evaluation result is displayed in the Gradio UI for Image QA responses and can be seen in the LangSmith traces for all `ask_bot` calls, providing insights into how well the RAG system is leveraging the retrieved information.
"""

with open("README.md", "a") as f:
    f.write(evaluation_content)
print("Evaluation Process section added to README.md.")

Mention langsmith tracing

Subtask:

Briefly explain the role of LangSmith in tracing the tool calls within the Ammacare Bot project.
Reasoning: Define the markdown content for the LangSmith Tracing section and append it to the README.md file, then print a confirmation message.

langsmith_tracing_content = """
## LangSmith Tracing

LangSmith is integrated into the Ammacare Bot project to provide detailed tracing and monitoring of the application's execution, particularly the flow within LangChain components and custom tools.

Key benefits of using LangSmith tracing:
- **Monitoring:** Visualize the sequence of calls, inputs, outputs, and duration for each step in a process (e.g., a user query leading to a RAG chain execution).
- **Debugging:** Identify where errors occur or why the bot might be returning unexpected results by inspecting the inputs and outputs at each stage of a trace.
- **Analysis:** Understand the performance of different components, analyze latency, and evaluate the cost of runs.

Several key functions and chains in the Ammacare Bot are decorated with the `@traceable` decorator (from `langsmith.traceable`). This automatically sends execution data for these functions to your LangSmith project (`AmmacareBot`, as configured in Step 2), allowing you to view and analyze the runs on the LangSmith platform. Look for traces named `ammacare_qa`, `ammacare_image_qa`, and `ammacare_transcribe_audio`.
"""

with open("README.md", "a") as f:
    f.write(langsmith_tracing_content)
print("LangSmith Tracing section added to README.md.")



**Summary:**

Data Analysis Key Findings

The Ammacare Bot is an AI assistant for maternal and newborn care with capabilities including Q&A, pregnancy checklists, image analysis, and voice input.
Setting up the project requires installing Python libraries via pip, setting OpenAI and LangChain API keys as environment variables, ensuring Google Drive access for persistent storage, and configuring a sources.txt file for data ingestion (supporting PDF and YouTube URLs).
Key components include data loading, model initialization (using OpenAIEmbeddings, Chroma, RecursiveCharacterTextSplitter, ChatOpenAI gpt-4o-mini, and RetrievalQA chain), data ingestion from PDFs and YouTube, a core chat function, an image QA tool (using Hugging Face image-to-text and RAG), a voice tool (using Whisper for transcription and gTTS for text-to-speech), a pregnancy checklist tool, a Gradio UI for interaction, an evaluation process (using gpt-4o-mini as a judge), and LangSmith tracing.
The Gradio UI provides interactive components for text chat, image upload and analysis, audio recording/upload for voice input, and a slider for generating pregnancy checklists.
The evaluation process uses a separate LLM instance to judge if the bot's answer is 'Correct', 'Partial', or 'Hallucinated' based on the provided source documents.
LangSmith tracing is integrated using the @traceable decorator to monitor, debug, and analyze the execution flow of key functions and LangChain components, with traces visible on the LangSmith platform.
Insights or Next Steps

The comprehensive README covers essential aspects of the project, providing a solid foundation for users or contributors to understand, set up, and use the Ammacare Bot.
Future documentation could include a troubleshooting section based on common setup or runtime issues, and more detailed explanations of the prompt templates used for the main RAG chain and the evaluation judge.










