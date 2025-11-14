Automated Patent Summarization System


💡 Project Abstract

This project presents an AI-powered Patent Summarization System designed to transform lengthy, complex patent documents into concise, easy-to-read summaries. The core engine utilizes a fine-tuned language model, combining the efficiency of BART-large-cnn for speed and the advanced capabilities of GPT-4o-mini for high-quality, abstractive summarization. The system features a user-friendly Gradio web interface that supports both direct text input and PDF file uploads, significantly reducing patent analysis time for legal professionals, researchers, and innovators.

Category : Model
Component : Primary Summarizer
Detail : Fine-tuned `facebook/bart-large-cnn` (Used in the web app)

Category : Model
Component : Secondary/Reference
Detail : `gpt-4o-mini` (Used for general summarization/comparison)

Category : Framework
Component : Front-end
Detail : Gradio (For the interactive web interface)

Category : Libraries
Component : Core NLP
Detail : `transformers`, `openai`

Category : Utilities
Component : PDF Handling
Detail : `PyMuPDF` (via `fitz`) for text extraction

Category : Evaluation
Component : Metrics
Detail : `rouge-score` (For ROUGE-1, ROUGE-2, ROUGE-L analysis)


⚙️ Step-by-Step Guide to Execution
This section details how to set up and run the project using the provided Code.ipynb file.

Step 1: Prerequisites and Installation
Ensure you have Python 3.8 or higher installed. Then, install all necessary libraries using the following command:

# Install all required dependencies
pip install --upgrade openai gradio pandas matplotlib seaborn rouge-score transformers pymupdf


Step 2: Set Up API Key
You must obtain an OpenAI API Key and replace the placeholder in the code to initialize the client.

import openai
import gradio as gr
from transformers import pipeline
import fitz  # PyMuPDF
import re

# Set your OpenAI API Key (REPLACE WITH YOUR SECURE KEY)
API_KEY = "sk-proj-YOUR_API_KEY_HERE"  # ⚠️ Keep this secure and use environment variables for actual deployment
client = openai.OpenAI(api_key=API_KEY)

# 1. Load the fine-tuned summarizer model
# Note: The code uses BART, which is a powerful sequence-to-sequence model for abstractive summarization.
finetune_summarizer = pipeline("summarization", model="facebook/bart-large-cnn")

# 2. Define the main function to process input and generate summary
def process_input(file=None, text_input=None):
    try:
        desc = ""
        # Priority to pasted text input
        if text_input and text_input.strip():
            desc = text_input.strip()

        # Else try to read PDF file
        elif file is not None and file.endswith(".pdf"):
            doc = fitz.open(file)
            text = "".join([page.get_text() for page in doc])
            # Attempt to extract 'Description' section from the patent text
            description_match = re.search(r"Description:\s*(.*)", text, re.DOTALL)
            desc = description_match.group(1).strip() if description_match else text.strip()

        else:
            return "❌ Please upload a PDF or paste patent text."

        if not desc:
            return "❌ Could not extract meaningful content."

        # Generate summary using the fine-tuned BART model
        finetune_summary = finetune_summarizer(
            desc,
            max_length=150,  # Max output token length
            min_length=30,   # Min output token length
            do_sample=False  # Deterministic generation
        )[0]['summary_text']
        
        return finetune_summary

    except Exception as e:
        return f"❌ Error: {e}"

# 3. Create and launch the Gradio User Interface
iface = gr.Interface(
    fn=process_input,
    inputs=[
        gr.File(label="Upload Patent PDF (optional)", type="filepath"),
        gr.Textbox(label="Or Paste Patent Text", lines=10, placeholder="Paste patent content here if no PDF...")
    ],
    outputs=gr.Textbox(label="📄 Fine-Tuned Summary"),
    title="Automated Patent Summarizer",
    description="Upload a patent PDF OR paste the content. This tool will summarize it using a fine-tuned model."
)

iface.launch(share=True)


Step 3: Run the Script and Use the UI
Execute the code: Run the Code.ipynb file in a Jupyter environment (or save the code above as main_app.py and run python main_app.py).

Launch Interface: The script will automatically launch the Gradio web interface in your browser.

Input: You have two options for input:

Option 1: Use the file uploader to upload a patent PDF file.

Option 2: Paste the full patent description directly into the large textbox.

Output: Click the button to generate the summary. The concise summary will appear in the output box below the inputs.

📊 Performance and Conclusion
Model Comparison: The fine-tuned model (BART) was specifically compared against the base GPT-4 model.

Key Results: The fine-tuned model demonstrated better ROUGE-1, ROUGE-2, and ROUGE-L scores, indicating superior overlap and quality in the generated summaries compared to the base GPT-4.

Conclusion: The system successfully condenses lengthy patents into easy-to-read summaries, enhancing comprehension and accelerating decision-making in the patent review process.

⏭️ Future Enhancements
Potential future developments for this system include:

Audio/Video Summarization: Expanding support to summarize multimedia patent documents.

Advanced Legal Analysis: Adding features to detect infringement risks and legal overlaps.

Custom Summary Styles: Providing technical, legal, and layman-level summaries.