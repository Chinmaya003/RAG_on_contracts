
# Data Extraction Pipeline

This repository provides a pipeline for extracting relevant fields from bid documents, including PDFs and HTML files, using natural language processing techniques. It uses pre-trained models from Hugging Face to answer specific questions from the bid documents.

## Installation

Ensure you have the required dependencies by running the following command:

```bash
!pip install html2text pymupdf4llm torch transformers pandas chromadb
```

## How It Works

This project processes bid documents from `Bid1` and `Bid2`, which can include both PDF and HTML files. The pipeline extracts important details, such as bid number, title, due date, product specifications, and contact information, from these documents.

### Steps:

1. **HTML to Markdown Conversion:**
   HTML files are converted into markdown format for easier processing using the `html2text` library.

2. **Bid Document Processing:**
   PDF documents are converted into markdown format using the `pymupdf4llm` library. This allows easier extraction of tables and text content from the bid files.

3. **Question-Answering Pipeline:**
   The model used for question-answering is `distilbert-base-uncased-distilled-squad`, which is fine-tuned for answering specific questions about documents. The pipeline is configured to run on a GPU if available.

4. **Field Extraction:**
   The pipeline extracts key fields from the documents based on predefined questions for each field. Some of the fields include:
   - Bid Number
   - Title
   - Due Date
   - Bid Submission Type
   - Product Specifications
   - Contact Information

5. **Output:**
   The extracted information is saved to JSON files for further use or analysis.

### Example Fields

Here are the questions asked during the field extraction process:
- "What is the Bid Number or identifier mentioned in the contract or proposal?"
- "What is the title or name of the contract or proposal?"
- "What is the due date for the submission of bids or responses?"
- "Does the contract mention any pre-bid meeting or event before submission?"
- "What are the product specifications, features, or requirements mentioned in the contract?"

## Usage

1. **Prepare Documents:**
   Place your bid documents (PDFs and HTML files) in the `Bid1` and `Bid2` directories. Make sure the file names match the ones used in the code or adjust them as needed.

2. **Run the Pipeline:**
   Execute the following code in a Jupyter notebook or script:

   ```python
   # Install dependencies (if not installed already)
   !pip install html2text pymupdf4llm torch transformers pandas chromadb

   # Import necessary libraries
   import chromadb
   import html2text
   import pymupdf4llm
   import torch
   import re
   import pandas as pd
   from io import StringIO
   from transformers import pipeline

   # Define the HTML to markdown conversion function
   def html_to_markdown(html_path):
       with open(html_path, 'r', encoding='utf-8') as html_file:
           html_content = html_file.read()
       h = html2text.HTML2Text()
       h.ignore_links = False
       return h.handle(html_content)

   # Define the QA pipeline and fields
   device = 0 if torch.cuda.is_available() else -1
   qa_pipeline = pipeline("question-answering", model="distilbert-base-uncased-distilled-squad", device=device)
   fields = {
       "Bid Number": "What is the Bid Number or identifier mentioned in the contract or proposal?",
       "Title": "What is the title or name of the contract or proposal?",
       "Due Date": "What is the due date for the submission of bids or responses?",
       "Bid Submission Type": "How is the bid or proposal submission expected (e.g., online, by mail)?",
       # Add other fields here...
   }

   # Document paths for Bid1 and Bid2
   doc1 = pymupdf4llm.to_markdown('Bid1/Addendum 1 RFP JA-207652 Student and Staff Computing Devices.pdf')
   doc2 = pymupdf4llm.to_markdown('Bid1/Addendum 2 RFP JA-207652 Student and Staff Computing Devices.pdf')
   doc3 = pymupdf4llm.to_markdown('Bid1/JA-207652 Student and Staff Computing Devices FINAL.pdf')
   html_path = 'Bid1/Student and Staff Computing Devices __SOURCING #168884__ - Bid Information - {3} _ BidNet Direct.html'
   doc4 = html_to_markdown(html_path)

   # Combine all Bid1 documents
   data1 = doc1 + doc2 + doc3 + doc4
   output1 = mixed_content_pipeline(data1, fields)

   # Save the output for Bid1
   with open("output/DallasOutput.json", "w") as f:
       json.dump(output1, f, indent=4)
   print(f"Pipeline results saved to output/DallasOutput.json")
   ```

3. **Inspect Output:**
   The extracted fields will be saved in a JSON file (`output/DallasOutput.json` or `output/DellOutput.json`). You can load and analyze this file for the extracted data.

   Example output structure:
   ```json
   {
     "extracted_fields": {
       "Bid Number": ["JA-207652"],
       "Title": ["Student and Staff Computing Devices"],
       "Due Date": ["30/11/2024"],
       "Bid Submission Type": ["Online"],
       "Product Specification": ["Dell Laptop", "Extended Warranty"]
     }
   }
   ```

## Folder Structure

```
project/
├── Bid1/                        # Bid1 documents (PDFs, HTML)
├── Bid2/                        # Bid2 documents (PDFs, HTML)
├── output/                      # Folder for storing JSON output
├── notebook.ipynb               # Jupyter notebook or script with pipeline code
└── README.md                    # This file
```

## Notes

- **Bid Documents:** Ensure that all bid documents are formatted correctly, and that the paths are accurate in the code.
- **Field Definitions:** You can modify the `fields` dictionary to extract other relevant information as needed.
- **Model:** The QA model used in this pipeline is a general-purpose model (`distilbert-base-uncased-distilled-squad`). For more specific needs, you can fine-tune a model on your own dataset.
```

