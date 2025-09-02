# 🌐 APIs: Making Computers Talk to Each Other Badly
## *A Prerequisites Guide for AI Survivors*

---

## The Beautiful Lie

**What they tell you:** "APIs are simple! Just send HTTP requests and get responses!"

**The brutal truth:** APIs are like human communication, but worse. They're full of unstated assumptions, random failures, authentication dances, and the occasional complete nervous breakdown for no apparent reason.

---

## Why You Need This (Unfortunately)

Every AI system talks to other systems through APIs:
- Getting data from external services
- Calling AI models (OpenAI, Anthropic, Huawei Pangu)
- Receiving webhooks from user interfaces
- Coordinating between microservices

If you think "I'll just use `requests.get()`", you're about to discover why senior developers have trust issues.

---

## 📚 Table of Contents

- [HTTP Basics: The Foundation of Disappointment](#http-basics)
- [REST APIs: The Theory vs Reality](#rest-apis)
- [Authentication: The Security Theater](#authentication)
- [Error Handling: When Things Go Wrong (Spoiler: Often)](#error-handling)
- [Rate Limiting: APIs Fighting Back](#rate-limiting)
- [Real-World API Patterns in AI](#ai-api-patterns)

---

## HTTP Basics: The Foundation of Disappointment

### The HTTP Methods You'll Actually Use

```python
import requests

# GET - "Please give me data"
response = requests.get("https://api.example.com/users")
# Translation: "I hope this works"

# POST - "Please accept this data"
response = requests.post("https://api.openai.com/v1/completions", 
                        json={"prompt": "Hello AI"})
# Translation: "I'm sending you JSON, please don't reject it"

# PUT - "Replace this entire thing"
response = requests.put("https://api.example.com/users/123", 
                       json={"name": "Updated User"})
# Translation: "Overwrite everything with this"

# PATCH - "Just change these specific fields"
response = requests.patch("https://api.example.com/users/123", 
                         json={"name": "Just the name"})
# Translation: "Please be surgical about this"

# DELETE - "Make it go away"
response = requests.delete("https://api.example.com/users/123")
# Translation: "I hope this doesn't cascade delete everything"
```

### Status Codes: The API's Mood Ring

```python
# The ones you'll see constantly
if response.status_code == 200:
    # Success! Everything worked!
    data = response.json()
    
elif response.status_code == 201:
    # Created! Your POST worked!
    created_item = response.json()
    
elif response.status_code == 400:
    # Bad Request - You sent something wrong
    print("API says I'm stupid:", response.json())
    
elif response.status_code == 401:
    # Unauthorized - Your auth is bad/expired
    print("API doesn't know who I am")
    
elif response.status_code == 403:
    # Forbidden - API knows who you are, doesn't like you
    print("API knows who I am but says no anyway")
    
elif response.status_code == 404:
    # Not Found - The thing doesn't exist
    print("API can't find what I asked for")
    
elif response.status_code == 429:
    # Too Many Requests - Slow down!
    print("API is tired of me")
    
elif response.status_code == 500:
    # Internal Server Error - Their problem, not yours
    print("API broke itself")
    
elif response.status_code == 502:
    # Bad Gateway - Infrastructure problems
    print("API's infrastructure is having a moment")
    
else:
    print(f"API returned {response.status_code}, no idea what that means")
```

### Headers: The Metadata Nobody Explains

```python
# Common headers you'll need
headers = {
    "Content-Type": "application/json",  # "I'm sending JSON"
    "Accept": "application/json",        # "Please respond with JSON"
    "Authorization": "Bearer your-token-here",  # "Please don't reject me"
    "User-Agent": "MyApp/1.0",          # "This is who I am"
    "X-API-Key": "secret-key",          # Alternative auth method
}

response = requests.post(
    "https://api.example.com/data",
    headers=headers,
    json={"key": "value"}
)

# Response headers tell you things
print(f"Rate limit remaining: {response.headers.get('X-RateLimit-Remaining')}")
print(f"Server: {response.headers.get('Server')}")
print(f"Content type: {response.headers.get('Content-Type')}")
```

---

## REST APIs: The Theory vs Reality

### What REST Is Supposed to Be

REST (Representational State Transfer) has these principles:
- Stateless (each request is independent)
- Cacheable (responses can be cached)
- Uniform interface (consistent patterns)
- Client-server architecture
- Layered system

### What REST Actually Is

"We use HTTP and call it REST" - Most APIs

```python
# "RESTful" API that's not really RESTful
# But this is what you'll work with in practice

# Users endpoint
GET /api/users           # List all users
GET /api/users/123       # Get specific user
POST /api/users          # Create new user
PUT /api/users/123       # Update user (full replacement)
PATCH /api/users/123     # Update user (partial)
DELETE /api/users/123    # Delete user

# Then reality hits and you also have:
POST /api/users/123/reset-password    # Not RESTful but necessary
GET /api/users/search?q=john          # Searching is hard to make RESTful
POST /api/users/bulk-create           # Bulk operations break REST
```

### AI API Reality

```python
# OpenAI API (mostly RESTful)
response = requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": "Bearer sk-..."},
    json={
        "model": "gpt-4",
        "messages": [{"role": "user", "content": "Hello"}]
    }
)

# But then you have streaming (not RESTful at all)
response = requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={"Authorization": "Bearer sk-..."},
    json={
        "model": "gpt-4",
        "messages": [{"role": "user", "content": "Hello"}],
        "stream": True  # Now it's not REST anymore
    },
    stream=True
)

# And you process it completely differently
for line in response.iter_lines():
    if line:
        # Parse Server-Sent Events
        # Welcome to the wild west of HTTP
```

---

## Authentication: The Security Theater

### API Keys: The Basic Level

```python
# The simplest authentication
headers = {
    "X-API-Key": "your-secret-key-here",
    "Authorization": "ApiKey your-secret-key-here",  # Alternative format
}

# Or in query params (less secure)
response = requests.get(
    "https://api.example.com/data?api_key=your-secret-key"
)

# Managing API keys (please don't hardcode them)
import os
from dotenv import load_dotenv

load_dotenv()  # Loads .env file

API_KEY = os.getenv("API_KEY")
if not API_KEY:
    raise ValueError("API_KEY not found in environment")

headers = {"Authorization": f"Bearer {API_KEY}"}
```

### Bearer Tokens: The JWT Dance

```python
# JWT tokens that expire (because security)
import jwt
import datetime

def is_token_expired(token):
    try:
        # Decode without verification to check expiration
        decoded = jwt.decode(token, options={"verify_signature": False})
        exp = decoded.get('exp')
        if exp:
            return datetime.datetime.utcnow() > datetime.datetime.utcfromtimestamp(exp)
        return False
    except:
        return True  # If we can't decode it, assume it's expired

# Token refresh dance
class APIClient:
    def __init__(self, api_key):
        self.api_key = api_key
        self.access_token = None
        self.refresh_token = None
    
    def get_access_token(self):
        if not self.access_token or is_token_expired(self.access_token):
            self.refresh_access_token()
        return self.access_token
    
    def refresh_access_token(self):
        # This dance varies by API
        response = requests.post(
            "https://api.example.com/auth/token",
            json={
                "grant_type": "client_credentials",  # or "refresh_token"
                "client_id": self.api_key,
                "client_secret": os.getenv("API_SECRET")
            }
        )
        
        if response.status_code == 200:
            data = response.json()
            self.access_token = data["access_token"]
            self.refresh_token = data.get("refresh_token")
        else:
            raise Exception("Failed to get access token")
    
    def make_request(self, url, **kwargs):
        headers = kwargs.get('headers', {})
        headers['Authorization'] = f"Bearer {self.get_access_token()}"
        kwargs['headers'] = headers
        
        response = requests.request(**kwargs)
        
        # Handle token expiration during request
        if response.status_code == 401:
            self.refresh_access_token()
            headers['Authorization'] = f"Bearer {self.access_token}"
            response = requests.request(**kwargs)
        
        return response
```

### OAuth 2.0: The Authentication Nightmare

```python
# OAuth flow (simplified, reality is much worse)
import secrets
import urllib.parse

class OAuthClient:
    def __init__(self, client_id, client_secret, redirect_uri):
        self.client_id = client_id
        self.client_secret = client_secret
        self.redirect_uri = redirect_uri
        
    def get_authorization_url(self):
        state = secrets.token_urlsafe(32)  # CSRF protection
        
        params = {
            "response_type": "code",
            "client_id": self.client_id,
            "redirect_uri": self.redirect_uri,
            "scope": "read write",  # Whatever scopes you need
            "state": state,
        }
        
        base_url = "https://api.example.com/oauth/authorize"
        query_string = urllib.parse.urlencode(params)
        
        return f"{base_url}?{query_string}", state
    
    def exchange_code_for_token(self, authorization_code):
        data = {
            "grant_type": "authorization_code",
            "code": authorization_code,
            "redirect_uri": self.redirect_uri,
            "client_id": self.client_id,
            "client_secret": self.client_secret,
        }
        
        response = requests.post(
            "https://api.example.com/oauth/token",
            data=data,
            headers={"Accept": "application/json"}
        )
        
        if response.status_code == 200:
            return response.json()  # Contains access_token, refresh_token, etc.
        else:
            raise Exception(f"Token exchange failed: {response.text}")

# Usage (in a web app)
oauth = OAuthClient("your-client-id", "your-client-secret", "http://localhost:8000/callback")

# Step 1: Redirect user to authorization URL
auth_url, state = oauth.get_authorization_url()
print(f"Go to: {auth_url}")

# Step 2: User authorizes and gets redirected back with code
# http://localhost:8000/callback?code=abc123&state=xyz789

# Step 3: Exchange code for access token
tokens = oauth.exchange_code_for_token("abc123")
access_token = tokens["access_token"]

# Step 4: Use access token for API calls
headers = {"Authorization": f"Bearer {access_token}"}
response = requests.get("https://api.example.com/user", headers=headers)
```

---

## Error Handling: When Things Go Wrong (Spoiler: Often)

### The Naive Approach (Don't Do This)

```python
# This will hurt you eventually
response = requests.get("https://api.example.com/data")
data = response.json()  # What if it's not JSON? What if it's an error?
return data["results"]  # What if there's no "results" key?
```

### The Paranoid Approach (Do This)

```python
import requests
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry
import time
import json

class RobustAPIClient:
    def __init__(self, base_url, api_key, max_retries=3, backoff_factor=1):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        
        # Set up retry strategy
        retry_strategy = Retry(
            total=max_retries,
            status_forcelist=[429, 500, 502, 503, 504],  # Retry on these codes
            method_whitelist=["HEAD", "GET", "OPTIONS", "POST"],  # Which methods to retry
            backoff_factor=backoff_factor  # Wait time between retries
        )
        
        adapter = HTTPAdapter(max_retries=retry_strategy)
        self.session.mount("http://", adapter)
        self.session.mount("https://", adapter)
        
        # Default headers
        self.session.headers.update({
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json",
            "Accept": "application/json",
            "User-Agent": "MyApp/1.0"
        })
    
    def make_request(self, method, endpoint, **kwargs):
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        
        try:
            response = self.session.request(method, url, timeout=30, **kwargs)
            
            # Log the request for debugging
            print(f"{method} {url} -> {response.status_code}")
            
            # Handle different response types
            if response.status_code == 204:  # No Content
                return None
            
            if response.status_code >= 400:
                self._handle_error(response)
            
            # Try to parse JSON
            if response.headers.get('Content-Type', '').startswith('application/json'):
                return response.json()
            else:
                return response.text
            
        except requests.exceptions.Timeout:
            raise APIException("Request timed out")
        except requests.exceptions.ConnectionError:
            raise APIException("Connection error")
        except requests.exceptions.RequestException as e:
            raise APIException(f"Request failed: {str(e)}")
    
    def _handle_error(self, response):
        """Handle different types of API errors"""
        try:
            error_data = response.json()
            error_message = error_data.get('message', error_data.get('error', 'Unknown error'))
        except:
            error_message = response.text or f"HTTP {response.status_code}"
        
        if response.status_code == 401:
            raise AuthenticationError(f"Authentication failed: {error_message}")
        elif response.status_code == 403:
            raise AuthorizationError(f"Access forbidden: {error_message}")
        elif response.status_code == 404:
            raise NotFoundError(f"Resource not found: {error_message}")
        elif response.status_code == 429:
            # Extract rate limit info from headers
            reset_time = response.headers.get('X-RateLimit-Reset')
            raise RateLimitError(f"Rate limit exceeded. Reset at: {reset_time}")
        elif response.status_code >= 500:
            raise ServerError(f"Server error: {error_message}")
        else:
            raise APIException(f"API error {response.status_code}: {error_message}")

# Custom exceptions for different error types
class APIException(Exception):
    pass

class AuthenticationError(APIException):
    pass

class AuthorizationError(APIException):
    pass

class NotFoundError(APIException):
    pass

class RateLimitError(APIException):
    pass

class ServerError(APIException):
    pass

# Usage with proper error handling
client = RobustAPIClient("https://api.example.com", "your-api-key")

try:
    data = client.make_request("GET", "/users")
    print(f"Retrieved {len(data)} users")
    
except AuthenticationError:
    print("Need to refresh API key")
except RateLimitError as e:
    print(f"Rate limited: {e}")
    time.sleep(60)  # Wait before retrying
except ServerError:
    print("Server is having issues, try again later")
except APIException as e:
    print(f"API error: {e}")
```

---

## Rate Limiting: APIs Fighting Back

### Understanding Rate Limits

```python
# APIs protect themselves with rate limits
# Common patterns:
# - X requests per minute
# - X requests per hour  
# - X requests per day
# - Burst allowances (X requests in Y seconds)

def check_rate_limit_headers(response):
    """Extract rate limit info from response headers"""
    headers = response.headers
    
    # Common header patterns
    limit = headers.get('X-RateLimit-Limit') or headers.get('RateLimit-Limit')
    remaining = headers.get('X-RateLimit-Remaining') or headers.get('RateLimit-Remaining')
    reset = headers.get('X-RateLimit-Reset') or headers.get('RateLimit-Reset')
    
    print(f"Rate limit: {limit}")
    print(f"Remaining: {remaining}")
    print(f"Reset at: {reset}")
    
    return {
        'limit': int(limit) if limit else None,
        'remaining': int(remaining) if remaining else None,
        'reset': int(reset) if reset else None,
    }
```

### Rate Limiting Strategies

```python
import time
from datetime import datetime, timedelta
from collections import deque

class RateLimitedClient:
    def __init__(self, requests_per_minute=60):
        self.requests_per_minute = requests_per_minute
        self.request_times = deque()
        
    def wait_if_needed(self):
        """Implement simple rate limiting"""
        now = datetime.now()
        
        # Remove requests older than 1 minute
        cutoff = now - timedelta(minutes=1)
        while self.request_times and self.request_times[0] < cutoff:
            self.request_times.popleft()
        
        # If we're at the limit, wait
        if len(self.request_times) >= self.requests_per_minute:
            wait_time = 60 - (now - self.request_times[0]).seconds
            if wait_time > 0:
                print(f"Rate limit reached, waiting {wait_time} seconds")
                time.sleep(wait_time)
        
        # Record this request
        self.request_times.append(now)
    
    def make_request(self, *args, **kwargs):
        self.wait_if_needed()
        return requests.request(*args, **kwargs)

# Exponential backoff for 429 responses
def exponential_backoff_request(url, max_retries=5, **kwargs):
    """Retry with exponential backoff on rate limit errors"""
    for attempt in range(max_retries):
        response = requests.get(url, **kwargs)
        
        if response.status_code != 429:
            return response
        
        # Extract retry-after header if available
        retry_after = response.headers.get('Retry-After')
        if retry_after:
            wait_time = int(retry_after)
        else:
            # Exponential backoff: 1s, 2s, 4s, 8s, 16s
            wait_time = 2 ** attempt
        
        print(f"Rate limited, waiting {wait_time} seconds (attempt {attempt + 1})")
        time.sleep(wait_time)
    
    raise Exception("Max retries exceeded for rate limiting")
```

### Async Rate Limiting (For High-Performance APIs)

```python
import asyncio
import aiohttp
import time
from asyncio import Semaphore

class AsyncRateLimitedClient:
    def __init__(self, requests_per_second=10, max_concurrent=5):
        self.requests_per_second = requests_per_second
        self.semaphore = Semaphore(max_concurrent)  # Limit concurrent requests
        self.last_request_time = 0
        self.min_interval = 1.0 / requests_per_second  # Minimum time between requests
    
    async def make_request(self, session, url, **kwargs):
        async with self.semaphore:  # Limit concurrency
            # Rate limiting
            now = time.time()
            time_since_last = now - self.last_request_time
            if time_since_last < self.min_interval:
                await asyncio.sleep(self.min_interval - time_since_last)
            
            self.last_request_time = time.time()
            
            # Make the actual request
            async with session.get(url, **kwargs) as response:
                return await response.json()

# Usage
async def fetch_many_urls():
    client = AsyncRateLimitedClient(requests_per_second=5)
    
    async with aiohttp.ClientSession() as session:
        tasks = []
        urls = ["https://api.example.com/data/1", "https://api.example.com/data/2"]  # ... many URLs
        
        for url in urls:
            task = client.make_request(session, url)
            tasks.append(task)
        
        results = await asyncio.gather(*tasks)
        return results
```

---

## Real-World API Patterns in AI

### OpenAI-Style Streaming APIs

```python
import json
import sseclient  # pip install sseclient-py

def stream_openai_response(prompt):
    """Handle OpenAI's streaming response format"""
    response = requests.post(
        "https://api.openai.com/v1/chat/completions",
        headers={"Authorization": f"Bearer {os.getenv('OPENAI_API_KEY')}"},
        json={
            "model": "gpt-4",
            "messages": [{"role": "user", "content": prompt}],
            "stream": True
        },
        stream=True
    )
    
    if response.status_code != 200:
        raise Exception(f"OpenAI API error: {response.text}")
    
    # Parse Server-Sent Events
    client = sseclient.SSEClient(response)
    
    for event in client.events():
        if event.data == "[DONE]":
            break
        
        try:
            data = json.loads(event.data)
            delta = data["choices"][0]["delta"]
            
            if "content" in delta:
                yield delta["content"]  # Stream content as it arrives
                
        except json.JSONDecodeError:
            continue  # Skip malformed data
        except KeyError:
            continue  # Skip events without expected structure

# Usage
for chunk in stream_openai_response("Tell me about AI"):
    print(chunk, end="", flush=True)  # Print as it streams
```

### Huawei Cloud API Pattern

```python
class HuaweiCloudClient:
    """Huawei Cloud APIs have their own special authentication"""
    
    def __init__(self, access_key, secret_key, project_id, region="ap-southeast-3"):
        self.access_key = access_key
        self.secret_key = secret_key
        self.project_id = project_id
        self.region = region
        self.base_url = f"https://pangu.{region}.myhuaweicloud.com"
    
    def _sign_request(self, method, url, headers, body):
        """Huawei's signature algorithm (simplified)"""
        import hmac
        import hashlib
        from datetime import datetime
        
        # This is a simplified version
        # Real implementation is much more complex
        timestamp = datetime.utcnow().strftime('%Y%m%dT%H%M%SZ')
        
        string_to_sign = f"{method}\n{url}\n{timestamp}\n{body or ''}"
        signature = hmac.new(
            self.secret_key.encode(),
            string_to_sign.encode(),
            hashlib.sha256
        ).hexdigest()
        
        headers.update({
            "X-Sdk-Date": timestamp,
            "Authorization": f"SDK-HMAC-SHA256 Access={self.access_key},SignedHeaders=x-sdk-date,Signature={signature}"
        })
    
    def call_pangu_model(self, prompt, model="pangu-alpha"):
        """Call Huawei's Pangu LLM"""
        url = f"{self.base_url}/v1/inferences/{model}"
        headers = {
            "Content-Type": "application/json",
            "X-Project-Id": self.project_id
        }
        
        body = json.dumps({
            "inputs": prompt,
            "parameters": {
                "max_new_tokens": 1024,
                "temperature": 0.7
            }
        })
        
        self._sign_request("POST", url, headers, body)
        
        response = requests.post(url, headers=headers, data=body)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Huawei API error: {response.text}")

# Usage (hypothetical, check actual Huawei docs)
client = HuaweiCloudClient(
    access_key=os.getenv("HUAWEI_ACCESS_KEY"),
    secret_key=os.getenv("HUAWEI_SECRET_KEY"),
    project_id=os.getenv("HUAWEI_PROJECT_ID")
)

result = client.call_pangu_model("What is artificial intelligence?")
print(result["outputs"])
```

### Webhook Handlers (Receiving API Calls)

```python
from flask import Flask, request, jsonify
import hmac
import hashlib

app = Flask(__name__)

def verify_webhook_signature(payload, signature, secret):
    """Verify that the webhook is from the expected sender"""
    expected_signature = hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(signature, expected_signature)

@app.route("/webhook", methods=["POST"])
def handle_webhook():
    """Handle incoming webhooks"""
    # Get raw payload for signature verification
    payload = request.get_data()
    signature = request.headers.get("X-Signature-SHA256", "").replace("sha256=", "")
    
    # Verify the webhook is legitimate
    if not verify_webhook_signature(payload, signature, os.getenv("WEBHOOK_SECRET")):
        return jsonify({"error": "Invalid signature"}), 401
    
    # Parse the JSON data
    try:
        data = request.get_json()
    except:
        return jsonify({"error": "Invalid JSON"}), 400
    
    # Process the webhook
    event_type = data.get("type")
    
    if event_type == "user.created":
        handle_user_created(data["user"])
    elif event_type == "payment.completed":
        handle_payment_completed(data["payment"])
    else:
        print(f"Unknown event type: {event_type}")
    
    # Always respond with 200 to acknowledge receipt
    return jsonify({"status": "received"}), 200

def handle_user_created(user_data):
    """Handle new user creation webhook"""
    print(f"New user created: {user_data['email']}")
    # Send welcome email, create user in database, etc.

def handle_payment_completed(payment_data):
    """Handle payment completion webhook"""
    print(f"Payment completed: ${payment_data['amount']}")
    # Upgrade user account, send receipt, etc.

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

---

## API Testing and Debugging

### Using httpie for Quick Testing

```bash
# Install httpie: pip install httpie

# Simple GET request
http GET https://api.example.com/users

# POST request with JSON data
http POST https://api.example.com/users name="John" email="john@example.com"

# With authentication
http GET https://api.example.com/protected Authorization:"Bearer your-token"

# Upload a file
http --form POST https://api.example.com/upload file@/path/to/file.txt

# Custom headers
http GET https://api.example.com/data User-Agent:"MyApp/1.0" X-Custom-Header:"value"
```

### Python Testing with pytest

```python
import pytest
import responses
import requests

class TestAPIClient:
    @responses.activate  # Mock HTTP responses
    def test_successful_request(self):
        # Mock the API response
        responses.add(
            responses.GET,
            "https://api.example.com/users",
            json={"users": [{"id": 1, "name": "John"}]},
            status=200
        )
        
        # Make the request
        response = requests.get("https://api.example.com/users")
        
        # Assert the results
        assert response.status_code == 200
        data = response.json()
        assert len(data["users"]) == 1
        assert data["users"][0]["name"] == "John"
    
    @responses.activate
    def test_error_handling(self):
        # Mock an error response
        responses.add(
            responses.GET,
            "https://api.example.com/users",
            json={"error": "Server error"},
            status=500
        )
        
        # Make the request and expect an exception
        with pytest.raises(requests.exceptions.HTTPError):
            response = requests.get("https://api.example.com/users")
            response.raise_for_status()  # This should raise an exception

# Run tests with: pytest test_api.py -v
```

### Monitoring and Observability

```python
import time
import logging
from functools import wraps

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def log_api_calls(func):
    """Decorator to log API calls with timing and results"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        
        # Log the request
        logger.info(f"Starting API call: {func.__name__}")
        
        try:
            result = func(*args, **kwargs)
            duration = time.time() - start_time
            
            # Log successful completion
            logger.info(f"API call {func.__name__} completed in {duration:.2f}s")
            return result
            
        except Exception as e:
            duration = time.time() - start_time
            
            # Log the error
            logger.error(f"API call {func.__name__} failed after {duration:.2f}s: {str(e)}")
            raise
    
    return wrapper

@log_api_calls
def call_openai_api(prompt):
    """Example API call with logging"""
    response = requests.post(
        "https://api.openai.com/v1/chat/completions",
        headers={"Authorization": f"Bearer {os.getenv('OPENAI_API_KEY')}"},
        json={
            "model": "gpt-4",
            "messages": [{"role": "user", "content": prompt}]
        }
    )
    response.raise_for_status()
    return response.json()

# Usage
try:
    result = call_openai_api("What is the meaning of life?")
    print(result["choices"][0]["message"]["content"])
except Exception as e:
    print(f"API call failed: {e}")
```

---

## Final Words

APIs are like public transportation: theoretically they get you where you need to go, but you'll spend a lot of time waiting, dealing with unexpected delays, and wondering if you should have just walked instead.

The key to API success in AI development:

1. **Assume everything will fail** - Build error handling first, features second
2. **Respect rate limits** - APIs will fight back if you're too aggressive
3. **Authentication expires** - Always handle auth renewal
4. **Log everything** - You'll need those logs at 3 AM
5. **Test with mocks** - Don't depend on external APIs for your tests
6. **Cache when possible** - API calls are slow and expensive

Now you're ready to make HTTP requests like a professional pessimist! Next up: parsing the JSON responses and pretending they make sense.

---
*"Updated at 02/09/2025"*

*"The S in API stands for 'Simple'"*

*- No one, ever*