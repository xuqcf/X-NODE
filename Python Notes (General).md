

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

# Get Files — Secure Directory Listing (LLM Tool)

## What this lesson is about

We’re giving our AI agent its **first real ability**:  
👉 **looking at files on the system**

But we do this **carefully**.

The agent:

- can list files in a directory
    
- can see file names and sizes
    
- can tell files vs folders
    
- **cannot** look outside a directory we choose
    

Since LLMs only work with **text**, this function must:

- take a directory path
    
- return a **string** describing the contents
    
- never crash — always return text, even on errors
    

---

## Project structure

```
project_root/
├── calculator/
│   ├── main.py
│   ├── pkg/
│   │   ├── calculator.py
│   │   └── render.py
│   └── tests.py
└── functions/
    └── get_files_info.py
```

- `calculator/` → **working directory** (allowed area)
    
- `functions/` → where we store LLM tools
    
- `get_files_info.py` → tool that lists files
    

---

## The core idea (important)

- The **LLM chooses** the directory it wants to inspect
    
- **We choose** the working directory
    
- The LLM can **only see inside** the working directory
    

This prevents:

- reading system files
    
- leaking secrets
    
- deleting or overwriting random files
    

This pattern will be reused for **every tool** we give the agent.

---

## Function signature

```python
def get_files_info(working_directory, directory="."):
```

- `working_directory`  
    The root folder we allow access to (e.g. `"calculator"`)
    
- `directory`  
    A **relative path** inside `working_directory`
    
    - `"."` → current directory
        
    - `"pkg"` → subfolder
        
    - `"../"` → ❌ blocked
        

---

## Step 1 — Convert working directory to absolute path

```python
working_dir_abs = os.path.abspath(working_directory)
```

Why:

- Relative paths depend on where the script runs
    
- Absolute paths are stable and safe to compare
    

Example:

```
"calculator"
→ "/home/user/ai-agent-project/calculator"
```

---

## Step 2 — Build the target directory path

```python
target_dir = os.path.normpath(
    os.path.join(working_dir_abs, directory)
)
```

What’s happening here:

- `os.path.join()`  
    Safely combines paths (handles slashes correctly)
    
- `os.path.normpath()`  
    Cleans the path:
    
    - removes `..`
        
    - removes `.`
        
    - simplifies weird paths
        

This protects against path tricks like `"../../etc"`

---

## Step 3 — Validate directory is inside working directory

```python
valid_target_dir = (
    os.path.commonpath([working_dir_abs, target_dir])
    == working_dir_abs
)
```

Plain English:

> “Does the target directory live **inside** the working directory?”

Why this matters:

- Prevents directory escape
    
- Stops the LLM from accessing `/bin`, `/etc`, or home directories
    

If invalid:

```python
return f'Error: Cannot list "{directory}" as it is outside the permitted working directory'
```

---

## Step 4 — Ensure the path is actually a directory

```python
if not os.path.isdir(target_dir):
    return f'Error: "{directory}" is not a directory'
```

Handles cases like:

- file instead of folder
    
- directory doesn’t exist
    

Again: return a **string**, never crash.

---

## Step 5 — Iterate over files in the directory

```python
for item in os.listdir(target_dir):
```

- `os.listdir()` returns file/folder names
    
- These are **not full paths**
    

So we build full paths:

```python
item_path = os.path.join(target_dir, item)
```

---

## Step 6 — Collect metadata for each item

```python
size = os.path.getsize(item_path)
is_dir = os.path.isdir(item_path)
```

We record:

- file/folder name
    
- size in bytes
    
- whether it’s a directory
    

---

## Step 7 — Build output as text

```python
lines.append(
    f"- {item}: file_size={size} bytes, is_dir={is_dir}"
)
```

Example output:

```
- main.py: file_size=719 bytes, is_dir=False
- pkg: file_size=44 bytes, is_dir=True
```

Join everything into one string:

```python
return "\n".join(lines)
```

---

## Step 8 — Error handling (very important)

All tool functions must **always return a string**.

So we wrap everything in:

```python
try:
    ...
except (OSError, ValueError) as e:
    return f"Error: {e}"
```

This ensures:

- no crashes
    
- LLM receives readable error messages
    
- agent can react intelligently
    

---

## Full function (reference)

```python
import os

def get_files_info(working_directory, directory="."):
    try:
        working_dir_abs = os.path.abspath(working_directory)
        target_dir = os.path.abspath(
            os.path.normpath(
                os.path.join(working_dir_abs, directory)
            )
        )

        if os.path.commonpath([working_dir_abs, target_dir]) != working_dir_abs:
            return f'Error: Cannot list "{directory}" as it is outside the permitted working directory'

        if not os.path.isdir(target_dir):
            return f'Error: "{directory}" is not a directory'

        lines = []

        for item in os.listdir(target_dir):
            item_path = os.path.join(target_dir, item)
            size = os.path.getsize(item_path)
            is_dir = os.path.isdir(item_path)

            lines.append(
                f"- {item}: file_size={size} bytes, is_dir={is_dir}"
            )

        return "\n".join(lines)

    except (OSError, ValueError) as e:
        return f"Error: {e}"
```

---

## Manual testing (debugging)

Create `test_get_files_info.py` in project root.

```python
from functions.get_files_info import get_files_info

def main():
    print("Result for current directory:")
    print(get_files_info("calculator", "."))
    print()

    print("Result for 'pkg' directory:")
    print(get_files_info("calculator", "pkg"))
    print()

    print("Result for '/bin' directory:")
    print(get_files_info("calculator", "/bin"))
    print()

    print("Result for '../' directory:")
    print(get_files_info("calculator", "../"))

if __name__ == "__main__":
    main()
```

Run:

```bash
uv run test_get_files_info.py
```

---

## Expected behavior (not exact sizes)

### Valid paths

- Lists files
    
- Shows size + directory status
    

### Invalid paths

- `/bin` → ❌ blocked
    
- `../` → ❌ blocked
    

Always returns a **string**, never crashes.

---

## Useful functions (plain English)

- `os.path.abspath()` → get full path
    
- `os.path.join()` → combine paths safely
    
- `os.path.normpath()` → clean paths (`..`, `.`)
    
- `os.path.commonpath()` → check directory containment
    
- `os.listdir()` → list directory contents
    
- `os.path.isdir()` → is this a folder?
    
- `os.path.isfile()` → is this a file?
    
- `os.path.getsize()` → size in bytes
    
- `"\n".join()` → turn list of lines into one string
    

---

## Key takeaways

- LLM tools must be **restricted**
    
- Always validate paths
    
- Always return strings
    
- Never trust user/LLM input
    
- This pattern applies to **every future tool**
    

---
