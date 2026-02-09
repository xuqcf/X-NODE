

---

## Python + Gemini API Notes

- **`.env` file** → stores secrets like API keys.
    
    - Example: `GEMINI_API_KEY='your_key_here'`
        
    - **Never** commit `.env` to Git.
        
- **`load_dotenv()`** → loads environment variables from `.env` into Python.
    
- **`os.environ.get("VAR_NAME")`** → reads an environment variable.
    
    - Returns `None` if not found.
        
    - Always check for `None` and raise error if key missing.
        
- **`genai.Client(api_key=...)`** → creates a client to talk to Google Gemini API.
    
    - Use lowercase `client` for clarity.
        
- **`client.models.generate_content(model=..., contents=...)`** → sends a prompt to a Gemini model.
    
    - `model` → name of model (e.g., `gemini-2.5-flash`).
        
    - `contents` → prompt text to generate content.
        
    - Returns a **response object**.
        
- **`response.text`** → gets the generated text from the response.
    
    
- **Best practice** → put API calls inside `main()` or functions for clean code.
    

---
