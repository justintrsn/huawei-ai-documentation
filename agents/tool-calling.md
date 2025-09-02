# 🔧 Tool Calling: Giving AI Dangerous Capabilities
## *Or: How to Let AI Control Your Infrastructure and Pretend Everything Is Fine*

---

## The Beautiful Lie

**What they tell you:** "Tool calling lets AI agents interact with external systems in a controlled, safe way!"

**The brutal truth:** Tool calling is like giving your car keys to a teenager who just learned to drive from YouTube videos. They know what the pedals do, but not why the garage door should be open before backing out.

---

## Prerequisites

Before diving into this particular nightmare, make sure you've read:
- [Introduction to AI Agents](introduction_to_agents.md) - Why we're doing this to ourselves
- [MCP Protocol Guide](mcp-guide.md) - The protocol that makes this madness possible
- [JSON: The Format That's Almost But Not Quite Human-Readable](/prerequisites/json-suffering.md) - You'll be debugging a lot of JSON

This guide assumes you understand why your AI needs tools and have accepted that giving it actual capabilities is both necessary and terrifying.

---

## 📚 Table of Contents

- [What Tool Calling Actually Is (Spoiler: Controlled Chaos)](#what-tool-calling-actually-is)
- [The Security Nightmare You're Signing Up For](#security-nightmare)
- [Tool Design: Making Footguns Safer](#tool-design-principles)
- [Parameter Validation: Trust Nothing](#parameter-validation)
- [Error Handling: Expecting Failure](#error-handling)
- [Auth: Who Gets to Break Things](#auth-and-authz)
- [Rate Limiting: Saving You From Bankruptcy](#rate-limiting)
- [Testing: Finding Bugs Before 3 AM](#testing-tools)
- [Production: Where Dreams Go to Die](#production-patterns)

---

## What Tool Calling Actually Is

### The Evolutionary Timeline

```
2019: "Our chatbot can tell you the weather!"
      (Returns: "I cannot access weather data")

2021: "Our AI can write code!"
      (Returns: Code that doesn't compile)

2023: "Our AI can execute functions!"
      (Returns: ExecutionError: Database deleted)

2024: "Our AI safely executes functions!"
      (Returns: SafeExecutionError: Database safely deleted)

2025: "We've given up and just monitor everything"
      (At least we know when things break)
```

### What's Really Happening

When an AI "calls a tool," here's the beautiful disaster that unfolds:

```python
# What the AI thinks it's doing:
result = send_email("john@example.com", "Meeting", "Let's meet")

# What actually happens:
1. AI generates JSON (probably malformed)
2. You parse the JSON (error handling #1)
3. You validate parameters (error handling #2)
4. You check permissions (error handling #3)
5. You rate limit (error handling #4)
6. You actually call the function (error handling #5)
7. The function fails anyway (error handling #6)
8. You return an error to the AI
9. AI tries again with the same parameters
10. Repeat until rate limit exceeded
```

### The Tool Calling Stack

As we learned in the [MCP Protocol Guide](mcp-guide.md), there are layers to this madness:

```
┌─────────────────────────────┐
│      AI Model Brain         │ <- "I should send an email!"
├─────────────────────────────┤
│     Tool Selection          │ <- "I'll use send_email tool"
├─────────────────────────────┤
│   Parameter Generation      │ <- "To: boss@company.com"
├─────────────────────────────┤
│    JSON Serialization       │ <- {"to": "bos@company.com"} (typo)
├─────────────────────────────┤
│    MCP Protocol Layer       │ <- Wraps in 3 more layers of JSON
├─────────────────────────────┤
│   Your Tool Function        │ <- Validates, sanitizes, cries
├─────────────────────────────┤
│    Actual Execution         │ <- EmailService.send() (crashes)
├─────────────────────────────┤
│   Error Propagation         │ <- Bubble up through all layers
├─────────────────────────────┤
│    AI Interpretation        │ <- "The email was sent successfully!"
└─────────────────────────────┘
```

---

## Security Nightmare

### The Hierarchy of Bad Ideas

```python
# Level 1: "What Could Go Wrong?"
@tool
def execute_command(command: str) -> str:
    """Execute any system command"""
    return subprocess.run(command, shell=True, capture_output=True).stdout
    # AI: execute_command("rm -rf /")
    # You: *unemployed*

# Level 2: "Basic Sanitization"
@tool
def execute_safe_command(command: str) -> str:
    """Execute 'safe' system commands"""
    if "rm" in command or "delete" in command:
        return "Nice try"
    return subprocess.run(command, shell=True, capture_output=True).stdout
    # AI: execute_command("unlink /vmlinuz")
    # You: *still unemployed*

# Level 3: "Allowlist Approach"
@tool
def execute_allowed_command(command: Literal["ls", "pwd", "date"]) -> str:
    """Execute only allowed commands"""
    return subprocess.run(command, shell=False, capture_output=True).stdout
    # AI can only run these three commands
    # AI: *finds creative ways to break things with just 'ls'*

# Level 4: "No System Commands At All"
@tool
def get_system_info(info_type: Literal["time", "memory", "cpu"]) -> dict:
    """Get system information safely"""
    if info_type == "time":
        return {"time": datetime.now().isoformat()}
    elif info_type == "memory":
        return {"memory_percent": psutil.virtual_memory().percent}
    elif info_type == "cpu":
        return {"cpu_percent": psutil.cpu_percent()}
    # AI can't execute commands, only get specific info
    # You: *slightly less likely to be unemployed*
```

### The Principle of Least Surprise (To Your Bank Account)

```python
# BAD: The "God Mode" Tool
@tool
def database_admin(sql: str) -> list:
    """Execute any SQL query"""
    # AI can now:
    # - DROP TABLE users
    # - SELECT * FROM credit_cards
    # - UPDATE prices SET amount = 0
    return db.execute(sql)

# GOOD: Specific, Limited Tools
@tool
def get_user_count(department: Optional[str] = None) -> int:
    """Get count of users in department"""
    if department:
        # Parameterized query, no SQL injection
        return db.query(
            "SELECT COUNT(*) FROM users WHERE department = ?",
            [department]
        ).scalar()
    else:
        return db.query("SELECT COUNT(*) FROM users").scalar()

@tool
def update_user_email(user_id: int, new_email: str) -> dict:
    """Update user email with validation"""
    # Validate email format
    if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', new_email):
        raise ValueError("Invalid email format")
    
    # Check user exists
    user = db.query("SELECT id FROM users WHERE id = ?", [user_id]).first()
    if not user:
        raise ValueError("User not found")
    
    # Update with constraints
    db.execute(
        "UPDATE users SET email = ?, updated_at = ? WHERE id = ?",
        [new_email, datetime.now(), user_id]
    )
    
    return {"success": True, "user_id": user_id, "new_email": new_email}
```

### Input Sanitization: Trust No One, Especially Not AI

```python
class SafeToolParameters:
    """Sanitize everything because AI is creative with inputs"""
    
    @staticmethod
    def sanitize_string(value: str, max_length: int = 1000) -> str:
        """Sanitize string inputs"""
        if not isinstance(value, str):
            raise ValueError(f"Expected string, got {type(value)}")
        
        # Remove null bytes (database killer)
        value = value.replace('\x00', '')
        
        # Strip control characters
        value = ''.join(char for char in value if ord(char) >= 32 or char == '\n')
        
        # Limit length
        if len(value) > max_length:
            value = value[:max_length]
        
        return value
    
    @staticmethod
    def sanitize_path(path: str) -> str:
        """Sanitize file paths"""
        # No directory traversal
        if '..' in path or path.startswith('/'):
            raise ValueError("Invalid path: directory traversal detected")
        
        # No special characters that could break things
        if any(char in path for char in ['<', '>', '|', '&', ';', '$', '`']):
            raise ValueError("Invalid path: contains shell metacharacters")
        
        # Must be within allowed directory
        safe_path = os.path.join('/safe/sandbox/', path)
        
        # Resolve to absolute path and check it's still in sandbox
        abs_path = os.path.abspath(safe_path)
        if not abs_path.startswith('/safe/sandbox/'):
            raise ValueError("Path escapes sandbox")
        
        return abs_path
    
    @staticmethod
    def sanitize_email(email: str) -> str:
        """Sanitize and validate email"""
        email = email.strip().lower()
        
        # Basic format validation
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', email):
            raise ValueError(f"Invalid email format: {email}")
        
        # Check for homoglyphs and weird Unicode
        if not all(ord(char) < 128 for char in email):
            raise ValueError("Email contains non-ASCII characters")
        
        # Prevent email bombs
        if len(email) > 254:  # RFC 5321
            raise ValueError("Email too long")
        
        return email
```

---

## Tool Design Principles

### The Single Responsibility Principle (Do One Thing, Break One Thing)

```python
# BAD: The Swiss Army Knife of Disaster
@tool
def user_management(
    action: str,  # "create", "update", "delete", "ban", "unban", "merge", "split"
    user_data: dict,  # Who knows what's in here
    options: dict = {}  # Even more mystery
) -> dict:
    """Manage users with maximum confusion"""
    # 500 lines of if-elif statements
    # 50 different ways to fail
    # 0 ways to debug when it breaks at 3 AM
    pass

# GOOD: Focused Tools That Do One Thing
@tool
def create_user(
    email: str,
    name: str,
    department: Literal["sales", "engineering", "support"]
) -> dict:
    """Create a user. That's it. That's all it does."""
    # Validate email
    email = SafeToolParameters.sanitize_email(email)
    
    # Validate name
    name = SafeToolParameters.sanitize_string(name, max_length=100)
    
    # Check if user exists
    if user_exists(email):
        return {"error": "User already exists", "suggestion": "Try update_user instead"}
    
    # Create user
    user_id = db.insert("users", {
        "email": email,
        "name": name,
        "department": department,
        "created_at": datetime.now()
    })
    
    return {"success": True, "user_id": user_id}

@tool
def deactivate_user(user_id: int, reason: str) -> dict:
    """Deactivate a user. Just deactivate. Nothing else."""
    # One responsibility, one way to fail
    pass
```

### The Idempotency Principle (Call It Twice, Break It Once)

```python
# BAD: Non-idempotent chaos
@tool
def add_credit(user_id: int, amount: float) -> dict:
    """Add credit to user account"""
    # Call this twice, user gets double credit
    # AI will definitely call this multiple times
    current = get_balance(user_id)
    new_balance = current + amount
    update_balance(user_id, new_balance)
    return {"new_balance": new_balance}

# GOOD: Idempotent operations
@tool
def ensure_credit_transaction(
    transaction_id: str,  # Unique identifier
    user_id: int,
    amount: float,
    description: str
) -> dict:
    """Ensure credit transaction exists (idempotent)"""
    
    # Check if transaction already exists
    existing = db.query(
        "SELECT * FROM transactions WHERE transaction_id = ?",
        [transaction_id]
    ).first()
    
    if existing:
        # Already processed, return existing result
        return {
            "status": "already_processed",
            "transaction": existing,
            "message": "Transaction already exists"
        }
    
    # Create new transaction
    with db.transaction():
        # Insert transaction record
        db.insert("transactions", {
            "transaction_id": transaction_id,
            "user_id": user_id,
            "amount": amount,
            "description": description,
            "created_at": datetime.now()
        })
        
        # Update balance
        db.execute(
            "UPDATE users SET balance = balance + ? WHERE id = ?",
            [amount, user_id]
        )
    
    return {
        "status": "created",
        "transaction_id": transaction_id,
        "new_balance": get_balance(user_id)
    }
```

### The Explicit Interface Principle (No Surprises)

Following the documentation patterns from our [MCP Protocol Guide](mcp-guide.md):

```python
# BAD: Mystery Meat Parameters
@tool
def process(data: Any, flags: dict = None) -> Any:
    """Process data with flags"""
    # AI has no idea what to send
    # You have no idea what you'll receive
    # Nobody knows what will happen
    pass

# GOOD: Crystal Clear Interface
@tool
def analyze_customer_sentiment(
    text: str,
    language: Literal["en", "es", "fr", "de", "ja"] = "en",
    include_confidence: bool = True,
    include_keywords: bool = False,
    min_confidence: float = 0.7
) -> dict:
    """
    Analyze customer sentiment from text.
    
    Args:
        text: Customer feedback text (max 5000 chars)
        language: Language of the text
        include_confidence: Include confidence scores in results
        include_keywords: Extract key phrases from text
        min_confidence: Minimum confidence threshold (0.0-1.0)
    
    Returns:
        {
            "sentiment": "positive" | "negative" | "neutral",
            "confidence": 0.85,  # If include_confidence=True
            "keywords": ["great service", "fast delivery"],  # If include_keywords=True
            "language_detected": "en",
            "processing_time_ms": 123
        }
    
    Examples:
        >>> analyze_customer_sentiment("Great product!", "en")
        {"sentiment": "positive", "confidence": 0.95}
        
        >>> analyze_customer_sentiment("Terrible service", "en", include_keywords=True)
        {"sentiment": "negative", "confidence": 0.92, "keywords": ["terrible service"]}
    
    Raises:
        ValueError: If text is empty or too long
        ValueError: If language is not supported
        ValueError: If min_confidence is not between 0 and 1
    """
    # AI knows EXACTLY what to send
    # You know EXACTLY what to validate
    # Everyone knows what will happen (mostly)
```

---

## Parameter Validation

### The Type System Struggle

Remember from [JSON: The Format That's Almost But Not Quite Human-Readable](/prerequisites/json-suffering.md) - everything becomes JSON, and JSON has like 6 types. Here's how to make that work:

```python
from typing import Union, Optional, List, Dict, Literal, Annotated
from pydantic import BaseModel, Field, validator
from datetime import datetime, date
import json

# The Pydantic Approach (Because Manual Validation Is Pain)
class EmailTaskParameters(BaseModel):
    """Parameters for email task with full validation"""
    
    to_addresses: List[str] = Field(
        ...,
        min_items=1,
        max_items=50,
        description="List of recipient email addresses"
    )
    
    subject: str = Field(
        ...,
        min_length=1,
        max_length=200,
        description="Email subject line"
    )
    
    body_html: str = Field(
        ...,
        min_length=10,
        max_length=50000,
        description="HTML body content"
    )
    
    priority: Literal["low", "normal", "high", "urgent"] = Field(
        "normal",
        description="Email priority level"
    )
    
    send_at: Optional[datetime] = Field(
        None,
        description="Schedule send time (None for immediate)"
    )
    
    attachments: Optional[List[str]] = Field(
        None,
        max_items=10,
        description="List of attachment file paths"
    )
    
    @validator('to_addresses', each_item=True)
    def validate_email_addresses(cls, email):
        """Validate each email address"""
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', email):
            raise ValueError(f"Invalid email address: {email}")
        
        # Check against disposable email domains
        disposable_domains = ['tempmail.com', 'guerrillamail.com', '10minutemail.com']
        domain = email.split('@')[1]
        if domain in disposable_domains:
            raise ValueError(f"Disposable email addresses not allowed: {email}")
        
        return email.lower()
    
    @validator('body_html')
    def sanitize_html(cls, html):
        """Sanitize HTML content"""
        # Remove script tags (security)
        html = re.sub(r'<script[^>]*>.*?</script>', '', html, flags=re.DOTALL)
        
        # Remove onclick and other event handlers
        html = re.sub(r'on\w+\s*=\s*["\'][^"\']*["\']', '', html)
        
        return html
    
    @validator('send_at')
    def validate_send_time(cls, send_time):
        """Validate scheduled send time"""
        if send_time and send_time < datetime.now():
            raise ValueError("Cannot schedule email in the past")
        
        if send_time and send_time > datetime.now() + timedelta(days=30):
            raise ValueError("Cannot schedule email more than 30 days in advance")
        
        return send_time
    
    @validator('attachments', each_item=True)
    def validate_attachments(cls, file_path):
        """Validate attachment paths"""
        # Check file exists
        if not os.path.exists(file_path):
            raise ValueError(f"Attachment not found: {file_path}")
        
        # Check file size
        file_size = os.path.getsize(file_path)
        if file_size > 10 * 1024 * 1024:  # 10MB
            raise ValueError(f"Attachment too large: {file_path}")
        
        # Check file type (basic)
        allowed_extensions = ['.pdf', '.doc', '.docx', '.txt', '.png', '.jpg']
        if not any(file_path.endswith(ext) for ext in allowed_extensions):
            raise ValueError(f"File type not allowed: {file_path}")
        
        return file_path

@tool
def send_email_validated(params: EmailTaskParameters) -> dict:
    """Send email with full parameter validation"""
    # Pydantic has already validated everything
    # We can trust the parameters (mostly)
    
    try:
        result = email_service.send(
            to=params.to_addresses,
            subject=params.subject,
            body=params.body_html,
            priority=params.priority,
            send_at=params.send_at,
            attachments=params.attachments
        )
        
        return {
            "success": True,
            "message_id": result["id"],
            "scheduled": params.send_at is not None
        }
        
    except Exception as e:
        return {
            "success": False,
            "error": str(e),
            "suggestion": "Check email service status"
        }
```

### Runtime Validation Patterns (When Types Aren't Enough)

```python
def validate_with_context(func):
    """Decorator for validation that needs context"""
    
    @wraps(func)
    async def wrapper(*args, **kwargs):
        # Pre-execution validation
        tool_name = func.__name__
        
        # Check rate limits before validation
        if not await check_rate_limit(tool_name):
            raise RateLimitError(f"Rate limit exceeded for {tool_name}")
        
        # Validate parameters based on current state
        if 'user_id' in kwargs:
            user = await get_user(kwargs['user_id'])
            if not user:
                raise ValueError("User not found")
            
            if user['status'] == 'banned':
                raise ValueError("Cannot perform operations on banned users")
            
            # Add user context for the tool
            kwargs['_user_context'] = user
        
        # Check resource availability
        if 'file_path' in kwargs:
            if not await check_file_exists(kwargs['file_path']):
                raise ValueError(f"File not found: {kwargs['file_path']}")
            
            file_size = await get_file_size(kwargs['file_path'])
            if file_size > MAX_FILE_SIZE:
                raise ValueError(f"File too large: {file_size} bytes")
        
        # Execute with validated context
        return await func(*args, **kwargs)
    
    return wrapper
```

### The AI Parameter Confusion Matrix

```python
# What AI sends vs What we expect
class AIParameterTranslator:
    """Fix common AI parameter mistakes"""
    
    @staticmethod
    def fix_common_mistakes(tool_name: str, params: dict) -> dict:
        """Fix common parameter mistakes from different AI models"""
        
        # OpenAI loves to send strings for everything
        if "OpenAI" in get_ai_model():
            params = AIParameterTranslator._fix_openai_types(params)
        
        # Claude sometimes wraps everything in 'parameters'
        if "parameters" in params and len(params) == 1:
            params = params["parameters"]
        
        # Gemini sends dates as strings in various formats
        params = AIParameterTranslator._normalize_dates(params)
        
        # Some AIs send null instead of omitting optional parameters
        params = {k: v for k, v in params.items() if v is not None}
        
        return params
    
    @staticmethod
    def _fix_openai_types(params: dict) -> dict:
        """Convert string types to proper types"""
        for key, value in params.items():
            # Try to convert string numbers
            if isinstance(value, str):
                # Try integer
                try:
                    params[key] = int(value)
                    continue
                except ValueError:
                    pass
                
                # Try float
                try:
                    params[key] = float(value)
                    continue
                except ValueError:
                    pass
                
                # Try boolean
                if value.lower() in ['true', 'false']:
                    params[key] = value.lower() == 'true'
                    continue
                
                # Try JSON (for nested objects)
                if value.startswith('{') or value.startswith('['):
                    try:
                        params[key] = json.loads(value)
                    except json.JSONDecodeError:
                        pass
        
        return params
    
    @staticmethod
    def _normalize_dates(params: dict) -> dict:
        """Normalize date formats from various AI models"""
        date_patterns = [
            r'\d{4}-\d{2}-\d{2}',  # ISO format
            r'\d{2}/\d{2}/\d{4}',  # US format
            r'\d{2}-\d{2}-\d{4}',  # Another format
        ]
        
        for key, value in params.items():
            if isinstance(value, str) and any(re.match(p, value) for p in date_patterns):
                try:
                    # Try to parse as date
                    parsed_date = dateutil.parser.parse(value)
                    params[key] = parsed_date.isoformat()
                except:
                    pass  # Keep original if parsing fails
        
        return params
```

---

## Error Handling

### The Error Taxonomy

Building on the error patterns from [MCP Protocol Guide](mcp-guide.md), here's the complete error hierarchy:

```python
class ToolError(Exception):
    """Base class for all tool errors"""
    def __init__(self, message: str, code: str = None, details: dict = None):
        self.message = message
        self.code = code or "UNKNOWN_ERROR"
        self.details = details or {}
        self.timestamp = datetime.now().isoformat()
        super().__init__(message)
    
    def to_dict(self) -> dict:
        return {
            "error": self.message,
            "code": self.code,
            "details": self.details,
            "timestamp": self.timestamp,
            "type": self.__class__.__name__
        }

# Client Errors (4xx equivalent - AI's fault)
class ValidationError(ToolError):
    """Invalid parameters provided"""
    def __init__(self, message: str, parameter: str = None, expected: Any = None, received: Any = None):
        super().__init__(message, "VALIDATION_ERROR")
        self.details = {
            "parameter": parameter,
            "expected": str(expected),
            "received": str(received)
        }

class AuthenticationError(ToolError):
    """Authentication failed"""
    def __init__(self, message: str):
        super().__init__(message, "AUTH_ERROR")

class AuthorizationError(ToolError):
    """Authorized but not allowed"""
    def __init__(self, message: str, required_permission: str = None):
        super().__init__(message, "FORBIDDEN")
        self.details = {"required_permission": required_permission}

class ResourceNotFoundError(ToolError):
    """Requested resource doesn't exist"""
    def __init__(self, message: str, resource_type: str = None, resource_id: str = None):
        super().__init__(message, "NOT_FOUND")
        self.details = {
            "resource_type": resource_type,
            "resource_id": resource_id
        }

# Server Errors (5xx equivalent - our fault)
class InternalError(ToolError):
    """Something broke on our side"""
    def __init__(self, message: str, original_error: Exception = None):
        super().__init__(message, "INTERNAL_ERROR")
        if original_error:
            self.details = {
                "original_error": str(original_error),
                "traceback": traceback.format_exc()
            }

class ExternalServiceError(ToolError):
    """External service failed"""
    def __init__(self, message: str, service: str = None, status_code: int = None):
        super().__init__(message, "SERVICE_ERROR")
        self.details = {
            "service": service,
            "status_code": status_code
        }

# Rate Limiting (429 equivalent)
class RateLimitError(ToolError):
    """Too many requests"""
    def __init__(self, message: str, retry_after: int = None):
        super().__init__(message, "RATE_LIMIT")
        self.details = {"retry_after_seconds": retry_after}

# The Special One
class AIConfusedError(ToolError):
    """AI sent something completely nonsensical"""
    def __init__(self, message: str, what_ai_sent: Any = None):
        super().__init__(message, "AI_CONFUSED")
        self.details = {
            "what_ai_sent": str(what_ai_sent),
            "suggestion": "Try rephrasing your request or using a different tool"
        }
```

### The Error Handler Pattern

```python
def comprehensive_error_handler(func):
    """Handle all possible errors gracefully"""
    
    @wraps(func)
    async def wrapper(*args, **kwargs):
        start_time = time.time()
        
        try:
            # Try to execute the tool
            result = await func(*args, **kwargs)
            
            # Add metadata
            execution_time = time.time() - start_time
            
            if isinstance(result, dict):
                result["_execution_time"] = execution_time
                result.setdefault("success", True)
            
            return result
            
        except ValidationError as e:
            # AI sent bad parameters
            logger.warning(f"Validation error in {func.__name__}: {e}")
            return {
                "success": False,
                "error": e.to_dict(),
                "suggestion": "Check the tool documentation for correct parameters"
            }
            
        except (AuthenticationError, AuthorizationError) as e:
            # Auth issues
            logger.warning(f"Auth error in {func.__name__}: {e}")
            return {
                "success": False,
                "error": e.to_dict(),
                "suggestion": "Check your credentials or permissions"
            }
            
        except ResourceNotFoundError as e:
            # Resource doesn't exist
            logger.info(f"Resource not found in {func.__name__}: {e}")
            return {
                "success": False,
                "error": e.to_dict(),
                "suggestion": "Verify the resource exists"
            }
            
        except RateLimitError as e:
            # Too many requests
            logger.warning(f"Rate limit hit in {func.__name__}: {e}")
            return {
                "success": False,
                "error": e.to_dict(),
                "suggestion": f"Wait {e.details.get('retry_after_seconds', 60)} seconds"
            }
            
        except ExternalServiceError as e:
            # External service failed
            logger.error(f"External service error in {func.__name__}: {e}")
            return {
                "success": False,
                "error": e.to_dict(),
                "suggestion": "The external service is having issues. Try again later."
            }
            
        except AIConfusedError as e:
            # AI sent nonsense
            logger.error(f"AI confusion in {func.__name__}: {e}")
            return {
                "success": False,
                "error": e.to_dict(),
                "suggestion": "The AI seems confused. Try a simpler request."
            }
            
        except Exception as e:
            # Unknown error (the worst kind)
            logger.error(f"Unexpected error in {func.__name__}: {e}\n{traceback.format_exc()}")
            return {
                "success": False,
                "error": {
                    "message": "An unexpected error occurred",
                    "code": "UNKNOWN_ERROR",
                    "details": str(e) if DEBUG_MODE else "Contact support"
                },
                "suggestion": "This shouldn't have happened. We're as surprised as you are."
            }
    
    return wrapper

# Apply to all tools automatically
def register_tool_with_error_handling(func):
    """Register tool with automatic error handling"""
    return comprehensive_error_handler(func)
```

### Retry Strategies (Because Failure Is Temporary, Sometimes)

```python
class RetryStrategy:
    """Different retry strategies for different failures"""
    
    @staticmethod
    async def exponential_backoff(
        func: callable,
        max_retries: int = 3,
        base_delay: float = 1.0,
        max_delay: float = 60.0
    ):
        """Retry with exponential backoff"""
        
        last_exception = None
        
        for attempt in range(max_retries):
            try:
                return await func()
                
            except (ValidationError, AuthorizationError) as e:
                # Don't retry client errors
                raise
                
            except RateLimitError as e:
                # Use the suggested retry time
                wait_time = e.details.get('retry_after_seconds', base_delay * (2 ** attempt))
                await asyncio.sleep(wait_time)
                last_exception = e
                
            except (ExternalServiceError, InternalError) as e:
                # Retry with exponential backoff
                wait_time = min(base_delay * (2 ** attempt), max_delay)
                
                # Add jitter to prevent thundering herd
                jitter = random.uniform(0, wait_time * 0.1)
                await asyncio.sleep(wait_time + jitter)
                last_exception = e
                
            except Exception as e:
                # Unknown error, try once more with delay
                if attempt < max_retries - 1:
                    await asyncio.sleep(base_delay)
                last_exception = e
        
        # All retries exhausted
        raise last_exception or Exception("All retry attempts failed")
    
    @staticmethod
    async def circuit_breaker(
        func: callable,
        failure_threshold: int = 5,
        recovery_timeout: int = 60,
        half_open_requests: int = 3
    ):
        """Circuit breaker pattern to prevent cascading failures"""
        
        if not hasattr(func, '_circuit_state'):
            func._circuit_state = {
                'state': 'CLOSED',  # CLOSED, OPEN, HALF_OPEN
                'failure_count': 0,
                'last_failure_time': None,
                'half_open_successes': 0
            }
        
        state = func._circuit_state
        
        # Check circuit state
        if state['state'] == 'OPEN':
            # Check if we should try recovery
            if time.time() - state['last_failure_time'] > recovery_timeout:
                state['state'] = 'HALF_OPEN'
                state['half_open_successes'] = 0
            else:
                raise ExternalServiceError("Circuit breaker is OPEN")
        
        try:
            # Try to execute
            result = await func()
            
            # Success - update circuit state
            if state['state'] == 'HALF_OPEN':
                state['half_open_successes'] += 1
                if state['half_open_successes'] >= half_open_requests:
                    # Fully recover
                    state['state'] = 'CLOSED'
                    state['failure_count'] = 0
            elif state['state'] == 'CLOSED':
                # Reset failure count on success
                state['failure_count'] = 0
            
            return result
            
        except Exception as e:
            # Failure - update circuit state
            state['failure_count'] += 1
            state['last_failure_time'] = time.time()
            
            if state['state'] == 'HALF_OPEN':
                # Failed during recovery, reopen immediately
                state['state'] = 'OPEN'
            elif state['failure_count'] >= failure_threshold:
                # Too many failures, open circuit
                state['state'] = 'OPEN'
            
            raise
```

---

## Auth and AuthZ

### The Security Layers (Like an Onion, Makes You Cry)

```python
from functools import wraps
import jwt
from typing import Set, Dict, Optional

class ToolSecurity:
    """Handle authentication and authorization for tools"""
    
    def __init__(self, secret_key: str):
        self.secret_key = secret_key
        self.permission_cache = {}  # Cache permissions for performance
        
    def require_auth(self, required_permissions: Set[str] = None):
        """Decorator requiring authentication and optional permissions"""
        
        def decorator(func):
            @wraps(func)
            async def wrapper(*args, **kwargs):
                # Extract auth from various possible locations
                auth_token = (
                    kwargs.pop('_auth_token', None) or
                    kwargs.pop('auth_token', None) or
                    get_auth_from_context() or
                    get_auth_from_headers()
                )
                
                if not auth_token:
                    raise AuthenticationError("No authentication token provided")
                
                # Verify token
                try:
                    payload = jwt.decode(
                        auth_token,
                        self.secret_key,
                        algorithms=["HS256"]
                    )
                except jwt.ExpiredSignatureError:
                    raise AuthenticationError("Token has expired")
                except jwt.InvalidTokenError as e:
                    raise AuthenticationError(f"Invalid token: {e}")
                
                # Extract user info
                user_id = payload.get('user_id')
                user_role = payload.get('role', 'user')
                
                # Check permissions if required
                if required_permissions:
                    user_permissions = await self.get_user_permissions(user_id, user_role)
                    
                    missing_permissions = required_permissions - user_permissions
                    if missing_permissions:
                        raise AuthorizationError(
                            f"Missing required permissions: {missing_permissions}",
                            required_permission=list(missing_permissions)[0]
                        )
                
                # Inject user context
                kwargs['_user_context'] = {
                    'user_id': user_id,
                    'role': user_role,
                    'permissions': user_permissions if required_permissions else None
                }
                
                # Execute tool with auth context
                return await func(*args, **kwargs)
            
            return wrapper
        return decorator
    
    async def get_user_permissions(self, user_id: str, role: str) -> Set[str]:
        """Get user permissions (with caching)"""
        
        cache_key = f"{user_id}:{role}"
        
        # Check cache first
        if cache_key in self.permission_cache:
            cached_time, permissions = self.permission_cache[cache_key]
            if time.time() - cached_time < 300:  # 5 minute cache
                return permissions
        
        # Role-based permissions
        role_permissions = {
            'admin': {'read', 'write', 'delete', 'admin', 'financial'},
            'manager': {'read', 'write', 'delete', 'financial'},
            'user': {'read', 'write'},
            'guest': {'read'},
            'ai_agent': {'read', 'write', 'tool_execute'}  # Special role for AI
        }
        
        permissions = role_permissions.get(role, set())
        
        # Add user-specific permissions from database
        user_permissions = await get_user_permissions_from_db(user_id)
        permissions.update(user_permissions)
        
        # Cache the result
        self.permission_cache[cache_key] = (time.time(), permissions)
        
        return permissions

# Initialize security
security = ToolSecurity(secret_key=os.getenv("JWT_SECRET", "change-me-please"))

# Example tools with different permission levels
@security.require_auth({'read'})
@tool
async def get_user_profile(user_id: int, _user_context: dict = None) -> dict:
    """Get user profile (requires read permission)"""
    
    # Check if user is accessing their own profile or is admin
    if _user_context['user_id'] != user_id and 'admin' not in _user_context.get('permissions', set()):
        # Allow reading own profile or admin reading any profile
        raise AuthorizationError("Can only view your own profile")
    
    profile = await fetch_user_profile(user_id)
    return {"success": True, "profile": profile}

@security.require_auth({'write', 'financial'})
@tool
async def process_payment(
    amount: float,
    currency: str,
    description: str,
    _user_context: dict = None
) -> dict:
    """Process payment (requires write AND financial permissions)"""
    
    # Additional validation for financial operations
    if amount > 10000 and 'admin' not in _user_context.get('permissions', set()):
        raise AuthorizationError("Large transactions require admin approval")
    
    # Process the payment
    result = await payment_gateway.process(amount, currency, description)
    
    # Audit log
    await audit_log(
        user_id=_user_context['user_id'],
        action='process_payment',
        details={'amount': amount, 'currency': currency}
    )
    
    return {"success": True, "transaction_id": result['id']}
```

### API Key Management (For Service-to-Service Auth)

```python
class APIKeyManager:
    """Manage API keys for different services"""
    
    def __init__(self):
        self.keys = {}  # service -> key info
        self.usage = defaultdict(int)  # Track usage
        
    async def validate_api_key(self, api_key: str) -> Optional[dict]:
        """Validate API key and return service info"""
        
        # Hash the key for comparison (never store plain keys)
        key_hash = hashlib.sha256(api_key.encode()).hexdigest()
        
        # Look up key in database
        service_info = await db.query(
            "SELECT * FROM api_keys WHERE key_hash = ? AND active = true",
            [key_hash]
        ).first()
        
        if not service_info:
            return None
        
        # Check rate limits
        if self.usage[api_key] >= service_info['rate_limit']:
            raise RateLimitError(
                f"API key rate limit exceeded",
                retry_after=60
            )
        
        # Update usage
        self.usage[api_key] += 1
        
        return service_info
    
    def require_api_key(self, scopes: Set[str] = None):
        """Decorator for API key authentication"""
        
        def decorator(func):
            @wraps(func)
            async def wrapper(*args, **kwargs):
                # Get API key from various sources
                api_key = (
                    kwargs.pop('api_key', None) or
                    get_header('X-API-Key') or
                    get_query_param('api_key')
                )
                
                if not api_key:
                    raise AuthenticationError("API key required")
                
                # Validate key
                service_info = await self.validate_api_key(api_key)
                if not service_info:
                    raise AuthenticationError("Invalid API key")
                
                # Check scopes
                if scopes:
                    service_scopes = set(service_info['scopes'].split(','))
                    if not scopes.issubset(service_scopes):
                        raise AuthorizationError("Insufficient API key scopes")
                
                # Add service context
                kwargs['_service_context'] = service_info
                
                return await func(*args, **kwargs)
            
            return wrapper
        return decorator

api_keys = APIKeyManager()

@api_keys.require_api_key({'data:read'})
@tool
async def export_data(format: str = "json", _service_context: dict = None) -> dict:
    """Export data (requires API key with data:read scope)"""
    
    # Log API usage
    await audit_log(
        service=_service_context['service_name'],
        action='export_data',
        format=format
    )
    
    # Export data
    data = await export_system_data(format)
    
    return {"success": True, "data": data}
```

---

## Rate Limiting

### Protecting Your Wallet and Sanity

Building on the patterns from [MCP Protocol Guide](mcp-guide.md), here's comprehensive rate limiting:

```python
from collections import defaultdict, deque
from datetime import datetime, timedelta
import asyncio

class RateLimiter:
    """Multi-strategy rate limiter"""
    
    def __init__(self):
        self.strategies = {}
        
    def add_strategy(self, name: str, strategy):
        """Add a rate limiting strategy"""
        self.strategies[name] = strategy
        
    def rate_limit(
        self,
        strategy: str = "token_bucket",
        **kwargs
    ):
        """Decorator for rate limiting tools"""
        
        def decorator(func):
            @wraps(func)
            async def wrapper(*args, **kwargs):
                # Get the appropriate strategy
                limiter = self.strategies.get(strategy)
                if not limiter:
                    raise ValueError(f"Unknown rate limit strategy: {strategy}")
                
                # Check rate limit
                user_id = kwargs.get('_user_context', {}).get('user_id', 'anonymous')
                
                if not await limiter.allow_request(user_id, func.__name__):
                    wait_time = await limiter.get_wait_time(user_id, func.__name__)
                    raise RateLimitError(
                        f"Rate limit exceeded for {func.__name__}",
                        retry_after=int(wait_time)
                    )
                
                # Execute tool
                return await func(*args, **kwargs)
            
            return wrapper
        return decorator

class TokenBucketLimiter:
    """Token bucket rate limiting strategy"""
    
    def __init__(self, capacity: int = 10, refill_rate: float = 1.0):
        self.capacity = capacity
        self.refill_rate = refill_rate  # tokens per second
        self.buckets = defaultdict(lambda: {
            'tokens': capacity,
            'last_refill': datetime.now()
        })
    
    async def allow_request(self, user_id: str, tool_name: str) -> bool:
        """Check if request is allowed"""
        
        key = f"{user_id}:{tool_name}"
        bucket = self.buckets[key]
        now = datetime.now()
        
        # Refill tokens
        elapsed = (now - bucket['last_refill']).total_seconds()
        tokens_to_add = elapsed * self.refill_rate
        bucket['tokens'] = min(self.capacity, bucket['tokens'] + tokens_to_add)
        bucket['last_refill'] = now
        
        # Check if token available
        if bucket['tokens'] >= 1:
            bucket['tokens'] -= 1
            return True
        
        return False
    
    async def get_wait_time(self, user_id: str, tool_name: str) -> float:
        """Get time to wait for next token"""
        
        key = f"{user_id}:{tool_name}"
        bucket = self.buckets[key]
        
        if bucket['tokens'] >= 1:
            return 0
        
        # Calculate time for next token
        tokens_needed = 1 - bucket['tokens']
        wait_time = tokens_needed / self.refill_rate
        
        return wait_time

class SlidingWindowLimiter:
    """Sliding window rate limiting strategy"""
    
    def __init__(self, window_size: int = 60, max_requests: int = 10):
        self.window_size = window_size  # seconds
        self.max_requests = max_requests
        self.requests = defaultdict(deque)
    
    async def allow_request(self, user_id: str, tool_name: str) -> bool:
        """Check if request is allowed"""
        
        key = f"{user_id}:{tool_name}"
        now = datetime.now()
        cutoff = now - timedelta(seconds=self.window_size)
        
        # Remove old requests
        while self.requests[key] and self.requests[key][0] < cutoff:
            self.requests[key].popleft()
        
        # Check limit
        if len(self.requests[key]) >= self.max_requests:
            return False
        
        # Add this request
        self.requests[key].append(now)
        return True
    
    async def get_wait_time(self, user_id: str, tool_name: str) -> float:
        """Get time until next request allowed"""
        
        key = f"{user_id}:{tool_name}"
        
        if len(self.requests[key]) < self.max_requests:
            return 0
        
        # Time until oldest request expires
        oldest = self.requests[key][0]
        now = datetime.now()
        wait_time = self.window_size - (now - oldest).total_seconds()
        
        return max(0, wait_time)

# Initialize rate limiters
rate_limiter = RateLimiter()
rate_limiter.add_strategy('token_bucket', TokenBucketLimiter(capacity=10, refill_rate=0.5))
rate_limiter.add_strategy('sliding_window', SlidingWindowLimiter(window_size=60, max_requests=30))

# Apply different strategies to different tools
@rate_limiter.rate_limit(strategy='token_bucket')
@tool
async def expensive_ai_operation(prompt: str) -> dict:
    """Expensive operation with token bucket limiting"""
    # This can be called 10 times initially, then once every 2 seconds
    result = await call_expensive_ai_api(prompt)
    return {"success": True, "result": result}

@rate_limiter.rate_limit(strategy='sliding_window')
@tool
async def send_notification(user_id: int, message: str) -> dict:
    """Send notification with sliding window limiting"""
    # Maximum 30 notifications per 60 seconds
    await notification_service.send(user_id, message)
    return {"success": True, "sent_at": datetime.now().isoformat()}
```

---

## Testing Tools

### The Testing Pyramid (It's Actually a House of Cards)

```python
import pytest
from unittest.mock import AsyncMock, patch, MagicMock
import asyncio

# Unit Tests (The Foundation)
class TestToolValidation:
    """Test parameter validation"""
    
    @pytest.mark.asyncio
    async def test_email_validation_success(self):
        """Test valid email passes validation"""
        params = EmailTaskParameters(
            to_addresses=["valid@example.com"],
            subject="Test Subject",
            body_html="<p>Test body</p>"
        )
        
        assert params.to_addresses == ["valid@example.com"]
    
    @pytest.mark.asyncio
    async def test_email_validation_failure(self):
        """Test invalid email fails validation"""
        with pytest.raises(ValueError) as exc_info:
            EmailTaskParameters(
                to_addresses=["not-an-email"],
                subject="Test",
                body_html="<p>Test</p>"
            )
        
        assert "Invalid email address" in str(exc_info.value)
    
    @pytest.mark.parametrize("email,should_pass", [
        ("valid@example.com", True),
        ("user+tag@example.co.uk", True),
        ("not.an.email", False),
        ("@example.com", False),
        ("user@", False),
        ("user@tempmail.com", False),  # Disposable email
    ])
    def test_email_formats(self, email, should_pass):
        """Test various email formats"""
        if should_pass:
            # Should not raise
            validate_email(email)
        else:
            with pytest.raises(ValueError):
                validate_email(email)

# Integration Tests (Where Things Start Breaking)
class TestToolIntegration:
    """Test tools with mocked services"""
    
    @pytest.fixture
    def mock_email_service(self):
        """Mock email service"""
        service = AsyncMock()
        service.send.return_value = {"id": "msg_123", "status": "sent"}
        return service
    
    @pytest.mark.asyncio
    async def test_send_email_success(self, mock_email_service):
        """Test successful email sending"""
        
        with patch('your_module.email_service', mock_email_service):
            result = await send_email_validated(
                EmailTaskParameters(
                    to_addresses=["test@example.com"],
                    subject="Test",
                    body_html="<p>Test</p>"
                )
            )
        
        assert result["success"] is True
        assert result["message_id"] == "msg_123"
        mock_email_service.send.assert_called_once()
    
    @pytest.mark.asyncio
    async def test_send_email_service_failure(self, mock_email_service):
        """Test email service failure handling"""
        
        mock_email_service.send.side_effect = ExternalServiceError("Service down")
        
        with patch('your_module.email_service', mock_email_service):
            result = await send_email_validated(
                EmailTaskParameters(
                    to_addresses=["test@example.com"],
                    subject="Test",
                    body_html="<p>Test</p>"
                )
            )
        
        assert result["success"] is False
        assert "Service down" in result["error"]

# End-to-End Tests (Where Dreams Die)
class TestToolE2E:
    """Test complete tool workflows"""
    
    @pytest.mark.asyncio
    @pytest.mark.slow  # Mark slow tests
    async def test_complete_user_workflow(self):
        """Test creating, updating, and deleting a user"""
        
        # Create user
        create_result = await create_user(
            email="e2e_test@example.com",
            name="E2E Test User",
            department="engineering"
        )
        
        assert create_result["success"] is True
        user_id = create_result["user_id"]
        
        try:
            # Update user
            update_result = await update_user_email(
                user_id=user_id,
                new_email="e2e_updated@example.com"
            )
            
            assert update_result["success"] is True
            
            # Verify update
            user = await get_user_by_id(user_id)
            assert user["email"] == "e2e_updated@example.com"
            
        finally:
            # Cleanup (always cleanup in E2E tests)
            await delete_user(user_id)
    
    @pytest.mark.asyncio
    async def test_rate_limiting_enforcement(self):
        """Test that rate limiting actually works"""
        
        # Make requests up to limit
        results = []
        for i in range(15):  # Assuming limit is 10
            result = await expensive_ai_operation(f"Test prompt {i}")
            results.append(result)
            
            # Small delay to prevent thundering herd
            await asyncio.sleep(0.1)
        
        # Count successes and rate limit errors
        successes = sum(1 for r in results if r.get("success"))
        rate_limited = sum(1 for r in results if r.get("error", {}).get("code") == "RATE_LIMIT")
        
        assert successes <= 10  # Should respect limit
        assert rate_limited > 0  # Should have some rate limited

# Chaos Testing (For the Brave)
class TestToolChaos:
    """Test tools under adverse conditions"""
    
    @pytest.mark.asyncio
    async def test_concurrent_requests(self):
        """Test tool under concurrent load"""
        
        async def make_request(i):
            return await get_user_list(limit=10)
        
        # Make 100 concurrent requests
        tasks = [make_request(i) for i in range(100)]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # Some should succeed
        successes = sum(1 for r in results if isinstance(r, dict) and r.get("success"))
        assert successes > 0
        
        # Some might be rate limited
        errors = sum(1 for r in results if isinstance(r, Exception)]
        # That's fine, we're testing chaos
    
    @pytest.mark.asyncio
    async def test_memory_exhaustion(self):
        """Test tool behavior when memory is constrained"""
        
        # Try to process huge data
        huge_data = "x" * (100 * 1024 * 1024)  # 100MB string
        
        with pytest.raises((MemoryError, ValidationError)):
            await process_large_data(huge_data)
    
    @pytest.mark.asyncio
    async def test_timeout_behavior(self):
        """Test tool timeout handling"""
        
        with patch('your_module.slow_operation') as mock_op:
            # Make operation take forever
            async def slow_op():
                await asyncio.sleep(1000)
            
            mock_op.side_effect = slow_op
            
            # Should timeout
            with pytest.raises(asyncio.TimeoutError):
                await asyncio.wait_for(
                    slow_operation_tool(),
                    timeout=5.0
                )
```

### Mocking Strategies (Because Real Services Cost Money)

```python
# Mock fixtures for testing
@pytest.fixture
def mock_ai_context():
    """Mock AI agent context"""
    return {
        'user_id': 'ai_agent_123',
        'role': 'ai_agent',
        'permissions': {'read', 'write', 'tool_execute'},
        'model': 'gpt-4',
        'session_id': 'session_456'
    }

@pytest.fixture
def mock_database():
    """Mock database for testing"""
    db = AsyncMock()
    db.query.return_value.first.return_value = {"id": 1, "name": "Test"}
    db.execute.return_value = None
    db.insert.return_value = 1
    return db

@pytest.fixture
def mock_external_services():
    """Mock all external services"""
    with patch.multiple(
        'your_module',
        email_service=AsyncMock(),
        payment_gateway=AsyncMock(),
        notification_service=AsyncMock(),
        ai_service=AsyncMock()
    ) as mocks:
        yield mocks
```

---

## Production Patterns

### The Production Checklist (Things You'll Forget)

```python
# production_config.py
@dataclass
class ProductionToolConfig:
    """Production configuration for tools"""
    
    # Timeouts (because everything hangs eventually)
    default_timeout: int = 30
    ai_operation_timeout: int = 120
    file_operation_timeout: int = 60
    
    # Rate limits (because money)
    rate_limit_per_user: int = 100  # per hour
    rate_limit_per_tool: Dict[str, int] = None
    
    # Resource limits (because OOM)
    max_memory_mb: int = 512
    max_cpu_percent: float = 50
    max_concurrent_tools: int = 100
    
    # Retry configuration (because networks)
    max_retries: int = 3
    retry_delay: float = 1.0
    retry_backoff: float = 2.0
    
    # Circuit breaker (because cascading failures)
    circuit_failure_threshold: int = 5
    circuit_recovery_timeout: int = 60
    
    # Monitoring (because observability)
    enable_metrics: bool = True
    enable_tracing: bool = True
    enable_profiling: bool = False  # Only when debugging
    
    # Security (because hackers)
    enable_auth: bool = True
    require_https: bool = True
    allowed_origins: List[str] = None
    
    def __post_init__(self):
        if self.rate_limit_per_tool is None:
            self.rate_limit_per_tool = {
                'expensive_ai_operation': 10,
                'send_email': 50,
                'process_payment': 20
            }
        
        if self.allowed_origins is None:
            self.allowed_origins = [
                'https://your-app.com',
                'https://api.your-app.com'
            ]

# Load config from environment
config = ProductionToolConfig(
    default_timeout=int(os.getenv('TOOL_TIMEOUT', 30)),
    enable_metrics=os.getenv('ENABLE_METRICS', 'true').lower() == 'true',
    enable_auth=os.getenv('ENABLE_AUTH', 'true').lower() == 'true'
)
```

### Monitoring and Metrics (Know When Things Break)

```python
from prometheus_client import Counter, Histogram, Gauge
import structlog

# Metrics
tool_calls = Counter('tool_calls_total', 'Total tool calls', ['tool', 'status'])
tool_duration = Histogram('tool_duration_seconds', 'Tool execution duration', ['tool'])
tool_errors = Counter('tool_errors_total', 'Tool errors', ['tool', 'error_type'])
active_tools = Gauge('active_tools', 'Currently executing tools')

# Structured logging
logger = structlog.get_logger()

def monitored_tool(func):
    """Add monitoring to tools"""
    
    @wraps(func)
    async def wrapper(*args, **kwargs):
        tool_name = func.__name__
        
        # Increment active tools
        active_tools.inc()
        
        # Log start
        logger.info(
            "tool_execution_start",
            tool=tool_name,
            args_count=len(args),
            kwargs_keys=list(kwargs.keys())
        )
        
        start_time = time.time()
        
        try:
            # Execute tool
            result = await func(*args, **kwargs)
            
            # Success metrics
            duration = time.time() - start_time
            tool_calls.labels(tool=tool_name, status='success').inc()
            tool_duration.labels(tool=tool_name).observe(duration)
            
            # Log success
            logger.info(
                "tool_execution_success",
                tool=tool_name,
                duration=duration
            )
            
            return result
            
        except Exception as e:
            # Error metrics
            duration = time.time() - start_time
            error_type = type(e).__name__
            
            tool_calls.labels(tool=tool_name, status='error').inc()
            tool_errors.labels(tool=tool_name, error_type=error_type).inc()
            tool_duration.labels(tool=tool_name).observe(duration)
            
            # Log error
            logger.error(
                "tool_execution_error",
                tool=tool_name,
                duration=duration,
                error=str(e),
                error_type=error_type
            )
            
            raise
            
        finally:
            # Decrement active tools
            active_tools.dec()
    
    return wrapper
```

### Health Checks (Is It Dead?)

```python
from fastapi import FastAPI
from typing import Dict

app = FastAPI()

@app.get("/health")
async def health_check() -> Dict:
    """Basic health check"""
    
    checks = {
        "status": "healthy",
        "timestamp": datetime.now().isoformat(),
        "checks": {}
    }
    
    # Check database
    try:
        await db.execute("SELECT 1")
        checks["checks"]["database"] = "healthy"
    except Exception as e:
        checks["checks"]["database"] = f"unhealthy: {e}"
        checks["status"] = "unhealthy"
    
    # Check external services
    for service_name, service in external_services.items():
        try:
            await service.health_check()
            checks["checks"][service_name] = "healthy"
        except Exception as e:
            checks["checks"][service_name] = f"unhealthy: {e}"
            checks["status"] = "degraded"
    
    # Check rate limiter
    if rate_limiter.is_healthy():
        checks["checks"]["rate_limiter"] = "healthy"
    else:
        checks["checks"]["rate_limiter"] = "unhealthy"
        checks["status"] = "degraded"
    
    return checks

@app.get("/ready")
async def readiness_check() -> Dict:
    """Readiness check for load balancer"""
    
    # Are we ready to serve traffic?
    if not all([
        database_connected,
        tools_loaded,
        rate_limiter_initialized
    ]):
        return {"ready": False}, 503
    
    return {"ready": True}
```

---

## Final Words

Tool calling is like giving a toddler a chainsaw - theoretically they could use it to help with yard work, but you'll spend most of your time preventing disasters and explaining to the neighbors why there's a hole in their fence.

The key to successful tool calling:

1. **Validate everything** - AI will send you garbage
2. **Limit everything** - AI will call tools infinitely
3. **Monitor everything** - You need to know what broke
4. **Test everything** - Find bugs before production finds them
5. **Document everything** - Future you will thank present you
6. **Trust nothing** - Especially not the AI's parameters
7. **Have a rollback plan** - You'll need it at 3 AM

Remember:
- Every tool is a potential security hole
- Every parameter is a potential injection attack
- Every external service will fail at the worst time
- Every rate limit will be hit
- Every timeout will be exceeded
- Every error will happen in production first

But despite all this, tool calling is what makes AI agents actually useful instead of just sophisticated chatbots. Just remember: with great power comes great opportunities for everything to go horribly wrong.

Ready to build your first agent? Check out [Building Your First Agent](first-agent.md) to put all this knowledge to use (and watch it fail spectacularly before eventually working).

---
*"Updated at 02/09/2025"*

*"Tool calling: Because what's the worst that could happen when you give an AI access to your production systems?"*

*- Anonymous AI DevOps Engineer, updating their resume*