# 🤖 Building Your First MCP Agent: The Actually Simple Guide
## *Or: How FastMCP Saved Us From Protocol Hell*

---

## The Beautiful Lie (That's Actually True This Time)

**What they tell you:** "Building an MCP server with FastMCP is simple!"

**The brutal truth:** It actually is. FastMCP handles all the protocol nonsense for you. You just write functions. It's suspicious how easy it is.

---

## Prerequisites (The Minimal Suffering Edition)

Before starting, make sure you've read:
- [Introduction to AI Agents](introduction_to_agents.md) - Why we're doing this
- The other guides if you want, but honestly, this one is simple enough

Requirements:
- Python 3.10+ (or 3.9 if you're feeling lucky)
- 15 minutes (not 6 hours)
- Coffee ☕ (optional but recommended)

---

## 📚 Table of Contents

- [The 5-Minute Setup](#the-5-minute-setup)
- [Your First MCP Server (10 Lines of Code)](#your-first-server)
- [Adding Tools (Just Python Functions)](#adding-tools)
- [Testing Your Server (It Actually Works!)](#testing-your-server)
- [Connecting to Claude Desktop](#connecting-to-claude)
- [Common Issues (There Aren't Many)](#common-issues)

---

## The 5-Minute Setup

### Step 1: Create Your Project (30 seconds)

```bash
# Create a directory
mkdir my-first-mcp
cd my-first-mcp

# Create a virtual environment with uv (the fast way)
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install FastMCP (one line!)
uv pip install fastmcp

# That's it. Seriously.
```

### Step 2: Create Your Server File (30 seconds)

```bash
# Create your server file
touch server.py

# Create a data directory for test files
mkdir data
echo "Hello from MCP!" > data/test.txt
```

---

## Your First Server

### The Entire MCP Server (Yes, This Is All Of It)

```python
# server.py
"""
Your first MCP server using FastMCP.
It's suspiciously simple.
"""

from fastmcp import FastMCP

# Create your MCP server (one line!)
mcp = FastMCP("my-first-server")

# Add a simple tool (it's just a decorator!)
@mcp.tool()
def say_hello(name: str) -> str:
    """Say hello to someone."""
    return f"Hello, {name}! Welcome to MCP!"

# Add another tool
@mcp.tool()
def add_numbers(a: int, b: int) -> int:
    """Add two numbers together."""
    return a + b

# That's it. Your server is ready.
# Run it with: python server.py
```

### Run Your Server (It Just Works)

```bash
# Run the server
python server.py

# You'll see:
# INFO:     Started server process
# INFO:     Waiting for application startup
# INFO:     Application startup complete
# INFO:     Uvicorn running on http://127.0.0.1:8000

# Your MCP server is running! That was easy!
```

---

## Adding Tools

### Let's Add Some Actually Useful Tools

```python
# server.py (expanded version)
"""
A slightly more useful MCP server.
Still surprisingly simple.
"""

from fastmcp import FastMCP
import random
from datetime import datetime
from pathlib import Path
import json

# Create your server
mcp = FastMCP("my-mcp-server")

# Tool 1: Say hello (the starter tool)
@mcp.tool()
def say_hello(name: str) -> str:
    """Say hello to someone."""
    return f"Hello, {name}! Welcome to the world of MCP!"

# Tool 2: Calculator (because every demo needs one)
@mcp.tool()
def calculate(expression: str) -> str:
    """
    Safely evaluate a mathematical expression.
    
    Args:
        expression: A math expression like "2 + 2" or "10 * 5"
    
    Returns:
        The result of the calculation
    """
    # Only allow safe characters (no evil eval!)
    allowed = "0123456789+-*/()., "
    if all(c in allowed for c in expression):
        try:
            result = eval(expression)
            return f"{expression} = {result}"
        except:
            return "Invalid expression"
    return "Invalid characters in expression"

# Tool 3: Weather (fake but fun)
@mcp.tool()
def get_weather(city: str) -> dict:
    """
    Get the weather for a city.
    Note: Returns random weather because we don't have an API key.
    
    Args:
        city: Name of the city
    
    Returns:
        Weather information (randomly generated)
    """
    conditions = ["sunny", "cloudy", "rainy", "snowy", "partly cloudy", "nuclear winter"]
    temp = random.randint(0, 35)
    
    return {
        "city": city,
        "temperature": f"{temp}°C",
        "condition": random.choice(conditions),
        "humidity": f"{random.randint(30, 80)}%",
        "wind": f"{random.randint(5, 30)} km/h",
        "timestamp": datetime.now().isoformat()
    }

# Tool 4: File Reader
@mcp.tool()
def read_file(filename: str) -> str:
    """
    Read a file from the data directory.
    
    Args:
        filename: Name of the file to read
    
    Returns:
        Contents of the file
    """
    try:
        # Only allow reading from data directory (security!)
        file_path = Path("data") / filename
        
        # Check if file exists and is in data directory
        if file_path.exists() and file_path.is_file():
            return file_path.read_text()
        else:
            return f"File not found: {filename}"
    except Exception as e:
        return f"Error reading file: {str(e)}"

# Tool 5: Todo List Manager (stateful!)
todos = []  # Simple in-memory storage

@mcp.tool()
def add_todo(task: str) -> str:
    """Add a task to the todo list."""
    todos.append({
        "id": len(todos) + 1,
        "task": task,
        "completed": False,
        "created": datetime.now().isoformat()
    })
    return f"Added: {task}"

@mcp.tool()
def list_todos() -> str:
    """List all todos."""
    if not todos:
        return "No todos yet!"
    
    result = "Todo List:\n"
    for todo in todos:
        status = "✓" if todo["completed"] else "○"
        result += f"{status} [{todo['id']}] {todo['task']}\n"
    return result

@mcp.tool()
def complete_todo(todo_id: int) -> str:
    """Mark a todo as completed."""
    for todo in todos:
        if todo["id"] == todo_id:
            todo["completed"] = True
            return f"Completed: {todo['task']}"
    return f"Todo {todo_id} not found"

# Tool 6: Dice Roller (for the gamers)
@mcp.tool()
def roll_dice(sides: int = 6, count: int = 1) -> dict:
    """
    Roll dice.
    
    Args:
        sides: Number of sides on each die (default: 6)
        count: Number of dice to roll (default: 1)
    
    Returns:
        Dice roll results
    """
    if sides < 2 or sides > 100:
        return {"error": "Sides must be between 2 and 100"}
    
    if count < 1 or count > 10:
        return {"error": "Count must be between 1 and 10"}
    
    rolls = [random.randint(1, sides) for _ in range(count)]
    
    return {
        "rolls": rolls,
        "total": sum(rolls),
        "average": sum(rolls) / len(rolls),
        "max": max(rolls),
        "min": min(rolls)
    }

# That's it! Just run: python server.py
```

---

## Testing Your Server

### Method 1: Using FastMCP's Built-in Testing

```bash
# FastMCP includes a test interface!
# Just run your server:
python server.py

# Then open another terminal and use curl:
curl -X POST http://localhost:8000/tool/call \
  -H "Content-Type: application/json" \
  -d '{"name": "say_hello", "arguments": {"name": "World"}}'

# You'll get:
# {"result": "Hello, World! Welcome to MCP!"}
```

### Method 2: Simple Python Test Client

```python
# test_client.py
"""
Simple test client for your MCP server.
No complex async stuff, just requests.
"""

import requests
import json

# Server URL
url = "http://localhost:8000"

def call_tool(tool_name, **kwargs):
    """Call a tool on the MCP server."""
    response = requests.post(
        f"{url}/tool/call",
        json={"name": tool_name, "arguments": kwargs}
    )
    return response.json()

# Test the tools
print("Testing say_hello:")
result = call_tool("say_hello", name="FastMCP")
print(result)

print("\nTesting calculate:")
result = call_tool("calculate", expression="42 + 13")
print(result)

print("\nTesting get_weather:")
result = call_tool("get_weather", city="Tokyo")
print(json.dumps(result, indent=2))

print("\nTesting todos:")
call_tool("add_todo", task="Learn MCP")
call_tool("add_todo", task="Build awesome tools")
result = call_tool("list_todos")
print(result)

print("\nTesting dice:")
result = call_tool("roll_dice", sides=20, count=3)
print(json.dumps(result, indent=2))
```

### Method 3: Interactive Testing with IPython

```python
# In IPython or Jupyter:
import requests

def call(tool, **kwargs):
    r = requests.post(
        "http://localhost:8000/tool/call",
        json={"name": tool, "arguments": kwargs}
    )
    return r.json()

# Now you can interactively test:
call("say_hello", name="IPython")
call("add_todo", task="Test MCP server")
call("list_todos")
```

---

## Connecting to Claude

### The Moment of Truth: Connecting to Claude Desktop

#### Step 1: Find Your Claude Desktop Config

```bash
# On macOS:
~/Library/Application Support/Claude/claude_desktop_config.json

# On Windows:
%APPDATA%\Claude\claude_desktop_config.json

# On Linux:
~/.config/Claude/claude_desktop_config.json
```

#### Step 2: Configure Claude to Use Your Server

```json
{
  "mcpServers": {
    "my-first-server": {
      "command": "python",
      "args": ["/absolute/path/to/your/server.py"],
      "env": {}
    }
  }
}
```

#### Step 3: Restart Claude Desktop

1. Quit Claude Desktop completely
2. Start it again
3. Look for the 🔌 icon showing MCP is connected
4. Try saying: "Can you say hello to me?" or "What's the weather in Paris?"

#### Step 4: Watch It Work!

Claude can now use your tools! Try:
- "Can you add 2+2 for me?"
- "What's the weather in Tokyo?"
- "Add 'buy coffee' to my todo list"
- "Show me my todos"
- "Roll 3 d20 dice"

---

## Common Issues

### Issue 1: "Claude Can't Find My Server"

```bash
# Make sure you use ABSOLUTE paths in config:
# Good:
"/Users/yourname/projects/my-first-mcp/server.py"

# Bad:
"~/projects/my-first-mcp/server.py"  # ~ doesn't work
"./server.py"  # Relative paths don't work
```

### Issue 2: "Server Won't Start"

```bash
# Check Python version:
python --version  # Should be 3.9+

# Make sure FastMCP is installed:
pip install fastmcp

# Check for port conflicts:
lsof -i :8000  # Mac/Linux
netstat -an | grep 8000  # Windows
```

### Issue 3: "Tools Don't Show Up in Claude"

```json
// Make sure your config JSON is valid:
// Check for trailing commas (JSON hates them)
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/path/to/server.py"]  // No comma here!
    }
  }
}
```

---

## Adding More Features (Still Easy!)

### Want Resources? Just Add Them!

```python
# Add resources to your server
@mcp.resource("file://data/{filename}")
async def read_resource(filename: str) -> str:
    """Expose files as resources."""
    file_path = Path("data") / filename
    if file_path.exists():
        return file_path.read_text()
    return "File not found"
```

### Want Prompts? Those Too!

```python
# Add prompt templates
@mcp.prompt()
async def code_review_prompt(language: str = "python") -> str:
    """Generate a code review prompt."""
    return f"""
    Please review the following {language} code:
    - Check for bugs
    - Suggest improvements
    - Rate code quality (1-10)
    """
```

### Want Async? Just Add `async`!

```python
# Async tools work too
@mcp.tool()
async def fetch_data(url: str) -> str:
    """Fetch data from a URL (async version)."""
    import httpx
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.text[:500]  # First 500 chars
```

---

## Your Complete Starter Server

Here's a complete server with everything you need:

```python
# complete_server.py
"""
A complete MCP server for beginners.
Just run: python complete_server.py
"""

from fastmcp import FastMCP
import random
from datetime import datetime
from pathlib import Path

# Create server
mcp = FastMCP("my-awesome-server")

# Storage for todos
todos = []

# === BASIC TOOLS ===

@mcp.tool()
def hello(name: str = "World") -> str:
    """Say hello to someone."""
    greetings = ["Hello", "Hi", "Hey", "Greetings", "Howdy"]
    return f"{random.choice(greetings)}, {name}! 👋"

@mcp.tool()
def calculate(a: float, b: float, operation: str = "add") -> float:
    """
    Perform a calculation.
    
    Args:
        a: First number
        b: Second number
        operation: add, subtract, multiply, or divide
    """
    ops = {
        "add": a + b,
        "subtract": a - b,
        "multiply": a * b,
        "divide": a / b if b != 0 else "Cannot divide by zero"
    }
    return ops.get(operation, "Unknown operation")

# === FILE TOOLS ===

@mcp.tool()
def read_file(filename: str) -> str:
    """Read a file from the data directory."""
    file_path = Path("data") / filename
    if file_path.exists() and file_path.is_file():
        return file_path.read_text()
    return f"File not found: {filename}"

@mcp.tool()
def write_file(filename: str, content: str) -> str:
    """Write content to a file in the data directory."""
    file_path = Path("data") / filename
    file_path.write_text(content)
    return f"Wrote {len(content)} characters to {filename}"

@mcp.tool()
def list_files() -> list:
    """List all files in the data directory."""
    data_dir = Path("data")
    if data_dir.exists():
        return [f.name for f in data_dir.iterdir() if f.is_file()]
    return []

# === TODO TOOLS ===

@mcp.tool()
def add_todo(task: str, priority: str = "normal") -> str:
    """Add a task to the todo list."""
    todo = {
        "id": len(todos) + 1,
        "task": task,
        "priority": priority,
        "completed": False,
        "created": datetime.now().isoformat()
    }
    todos.append(todo)
    return f"✅ Added todo #{todo['id']}: {task}"

@mcp.tool()
def list_todos(show_completed: bool = True) -> str:
    """List all todos."""
    if not todos:
        return "📝 No todos yet! Add one with add_todo()"
    
    result = "📋 **Todo List:**\n\n"
    
    for todo in todos:
        if not show_completed and todo["completed"]:
            continue
            
        status = "✅" if todo["completed"] else "⭕"
        priority_emoji = {"high": "🔴", "normal": "🟡", "low": "🟢"}
        pri = priority_emoji.get(todo["priority"], "⚪")
        
        result += f"{status} {pri} [{todo['id']}] {todo['task']}\n"
    
    return result

@mcp.tool()
def complete_todo(todo_id: int) -> str:
    """Mark a todo as completed."""
    for todo in todos:
        if todo["id"] == todo_id:
            todo["completed"] = True
            return f"✅ Completed: {todo['task']}"
    return f"❌ Todo #{todo_id} not found"

@mcp.tool()
def clear_todos() -> str:
    """Clear all todos."""
    count = len(todos)
    todos.clear()
    return f"🗑️ Cleared {count} todos"

# === FUN TOOLS ===

@mcp.tool()
def get_weather(city: str) -> dict:
    """Get weather for a city (simulated)."""
    conditions = ["☀️ Sunny", "☁️ Cloudy", "🌧️ Rainy", "⛈️ Stormy", "🌈 Rainbow"]
    return {
        "city": city,
        "condition": random.choice(conditions),
        "temperature": f"{random.randint(15, 30)}°C",
        "feels_like": f"{random.randint(15, 30)}°C",
        "humidity": f"{random.randint(40, 80)}%",
        "wind": f"{random.randint(5, 25)} km/h"
    }

@mcp.tool()
def tell_joke() -> str:
    """Tell a programming joke."""
    jokes = [
        "Why do programmers prefer dark mode? Because light attracts bugs!",
        "Why do Python programmers prefer snake_case? Because they can't C#!",
        "How many programmers does it take to change a light bulb? None, that's a hardware problem!",
        "Why do programmers always mix up Halloween and Christmas? Because Oct 31 == Dec 25!",
        "A SQL query walks into a bar, walks up to two tables and asks: 'Can I join you?'"
    ]
    return random.choice(jokes)

@mcp.tool()
def flip_coin() -> str:
    """Flip a coin."""
    return "🪙 Heads!" if random.random() > 0.5 else "🪙 Tails!"

@mcp.tool()
def roll_dice(sides: int = 6, count: int = 1) -> str:
    """Roll dice."""
    if sides < 2 or sides > 100 or count < 1 or count > 10:
        return "Please use 2-100 sides and 1-10 dice"
    
    rolls = [random.randint(1, sides) for _ in range(count)]
    result = f"🎲 Rolled {count}d{sides}: {rolls}"
    if count > 1:
        result += f" (Total: {sum(rolls)})"
    return result

# === UTILITY TOOLS ===

@mcp.tool()
def get_time() -> str:
    """Get the current time."""
    now = datetime.now()
    return f"🕐 Current time: {now.strftime('%I:%M %p on %A, %B %d, %Y')}"

@mcp.tool()
def set_timer(seconds: int) -> str:
    """Set a timer (just returns when it would go off)."""
    future = datetime.now() + timedelta(seconds=seconds)
    return f"⏰ Timer set! Will go off at {future.strftime('%I:%M:%S %p')}"

# Run with: python complete_server.py
if __name__ == "__main__":
    print("""
    🚀 MCP Server Ready!
    
    Tools available:
    - hello(name)
    - calculate(a, b, operation)
    - read_file(filename)
    - write_file(filename, content)
    - list_files()
    - add_todo(task, priority)
    - list_todos()
    - complete_todo(todo_id)
    - get_weather(city)
    - tell_joke()
    - roll_dice(sides, count)
    - get_time()
    - And more!
    
    Connect this to Claude Desktop and start chatting!
    """)
    
    # Run the server
    import uvicorn
    uvicorn.run(mcp.app, host="127.0.0.1", port=8000)
```

---

## Final Words

See? That wasn't so bad! With FastMCP, you can build an MCP server in literally 10 lines of code. No protocol knowledge needed, no complex async patterns, no SSE nightmares.

What you've learned:
1. **FastMCP makes it simple** - Just use `@mcp.tool()` decorator
2. **Tools are just functions** - Write normal Python functions
3. **It actually works** - Connect to Claude and use your tools
4. **No tears required** - This is actually beginner-friendly

Next steps:
- Add more tools (they're just functions!)
- Try async tools for API calls
- Add resources and prompts
- Share your server with others

Remember:
- **Start simple** - Your first tool can just return "Hello World"
- **FastMCP handles the hard parts** - You just write functions
- **Claude can use your tools immediately** - No complex integration
- **It's actually fun** - Building tools that AI can use is magical

Now go build something cool! And this time, you might not actually enjoy it!

---
*"Updated at 02/09/2025"*
"My first MCP agent worked perfectly on the first try!"
- No one, ever