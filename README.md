# Quran-PDF
To build a chatbot that answers queries based on the content of a Quran PDF, we need to follow these steps:

    Extract Text from PDF: First, we'll need to extract the text from the Quran PDF. We can use libraries like PyMuPDF or PyPDF2 to extract the text.

    Preprocessing: After extracting the text, we will preprocess it to make it ready for querying. This could involve splitting the text into chapters, verses, and further organizing it.

    Create a Simple Querying System: Once the text is extracted, we can use simple keyword matching or a more advanced approach like using BERT or GPT for more natural language queries.

    Integrating with a Chatbot: We can use a simple Python Flask app to create an interface for the chatbot that allows users to input their questions, which the system will respond to based on the Quranic text.

Here’s how you can set this up step by step:
1. Extract Text from Quran PDF

We'll start by extracting text from the PDF. You can use libraries like PyMuPDF or PyPDF2. Here is an example using PyMuPDF to extract text from each page.

pip install pymupdf

import fitz  # PyMuPDF

def extract_text_from_pdf(pdf_path):
    doc = fitz.open(pdf_path)
    text = ""
    
    for page_num in range(len(doc)):
        page = doc.load_page(page_num)
        text += page.get_text("text")  # Extracting text
    return text

# Example usage:
pdf_path = "path_to_quran.pdf"
quran_text = extract_text_from_pdf(pdf_path)
print(quran_text[:1000])  # Print the first 1000 characters to check the extraction

2. Preprocess the Extracted Text

Next, we should preprocess the extracted text. Typically, a Quranic text will be structured into chapters (Surahs) and verses (Ayahs). We can extract these by looking for patterns that identify Surah and Ayah numbers in the extracted text.

Here's an example of a very basic preprocessing step:

import re

def preprocess_quran_text(text):
    # Let's assume each Surah starts with "Surah" followed by the Surah number, e.g., "Surah 1"
    surah_pattern = r"Surah (\d+)"
    ayah_pattern = r"(\d+):\s*(.*?)\n"  # Example pattern for Ayahs
    
    # Split the text into Surahs and Ayahs
    surahs = {}
    current_surah = None
    
    for line in text.splitlines():
        # Check for Surah start
        surah_match = re.match(surah_pattern, line)
        if surah_match:
            current_surah = int(surah_match.group(1))
            surahs[current_surah] = []
        
        # Extract Ayahs under each Surah
        ayah_match = re.match(ayah_pattern, line)
        if ayah_match and current_surah:
            ayah_number = ayah_match.group(1)
            ayah_text = ayah_match.group(2)
            surahs[current_surah].append({'ayah_number': ayah_number, 'text': ayah_text})
    
    return surahs

# Example usage:
quran_surahs = preprocess_quran_text(quran_text)
print(quran_surahs[1][:3])  # Print first 3 verses of Surah 1

3. Building a Querying System for the Chatbot

Now that we have the Quran's text organized into Surahs and Ayahs, we can create a basic query system. We'll use simple keyword-based matching or a more advanced model like GPT-3 (via the OpenAI API) to handle queries.
Simple Keyword Search Example

def search_quran(query, quran_surahs):
    query = query.lower()
    results = []
    
    for surah_num, ayahs in quran_surahs.items():
        for ayah in ayahs:
            if query in ayah['text'].lower():
                results.append({
                    'surah': surah_num,
                    'ayah': ayah['ayah_number'],
                    'text': ayah['text']
                })
    return results

# Example usage:
query = "mercy"
results = search_quran(query, quran_surahs)

for result in results:
    print(f"Surah {result['surah']} - Ayah {result['ayah']}: {result['text']}")

Using OpenAI API for Advanced Querying

For more natural language processing, you can integrate GPT-3 or GPT-4 via OpenAI's API. Below is an example of how you can create a more intelligent query-response system with OpenAI's API:

pip install openai

import openai

openai.api_key = 'your_openai_api_key_here'

def ask_openai(question, context):
    prompt = f"Context: {context}\nQuestion: {question}\nAnswer:"
    
    response = openai.Completion.create(
        engine="text-davinci-003",  # You can use the latest available GPT model
        prompt=prompt,
        max_tokens=200
    )
    
    return response.choices[0].text.strip()

# Example usage:
context = "Surah 1: In the name of Allah, the Most Gracious, the Most Merciful."
question = "What is mercy?"
response = ask_openai(question, context)
print(response)

4. Flask App for Chatbot Interface

To create a simple web interface for the chatbot, we can use Flask. Install Flask first:

pip install Flask

Create a app.py file for the web server:

from flask import Flask, request, jsonify
import openai

app = Flask(__name__)

# Function to process the user query using OpenAI
openai.api_key = 'your_openai_api_key_here'

def ask_openai(question, context):
    prompt = f"Context: {context}\nQuestion: {question}\nAnswer:"
    
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=prompt,
        max_tokens=200
    )
    
    return response.choices[0].text.strip()

@app.route("/ask", methods=["POST"])
def ask():
    data = request.json
    question = data.get('question')
    surah_num = data.get('surah')
    ayah_num = data.get('ayah')
    
    # Retrieve context from the Quran (e.g., Surah and Ayah)
    context = quran_surahs[surah_num][ayah_num-1]['text']
    
    # Use OpenAI to answer the question based on context
    answer = ask_openai(question, context)
    
    return jsonify({'answer': answer})

if __name__ == "__main__":
    app.run(debug=True)

Run the Flask app with:

python app.py

5. Testing the API

You can now send POST requests to the /ask endpoint with a JSON payload containing a question, surah, and ayah.

Example JSON payload:

{
    "question": "What is mercy?",
    "surah": 1,
    "ayah": 1
}

Conclusion

This script provides:

    Text Extraction from a Quran PDF.
    Preprocessing of the Quran text into Surahs and Ayahs.
    Basic Keyword Search for matching queries with the Quran's content.
    An Advanced Querying System using OpenAI's API.
    A Flask-based Chatbot Interface where users can query the Quran based on Surah and Ayah.

You can further enhance this system by adding features like:

    Contextual search.
    More sophisticated natural language processing (NLP).
    Integration with databases for large-scale storage.

