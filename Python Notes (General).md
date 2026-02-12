

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

### 2️ Add Arguments

#### Positional Argument (required)

```python
parser.add_argument("user_prompt", type=str, help="User prompt")
```

- Required by default
    
- Access in code: `args.user_prompt`
    

#### Optional Flag (boolean)

```python
parser.add_argument("--verbose", action="store_true", help="Enable verbose output")
```

- `--verbose` → CLI flag
    
- `action="store_true"` → `True` if flag present, `False` if absent
    
- Access in code: `args.verbose`
    
- You can name it anything using `dest=`:
    

```python
parser.add_argument("--verbose", dest="show_metadata", action="store_true")
# Access with args.show_metadata
```

---

 Using Arguments in Code

```python
if args.verbose:
    print(f"User prompt: {args.user_prompt}")
    # Print metadata, tokens, etc.
else:
    print("Response only")
```

- Use **optional args in if statements** to toggle behavior
    
- Positional arguments can be used directly as variables
    

---

###  Example CLI Usage

```bash
python main.py "Hello world" --verbose   # args.verbose = True
python main.py "Hello world"             # args.verbose = False
```

---

### Key Points

- Positional → required, accessed via `args.<name>`
    
- Optional (`--flag`) → use `action="store_true"` for booleans
    
- `dest=` can override attribute name in `args`
    
- Always check optional flags with `if args.flag:`
    

---

## Gemini API: `types` & Messages

- **`types` module** → provides **data classes** for structured API requests.
    
    - Example: `from google.genai import types`
        

### Core classes

- **`types.Content`** → represents a **single message**
    
    - Properties:
        
        - `role` → `"user"`, `"assistant"`, `"system"`
            
        - `parts` → list of `Part` objects (text pieces)
            
- **`types.Part`** → represents a **text snippet** inside a message
    
    - Property: `text` → the actual message string
        

### How to create messages

```python
messages = [types.Content(role="user", parts=[types.Part(text=args.user_prompt)])]
```

- `messages` → list of `Content` objects representing a **conversation**
    
- Each `Content` object can have multiple `Part`s
    
- Later, append assistant or system messages to `messages` for multi-turn context
    

### Sending messages to Gemini

```python
response = client.models.generate_content(model="gemini-2.5-flash", contents=messages)1
```

- Gemini reads the **list of `Content` objects** and generates a response based on conversation context
    

### Notes / Best Practices

- Always store the user prompt in a variable → easy to print/log
    
- Each message must have a role (`user` / `assistant` / `system`)
    
- Use multiple `Content` objects in `messages` for **multi-turn conversations**
    
- `Part` allows multiple pieces of text per message, though usually just one
    

---
