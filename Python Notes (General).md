

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

### Token Usage in Gemini API

- Most APIs return **metadata** about token usage to help monitor consumption and avoid rate limits.
    
- The `GenerateContentResponse` object has a `usage_metadata` property.
    
- `usage_metadata` includes:
    
    - **`prompt_token_count`** → number of tokens in your **prompt**
        
        ```python
        response.usage_metadata.prompt_token_count
        ```
        
    - **`candidates_token_count`** → number of tokens in the **model’s response**
        
        ```python
        response.usage_metadata.candidates_token_count
        ```
        

---

## Python `argparse` & Command-Line Arguments

- **Purpose**: Allow scripts to accept input from the command line. Useful for dynamic prompts or configs.
    

### Basic Usage

```python
import argparse

parser = argparse.ArgumentParser(description="Description of your script")
parser.add_argument("arg_name", type=str, help="Help text for this argument")
args = parser.parse_args()
```

- `ArgumentParser(description=...)` → creates the parser
    
- `add_argument(...)` → defines an argument:
    
    - `"arg_name"` → positional argument (required)
        
    - `type=...` → expected data type
        
    - `help=...` → description shown in auto help message
        
- `args = parser.parse_args()` → parses arguments passed via CLI
    
- Access argument: `args.arg_name`
    

### Features

- Positional arguments → **required by default**
    
- Optional arguments → use `--flag` or `-f`
    
- Missing required arguments → prints error and exits with code 2 automatically
    
- `args` object can hold multiple arguments
    

### Example with Gemini API

```python
import argparse
from google import genai

parser = argparse.ArgumentParser(description="Chatbot")
parser.add_argument("user_prompt", type=str, help="User prompt")
args = parser.parse_args()

prompt = args.user_prompt
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents=prompt
)
```

- `args.user_prompt` → gets the user-entered prompt string
    
- Use the string as `contents` for API calls
    
- Works like:
    

```bash
python main.py "how was burj khalifa built?"
```

### Tips

- Always store the prompt in a variable → easy to print, reuse, or log
    
- Optional arguments example:
    

```python
parser.add_argument("--model", type=str, default="gemini-2.5-flash", help="Model name")
```

- Access via `args.model`
    

---

I can also make a **super-short flashcard version** for memorizing `argparse` essentials in 3–4 lines if you want.

Do you want me to make that too?