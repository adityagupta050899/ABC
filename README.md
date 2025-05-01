# 01-ABC

# 🚨 Suspicious Activity Report (SAR) Generator Using GenAI

This project automates the generation of Suspicious Activity Reports (SARs) for the Agricultural Bank of China, New York Branch. It combines a PostgreSQL database, document-based Retrieval-Augmented Generation (RAG), and a Large Language Model (LLM) to produce structured SAR narratives.

---

## 🖼️ 1. Visual Overview

- Visual representations of the SAR generation pipeline, metadata structure, and relational database schemas.
![image](https://github.com/user-attachments/assets/92e2f83d-aa6c-4914-ae3f-e882e946a628)

---

## ⚙️ 2. Configuration Steps

- This section contains all related configuration steps from database creation to querying the data, creating the prompt and any function required for greater automation of the SAR Creation process.

### 📌 2.1 Database Creation

- Before create the datebase, you need to uplod 4 csv files
- PostgreSQL schema with 4 tables: `Customer`, `Account`, `Transaction`, `Alert`
![image](https://github.com/user-attachments/assets/0f0130bd-5da1-4e63-88a5-61b771e5c9cf)
- Use `psycopg2` to set up the connection to PostgreSQL
- The four tables to be created will be connected to each other as per the ERD Diagram below
![image](https://github.com/user-attachments/assets/1fe69a20-120e-4f93-84ee-50cb69fa60c0)
- Creating the four tables and importing four local CSV files into Database

### 🔍 2.2 RAG Setup: Alert Narratives and Guidelines (Section 4)

- Load 3 fake SAR narratives and a guidelines document from `.docx` files(need to upload):
  - `A-1 Alert Narrative.docx`
  - `A-2 Fake Alert Narrative Fixed.docx`
  - `A-5 Fake Alert Narrative.docx`
  - `Suspicious Activity Guidelines.docx`
- Split each document into text chunks (paragraphs) using `docx2txt`, keeping only those with more than 50 characters.
- Generate embeddings with `HuggingFaceEmbeddings`(model name = "all-MiniLM-L6-v2")
- Store them in a `FAISS` vector store using LangChain.

### 🧠 2.3 Prompt Generation

- A structured prompt template is defined for use by the LLM to generate each SAR narrative.
- Placeholders in {braces} are dynamically filled using data queried from the database.
- Two versions of the prompt are used: one for Individual clients and another for Business clients, reflecting different available data (e.g., DOB, SSN, address).
- Construct dynamic prompts using information retrieved from both:
  - SQL queries (for entity information)
  - FAISS (for similar SAR examples)
- The initial draft of the prompt was created with help from ChatGPT, and iteratively refined based on project needs.

### 🔗 2.4 Query Functions: Building Prompt Variables

- To generate customized SAR narratives, two query functions are used—one for individual clients and one for business clients.
- Each function:
  - Queries the Alert, Transaction, Customer, and Account tables to extract relevant data.
  - Builds a transaction summary (including direction and date of wire transfers).
  - Assembles KYC details and summary statistics (transaction count, amount, etc.).
  - Retrieves similar case examples from the FAISS index based on the transaction summary.
  - Returns a dictionary of variables to fill the prompt's placeholders.
- A separate utility extracts the Customer ID from .docx files using regular expressions.

### 🤖 2.5 LLM Set Up

- Set up AWS credentials (Access Key ID, Secret Access Key, and Region) via aws configure
- Connect to AWS Bedrock using boto3 with the bedrock-runtime client in region us-east-1.
- Construct the request payload, including:
  - prompt: filled template with placeholders resolved
  - temperature: 0.6
  - top_p: 0.9
  - max_gen_len: 2048
-  Supported models:
  - LLaMA 3 70B → us.meta.llama3-3-70b-instruct-v1:0
  - Deepseek R1 70B → us.deepseek.r1-v1:0

---

## 🧾 3. SAR Generation

- In order to analyze the capability of LLMs in generating SARs, the code below will create 3 SAR Narratives based on the 3 SAR Alert Narratives provided by the client.
- For each of the available SAR Alert Narratives, output will be provided from two models, Llama 3.3 and DeepSeek R-1, on 3 three different temperature (0.3, 0.6, 0.9) settings.
- This will allow us to then guage which temperature is ideal per model and by using the newly created grading rubric we will be able to asses with model performs best.

### 📝 SAR Narrative 1

### 📝 SAR Narrative 2

### 📝 SAR Narrative 3

---

## 🧪 Alternatives Tried but Not Adopted
- We explored several other approaches before settling on the current AWS Bedrock-based solution:
- **Local LLM via Ollama**: Initially attempted to run LLaMA models locally using Ollama. However:
  - High memory requirement (70B model needs ≥20GB VRAM)
  - Smaller models performed poorly in generating quality SAR narratives
- **AWS EC2 + Bedrock (Direct Invocation without aws configure)**: Running inference via an EC2 instance had some advantages (no manual credential config needed), but was eventually rejected due to:
  - Long startup times each time the instance was rebooted
  - Higher cost for keeping instances alive during development
  - Operational complexity for a student project
- We ultimately chose to call Bedrock APIs from Google Colab, which balanced simplicity, speed, and flexibility.

---
