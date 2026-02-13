This is the simplified version of the [sophisticated](obsidian://open?vault=X-NODE&file=AI%20Agent%20in%20Python%20Project%20Notes%20-%20SIMPLIFIED) version 

---

# Python + File System & OS Concepts

## 1. OS Module (`os`)

- Python built-in module to **interact with the operating system**.
    
- Used for:
    
    - File and directory operations
        
    - Path manipulations
        
    - Environment variables
        
    - Running system commands indirectly (with `subprocess`)
        

### Common Functions

|Function|Purpose|
|---|---|
|`os.path.abspath(path)`|Converts `path` to an absolute path (stable reference, independent of current working directory)|
|`os.path.normpath(path)`|Normalizes path, resolving `..`, `.`, and redundant slashes|
|`os.path.join(path1, path2, ...)`|Safely join multiple path segments, independent of OS (`/` vs `\`)|
|`os.path.commonpath([path1, path2, ...])`|Returns the common prefix of paths; used to **check directory containment**|
|`os.path.isfile(path)`|Returns `True` if path exists and is a **regular file**|
|`os.path.isdir(path)`|Returns `True` if path exists and is a **directory**|
|`os.makedirs(path, exist_ok=True)`|Creates directories recursively; `exist_ok=True` avoids error if folder exists|
|`os.path.dirname(path)`|Returns the parent directory of the given path|

**Notes / Best Practices**

- Always use **absolute paths** for safety in scripts.
    
- Normalize paths to prevent path traversal attacks (`..`, `.`, weird slashes).
    
- Combine paths with `os.path.join` instead of string concatenation.
    
- Use `commonpath` to ensure a user/LLM cannot access directories outside allowed root.
    

---

## 2. Reading Files in Python

```python
with open(file_path, "r") as f:
    content = f.read(1000)  # reads first 1000 characters
```

- `"r"` → read mode
    
- `f.read(n)` → reads **at most** n characters
    
- `f.read()` → reads the **entire file**
    
- `with open(...)` → automatically closes file after block ends
    
- For large files: read in chunks to avoid memory issues
    

**Advanced Tip**: You can check file size with `os.path.getsize(file_path)` before reading to handle huge files.

---

## 3. Writing Files in Python

```python
with open(file_path, "w") as f:
    f.write(content)
```

- `"w"` → overwrite file
    
- `"a"` → append to existing file
    
- `"x"` → create new file, **fails if exists**
    
- Ensure **parent directories exist**:
    

```python
os.makedirs(os.path.dirname(file_path), exist_ok=True)
```

- `len(content)` → number of characters written
    

**Advanced Tip**: Use `with open(..., buffering=...)` for better performance with large files.

---

## 4. Subprocess Module (`subprocess`)

- Run external commands/scripts from Python.
    
- Example: running another Python script with arguments
    

```python
import subprocess

result = subprocess.run(
    ["python", "script.py", "arg1", "arg2"],
    text=True,            # decode output as string
    capture_output=True,  # capture stdout and stderr
    cwd="working_dir",    # set working directory
    timeout=30            # stop if running >30s
)
```

- `result` is a `CompletedProcess` object:
    
    - `result.stdout` → command output
        
    - `result.stderr` → command error messages
        
    - `result.returncode` → exit code (0 = success, non-zero = error)
        

**Tips**

- Always handle errors:
    

```python
if result.returncode != 0:
    print("Error:", result.stderr)
```

- Use `cwd` to run scripts in a safe, controlled directory.
    
- Can pass optional args dynamically via a list:
    

```python
args = ["--verbose", "input.txt"]
subprocess.run(["python", "script.py"] + args)
```

---

## 5. Safe File & Directory Handling Pattern

1. Convert paths to **absolute paths** (`os.path.abspath`)
    
2. Normalize paths (`os.path.normpath`)
    
3. Ensure the path is **inside allowed root** using `os.path.commonpath`
    
4. Check if path is a file (`os.path.isfile`) or folder (`os.path.isdir`)
    
5. Create missing directories (`os.makedirs(..., exist_ok=True)`) before writing
    
6. Wrap code in `try/except` to **always return error messages**, not crash
    

**Why**: prevents path traversal attacks, crashes, and allows safe LLM tools for directories.

---

## 6. Gemini API — Python Integration

### Environment & Setup

- `.env` file → store secrets (never commit!)
    

```env
GEMINI_API_KEY="your_key_here"
```

- `load_dotenv()` → load `.env` variables
    
- `os.environ.get("VAR_NAME")` → read environment variable, returns `None` if missing
    

**Best Practice:** raise error if key missing:

```python
api_key = os.environ.get("GEMINI_API_KEY")
if not api_key:
    raise ValueError("Missing API Key")
```

---

### Client & API Calls

```python
from google import genai

client = genai.Client(api_key=api_key)
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Hello world"
)
print(response.text)
```

- `response` → object with generated text and metadata
    
- `response.text` → generated output
    
- `response.usage_metadata` → token usage:
    

```python
response.usage_metadata.prompt_token_count      # tokens in your prompt
response.usage_metadata.candidates_token_count  # tokens generated by model
```

**Best Practice:** wrap API calls in `main()` or functions.

---

## 7. Command-Line Arguments (`argparse`)

- Lets scripts accept **dynamic input** from CLI.
    
- Basic setup:
    

```python
import argparse

parser = argparse.ArgumentParser(description="My script")
parser.add_argument("prompt", type=str, help="User prompt")
parser.add_argument("--verbose", action="store_true", help="Print extra info")
args = parser.parse_args()

print(args.prompt)
if args.verbose:
    print("Verbose mode enabled")
```

**Notes**

- Positional arguments → required, accessed via `args.<name>`
    
- Optional flags → `--flag`, boolean with `action="store_true"`
    
- Use `dest=` to rename attribute:
    

```python
parser.add_argument("--verbose", dest="show_metadata", action="store_true")
```

- Access: `args.show_metadata`
    

**CLI Usage**

```bash
python main.py "Hello world" --verbose
python main.py "Hello world"
```

---

## 8. Gemini API — `types` & Structured Messages

- `from google.genai import types` → defines structured objects
    

### Core Classes

- `types.Content` → represents a **message**
    
    - `role`: `"user"`, `"assistant"`, `"system"`
        
    - `parts`: list of `types.Part`
        
- `types.Part` → represents a **text snippet**
    
    - `text` → message string
        

### Example

```python
messages = [
    types.Content(role="user", parts=[types.Part(text=args.user_prompt)])
]
response = client.models.generate_content(model="gemini-2.5-flash", contents=messages)
```

- For multi-turn conversations, append multiple `Content` objects.
    
- Always assign `role` to each message.
    

---

## 9. Secure Directory Listing (LLM Tool Example)

- LLM tools must be **restricted** to allowed directories.
    
- **Do not allow access outside root folder.**
    

### Safe Pattern

```python
import os

def get_files_info(working_directory, directory="."):
    try:
        working_dir_abs = os.path.abspath(working_directory)
        target_dir = os.path.abspath(os.path.normpath(os.path.join(working_dir_abs, directory)))

        if os.path.commonpath([working_dir_abs, target_dir]) != working_dir_abs:
            return f'Error: Cannot list "{directory}" outside working directory'

        if not os.path.isdir(target_dir):
            return f'Error: "{directory}" is not a directory'

        lines = []
        for item in os.listdir(target_dir):
            item_path = os.path.join(target_dir, item)
            size = os.path.getsize(item_path)
            is_dir = os.path.isdir(item_path)
            lines.append(f"- {item}: file_size={size} bytes, is_dir={is_dir}")

        return "\n".join(lines)
    except (OSError, ValueError) as e:
        return f"Error: {e}"
```

**Notes**

- Always return a string — never crash
    
- Check paths before reading (`commonpath`)
    
- Protect against directory traversal (`..`, `/etc`)
    
- Works for multi-level directories safely
    

---

## 10. Useful Python Functions (Quick Reference)

- `os.path.abspath()` → absolute path
    
- `os.path.normpath()` → normalize path (`..`, `.`)
    
- `os.path.join()` → safely combine paths
    
- `os.path.commonpath()` → check directory containment
    
- `os.listdir()` → list files/folders in directory
    
- `os.path.isdir()` → True if folder
    
- `os.path.isfile()` → True if file
    
- `os.path.getsize()` → file size in bytes
    
- `"\n".join(list_of_strings)` → combine lines into single string
    

---
[[#10. Useful Python Functions (Quick Reference)]]
## 11. Key Takeaways

- Always validate user/LLM input paths
    
- Return **strings** for error handling
    
- Normalize paths and check directory containment
    
- Wrap API and file calls in `try/except`
    
- Use `argparse` for dynamic CLI inputs
    
- Gemini API: store prompts, tokens, and usage metadata carefully
    
- `types.Content` + `types.Part` → multi-turn conversation support
    

---

If you want, I can **also make a fully linked Obsidian vault structure** with this organized into separate notes (`OS module`, `File Handling`, `Subprocess`, `Gemini API`, `Argparse`, `Secure LLM Tool`) for **easy navigation inside Obsidian**, so you can quickly jump between topics.

Do you want me to do that next?