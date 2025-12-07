# cURL Complete Mastery Tutorial

> **A comprehensive guide to mastering cURL - From beginner to advanced level**

**Version:** 1.0  
**Last Updated:** December 2025  
**Author:** Ninja Developer Collection  
**License:** MIT

---

## Table of Contents

### Phase 1: Introduction & Basics
- [1.1 What is cURL?](#11-what-is-curl)
- [1.2 Installation & Setup](#12-installation--setup)
- [1.3 Basic Syntax & Structure](#13-basic-syntax--structure)
- [1.4 Your First cURL Request](#14-your-first-curl-request)

### Phase 2: HTTP Methods
- [2.1 GET Requests](#21-get-requests)
- [2.2 POST Requests](#22-post-requests)
- [2.3 PUT Requests](#23-put-requests)
- [2.4 PATCH Requests](#24-patch-requests)
- [2.5 DELETE Requests](#25-delete-requests)
- [2.6 HEAD Requests](#26-head-requests)
- [2.7 OPTIONS Requests](#27-options-requests)

### Phase 3: Headers & Authentication
- [3.1 Working with Headers](#31-working-with-headers)
- [3.2 Custom Headers](#32-custom-headers)
- [3.3 Basic Authentication](#33-basic-authentication)
- [3.4 Bearer Token Authentication](#34-bearer-token-authentication)
- [3.5 API Key Authentication](#35-api-key-authentication)
- [3.6 OAuth 2.0](#36-oauth-20)

### Phase 4: Request Data Formats
- [4.1 Sending JSON Data](#41-sending-json-data)
- [4.2 Sending Form Data](#42-sending-form-data)
- [4.3 Multipart Form Data](#43-multipart-form-data)
- [4.4 File Uploads](#44-file-uploads)
- [4.5 XML Data](#45-xml-data)
- [4.6 URL Encoding](#46-url-encoding)

### Phase 5: Response Handling
- [5.1 Viewing Response Headers](#51-viewing-response-headers)
- [5.2 Status Codes](#52-status-codes)
- [5.3 Saving Responses](#53-saving-responses)
- [5.4 Silent Mode](#54-silent-mode)
- [5.5 Verbose Mode](#55-verbose-mode)
- [5.6 Response Formatting](#56-response-formatting)

### Phase 6: Advanced Options
- [6.1 Timeouts & Retries](#61-timeouts--retries)
- [6.2 Following Redirects](#62-following-redirects)
- [6.3 Cookies Management](#63-cookies-management)
- [6.4 User Agent](#64-user-agent)
- [6.5 Rate Limiting](#65-rate-limiting)
- [6.6 Compression](#66-compression)

### Phase 7: Security & SSL/TLS
- [7.1 HTTPS Requests](#71-https-requests)
- [7.2 SSL Certificates](#72-ssl-certificates)
- [7.3 Certificate Verification](#73-certificate-verification)
- [7.4 Client Certificates](#74-client-certificates)
- [7.5 Secure Options](#75-secure-options)

### Phase 8: Advanced Techniques
- [8.1 Parallel Requests](#81-parallel-requests)
- [8.2 HTTP/2 & HTTP/3](#82-http2--http3)
- [8.3 Proxy Configuration](#83-proxy-configuration)
- [8.4 DNS Resolution](#84-dns-resolution)
- [8.5 Performance Optimization](#85-performance-optimization)
- [8.6 Debugging & Troubleshooting](#86-debugging--troubleshooting)

### Phase 9: Real-world Use Cases
- [9.1 API Testing](#91-api-testing)
- [9.2 Web Scraping](#92-web-scraping)
- [9.3 File Downloads](#93-file-downloads)
- [9.4 FTP Operations](#94-ftp-operations)
- [9.5 Automation Scripts](#95-automation-scripts)
- [9.6 CI/CD Integration](#96-cicd-integration)

---

## Phase 1: Introduction & Basics

### 1.1 What is cURL?

**Description:** cURL (Client URL) is a command-line tool and library for transferring data using various network protocols. It supports HTTP, HTTPS, FTP, FTPS, SCP, SFTP, TFTP, and many more.

**Key Features:**
- Cross-platform (Windows, Linux, macOS)
- Supports multiple protocols
- Scriptable and automatable
- Extensive options for customization
- Used for API testing, file transfers, and automation
- Free and open-source

**Why Use cURL?**
- Test APIs without a GUI
- Automate file downloads/uploads
- Debug network requests
- Integrate with shell scripts
- CI/CD pipelines
- Web scraping

---

### 1.2 Installation & Setup

#### Linux (Ubuntu/Debian)
```bash
# Check if curl is installed
curl --version

# Install curl if not present
sudo apt update
sudo apt install curl

# Verify installation
curl --version
# Output: curl 7.81.0 (x86_64-pc-linux-gnu)
```

#### macOS
```bash
# curl comes pre-installed on macOS
curl --version

# If needed, install via Homebrew
brew install curl
```

#### Windows
```powershell
# Windows 10/11 comes with curl
curl --version

# Check PowerShell alias (remove if needed)
Get-Alias curl

# Remove PowerShell alias to use actual curl
Remove-Item alias:curl

# Or use curl.exe explicitly
curl.exe --version
```

**Verify Installation:**
```bash
# Display version
curl --version

# Display help
curl --help

# Display manual
curl --manual
```

---

### 1.3 Basic Syntax & Structure

**General Syntax:**
```bash
curl [options] [URL]
```

**Common Options:**

| Option | Short | Description |
|--------|-------|-------------|
| `--request` | `-X` | Specify HTTP method (GET, POST, PUT, DELETE) |
| `--data` | `-d` | Send POST data |
| `--header` | `-H` | Add custom header |
| `--user` | `-u` | Specify username:password |
| `--output` | `-o` | Write output to file |
| `--remote-name` | `-O` | Save with remote filename |
| `--location` | `-L` | Follow redirects |
| `--include` | `-i` | Include response headers |
| `--verbose` | `-v` | Verbose mode (detailed info) |
| `--silent` | `-s` | Silent mode (no progress) |
| `--fail` | `-f` | Fail silently on HTTP errors |

---

### 1.4 Your First cURL Request

**Simple GET Request:**
```bash
# Fetch a webpage
curl https://example.com

# Output:
# <!doctype html>
# <html>
# <head>
#     <title>Example Domain</title>
# ...
```

**With Response Headers:**
```bash
# Include headers in output (-i)
curl -i https://example.com

# Output:
# HTTP/2 200 
# content-type: text/html; charset=UTF-8
# date: Sun, 07 Dec 2025 21:00:00 GMT
# ...
```

**Save to File:**
```bash
# Save output to file (-o)
curl https://example.com -o example.html

# Or use remote filename (-O)
curl -O https://example.com/image.jpg
```

**Basic API Request:**
```bash
# Fetch JSON data from API
curl https://api.github.com/users/octocat

# Output:
# {
#   "login": "octocat",
#   "id": 583231,
#   "name": "The Octocat",
#   ...
# }
```

---

## Phase 2: HTTP Methods

### 2.1 GET Requests

**Description:** GET is the default HTTP method used to retrieve data from a server.

**Basic GET:**
```bash
# Simple GET request (default method)
curl https://api.example.com/users

# Explicitly specify GET method
curl -X GET https://api.example.com/users
```

**GET with Query Parameters:**
```bash
# Single parameter
curl "https://api.example.com/search?q=nodejs"

# Multiple parameters
curl "https://api.example.com/users?page=2&limit=10&sort=name"

# URL encode spaces and special characters
curl "https://api.example.com/search?q=node%20js"

# Using --get with --data (automatically URL encodes)
curl --get --data "q=nodejs" --data "page=1" https://api.example.com/search
```

**GET with Headers:**
```bash
# Add Accept header for JSON response
curl -H "Accept: application/json" https://api.example.com/users

# Add multiple headers
curl -H "Accept: application/json" \
     -H "User-Agent: MyApp/1.0" \
     https://api.example.com/users
```

---

### 2.2 POST Requests

**Description:** POST is used to send data to a server to create or update resources.

**Simple POST with Data:**
```bash
# POST with simple data
curl -X POST -d "name=John&email=john@example.com" \
     https://api.example.com/users

# POST with JSON data (recommended)
curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"name":"John","email":"john@example.com"}' \
     https://api.example.com/users
```

**POST with JSON File:**
```bash
# Create JSON file
cat > user.json << EOF
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}
EOF

# Send JSON file as POST data
curl -X POST \
     -H "Content-Type: application/json" \
     -d @user.json \
     https://api.example.com/users
```

**POST Form Data:**
```bash
# URL-encoded form data
curl -X POST \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "username=john&password=secret123" \
     https://api.example.com/login

# Or use --data-urlencode for automatic encoding
curl -X POST \
     --data-urlencode "username=john doe" \
     --data-urlencode "password=secret@123" \
     https://api.example.com/login
```

**POST with Authentication:**
```bash
# POST with bearer token
curl -X POST \
     -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
     -H "Content-Type: application/json" \
     -d '{"title":"New Post","content":"Hello World"}' \
     https://api.example.com/posts
```

---

### 2.3 PUT Requests

**Description:** PUT is used to update an existing resource completely.

**Update Resource:**
```bash
# Update user with ID 123
curl -X PUT \
     -H "Content-Type: application/json" \
     -d '{"name":"John Updated","email":"john.updated@example.com","age":31}' \
     https://api.example.com/users/123
```

**PUT from File:**
```bash
# Update from JSON file
curl -X PUT \
     -H "Content-Type: application/json" \
     -d @updated-user.json \
     https://api.example.com/users/123
```

**PUT with Authentication:**
```bash
# Update with bearer token
curl -X PUT \
     -H "Authorization: Bearer YOUR_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"status":"published"}' \
     https://api.example.com/posts/456
```

---

### 2.4 PATCH Requests

**Description:** PATCH is used for partial updates to a resource.

**Partial Update:**
```bash
# Update only specific fields
curl -X PATCH \
     -H "Content-Type: application/json" \
     -d '{"email":"newemail@example.com"}' \
     https://api.example.com/users/123

# PATCH multiple fields
curl -X PATCH \
     -H "Content-Type: application/json" \
     -d '{"age":32,"city":"New York"}' \
     https://api.example.com/users/123
```

---

### 2.5 DELETE Requests

**Description:** DELETE is used to remove a resource from the server.

**Delete Resource:**
```bash
# Delete user by ID
curl -X DELETE https://api.example.com/users/123

# Delete with authentication
curl -X DELETE \
     -H "Authorization: Bearer YOUR_TOKEN" \
     https://api.example.com/posts/456

# Delete with confirmation in body
curl -X DELETE \
     -H "Content-Type: application/json" \
     -d '{"confirm":true}' \
     https://api.example.com/users/123
```

---

### 2.6 HEAD Requests

**Description:** HEAD retrieves only headers without the response body (useful for checking if resource exists).

**Check Resource Existence:**
```bash
# Get only headers
curl -I https://example.com

# Output:
# HTTP/2 200
# content-type: text/html; charset=UTF-8
# content-length: 1256
# ...

# Using HEAD method explicitly
curl -X HEAD https://api.example.com/users/123

# Check if file exists (returns 200 or 404)
curl -I https://example.com/file.pdf
```

---

### 2.7 OPTIONS Requests

**Description:** OPTIONS shows which HTTP methods are allowed for a resource.

**Check Allowed Methods:**
```bash
# Get allowed methods
curl -X OPTIONS https://api.example.com/users -i

# Output includes:
# Allow: GET, POST, PUT, DELETE, OPTIONS
```

---

## Phase 3: Headers & Authentication

### 3.1 Working with Headers

**Description:** Headers provide additional information about the request or response.

**View Response Headers:**
```bash
# Include headers in output (-i)
curl -i https://api.example.com/users

# View only headers (-I or --head)
curl -I https://api.example.com/users

# Verbose mode shows request and response headers (-v)
curl -v https://api.example.com/users
```

---

### 3.2 Custom Headers

**Adding Custom Headers:**
```bash
# Single header
curl -H "Accept: application/json" https://api.example.com/users

# Multiple headers
curl -H "Accept: application/json" \
     -H "Content-Type: application/json" \
     -H "X-Custom-Header: value" \
     https://api.example.com/users

# Common headers
curl -H "User-Agent: MyApp/1.0" \
     -H "Accept-Language: en-US" \
     -H "Cache-Control: no-cache" \
     https://api.example.com/data
```

---

### 3.3 Basic Authentication

**Description:** Basic authentication sends username and password encoded in Base64.

**Using Basic Auth:**
```bash
# Method 1: Using -u flag (recommended)
curl -u username:password https://api.example.com/secure

# Method 2: Prompt for password (more secure)
curl -u username https://api.example.com/secure
# Enter host password for user 'username':

# Method 3: Authorization header (manual)
# echo -n "username:password" | base64
# Output: dXNlcm5hbWU6cGFzc3dvcmQ=
curl -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" \
     https://api.example.com/secure
```

**Examples:**
```bash
# GitHub API with basic auth
curl -u "username:personal_access_token" \
     https://api.github.com/user

# Protected endpoint
curl -u "admin:secretpass" \
     https://api.example.com/admin/users
```

---

### 3.4 Bearer Token Authentication

**Description:** Bearer tokens (like JWT) are used in modern API authentication.

**Using Bearer Tokens:**
```bash
# Standard bearer token format
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     https://api.example.com/protected

# Example with actual token
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"

curl -H "Authorization: Bearer $TOKEN" \
     https://api.example.com/users/me

# POST request with bearer token
curl -X POST \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"title":"New Article"}' \
     https://api.example.com/articles
```

---

### 3.5 API Key Authentication

**Description:** API keys are passed in headers or query parameters.

**Header-based API Key:**
```bash
# Custom header for API key
curl -H "X-API-Key: your-api-key-here" \
     https://api.example.com/data

# Common variations
curl -H "X-API-KEY: your-api-key" \
     https://api.example.com/data

curl -H "api-key: your-api-key" \
     https://api.example.com/data

# Multiple authentication headers
curl -H "X-API-Key: your-api-key" \
     -H "X-Client-ID: your-client-id" \
     https://api.example.com/data
```

**Query Parameter API Key:**
```bash
# API key in URL
curl "https://api.example.com/data?api_key=your-api-key"

# With additional parameters
curl "https://api.example.com/weather?city=London&apikey=your-api-key"
```

---

### 3.6 OAuth 2.0

**Description:** OAuth 2.0 is an authorization framework for secure API access.

**Getting Access Token:**
```bash
# Step 1: Get access token (client credentials flow)
curl -X POST \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=client_credentials" \
     -d "client_id=YOUR_CLIENT_ID" \
     -d "client_secret=YOUR_CLIENT_SECRET" \
     https://oauth.example.com/token

# Response:
# {
#   "access_token": "eyJhbGci...",
#   "token_type": "Bearer",
#   "expires_in": 3600
# }
```

**Using Access Token:**
```bash
# Store token in variable
ACCESS_TOKEN=$(curl -s -X POST \
     -d "grant_type=client_credentials" \
     -d "client_id=YOUR_CLIENT_ID" \
     -d "client_secret=YOUR_CLIENT_SECRET" \
     https://oauth.example.com/token | jq -r '.access_token')

# Use token for API requests
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
     https://api.example.com/protected/resource
```

**Refresh Token Flow:**
```bash
# Refresh expired token
curl -X POST \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=refresh_token" \
     -d "refresh_token=YOUR_REFRESH_TOKEN" \
     -d "client_id=YOUR_CLIENT_ID" \
     -d "client_secret=YOUR_CLIENT_SECRET" \
     https://oauth.example.com/token
```

---

## Phase 4: Request Data Formats

### 4.1 Sending JSON Data

**Description:** JSON is the most common format for API communication.

**Inline JSON:**
```bash
# Simple JSON object
curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"name":"John","age":30}' \
     https://api.example.com/users

# Complex nested JSON
curl -X POST \
     -H "Content-Type: application/json" \
     -d '{
       "user": {
         "name": "John Doe",
         "email": "john@example.com",
         "address": {
           "street": "123 Main St",
           "city": "New York",
           "zip": "10001"
         },
         "tags": ["developer", "nodejs", "javascript"]
       }
     }' \
     https://api.example.com/users
```

**JSON from File:**
```bash
# Create JSON file
cat > request.json << 'EOF'
{
  "title": "My Blog Post",
  "content": "This is the content of my blog post",
  "author": {
    "name": "John Doe",
    "email": "john@example.com"
  },
  "tags": ["tutorial", "curl", "api"],
  "published": true
}
EOF

# Send JSON file
curl -X POST \
     -H "Content-Type: application/json" \
     -d @request.json \
     https://api.example.com/posts
```

**Pretty Print JSON Response:**
```bash
# Using jq for formatting (install: apt install jq)
curl -s https://api.github.com/users/octocat | jq '.'

# Extract specific fields
curl -s https://api.github.com/users/octocat | jq '.name, .login'

# Filter and format
curl -s https://api.github.com/users/octocat/repos | jq '.[].name'
```

---

### 4.2 Sending Form Data

**Description:** Form data is URL-encoded key-value pairs.

**URL-Encoded Form:**
```bash
# Simple form data
curl -X POST \
     -d "username=john" \
     -d "password=secret123" \
     https://api.example.com/login

# Multiple fields
curl -X POST \
     -d "name=John Doe" \
     -d "email=john@example.com" \
     -d "age=30" \
     -d "subscribe=true" \
     https://api.example.com/register

# Or combine in one -d flag
curl -X POST \
     -d "username=john&password=secret123&remember=true" \
     https://api.example.com/login
```

**URL Encoding:**
```bash
# Automatic URL encoding
curl -X POST \
     --data-urlencode "name=John Doe" \
     --data-urlencode "message=Hello World!" \
     --data-urlencode "email=john+doe@example.com" \
     https://api.example.com/contact

# Special characters are automatically encoded
# Space becomes %20, @ becomes %40, etc.
```

---

### 4.3 Multipart Form Data

**Description:** Multipart form data is used for forms with file uploads and mixed data types.

**Multipart Forms:**
```bash
# Simple multipart form
curl -X POST \
     -F "name=John Doe" \
     -F "email=john@example.com" \
     -F "age=30" \
     https://api.example.com/users

# Multipart with file
curl -X POST \
     -F "username=john" \
     -F "avatar=@profile.jpg" \
     https://api.example.com/upload

# Multiple files
curl -X POST \
     -F "document1=@file1.pdf" \
     -F "document2=@file2.pdf" \
     -F "description=Important documents" \
     https://api.example.com/documents
```

---

### 4.4 File Uploads

**Description:** Upload files to servers using various methods.

**Upload Single File:**
```bash
# Upload with -F (multipart/form-data)
curl -X POST \
     -F "file=@image.jpg" \
     https://api.example.com/upload

# Specify content type
curl -X POST \
     -F "file=@document.pdf;type=application/pdf" \
     https://api.example.com/upload

# Set custom filename
curl -X POST \
     -F "file=@/path/to/file.txt;filename=custom-name.txt" \
     https://api.example.com/upload
```

**Upload Binary Data:**
```bash
# Upload file as raw binary
curl -X POST \
     -H "Content-Type: application/octet-stream" \
     --data-binary @file.zip \
     https://api.example.com/upload

# Upload image
curl -X POST \
     -H "Content-Type: image/jpeg" \
     --data-binary @photo.jpg \
     https://api.example.com/images
```

**Upload with Metadata:**
```bash
# File + JSON metadata
curl -X POST \
     -F "file=@image.jpg" \
     -F 'metadata={"title":"My Image","tags":["photo","sunset"]};type=application/json' \
     https://api.example.com/upload

# Multiple files with description
curl -X POST \
     -F "files[]=@file1.txt" \
     -F "files[]=@file2.txt" \
     -F "description=Project files" \
     -F "project_id=123" \
     https://api.example.com/project/upload
```

---

### 4.5 XML Data

**Description:** XML format for data exchange (less common than JSON).

**Send XML:**
```bash
# Create XML file
cat > request.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<user>
  <name>John Doe</name>
  <email>john@example.com</email>
  <age>30</age>
</user>
EOF

# Send XML data
curl -X POST \
     -H "Content-Type: application/xml" \
     -d @request.xml \
     https://api.example.com/users

# Inline XML
curl -X POST \
     -H "Content-Type: text/xml" \
     -d '<?xml version="1.0"?><user><name>John</name></user>' \
     https://api.example.com/users
```

---

### 4.6 URL Encoding

**Description:** Properly encode URLs and parameters.

**Manual Encoding:**
```bash
# Spaces as %20 or +
curl "https://api.example.com/search?q=hello%20world"
curl "https://api.example.com/search?q=hello+world"

# Special characters
curl "https://api.example.com/search?email=user%40example.com"

# Complex encoding
curl "https://api.example.com/search?filter=name%3DJohn%26age%3D30"
```

**Automatic Encoding:**
```bash
# Let curl encode for you
curl -G --data-urlencode "q=hello world" \
        --data-urlencode "filter=name:John Doe" \
        https://api.example.com/search
```

---

## Phase 5: Response Handling

### 5.1 Viewing Response Headers

**Description:** Inspect response headers to understand server behavior.

**Include Headers:**
```bash
# Show headers and body (-i)
curl -i https://api.example.com/users

# Output:
# HTTP/2 200
# content-type: application/json
# content-length: 1234
# date: Sun, 07 Dec 2025 21:00:00 GMT
#
# {"users":[...]}

# Only headers (-I or --head)
curl -I https://api.example.com/users

# Verbose output with timing (-v)
curl -v https://api.example.com/users
```

**Extract Specific Headers:**
```bash
# Get content-type header
curl -s -I https://api.example.com/users | grep -i content-type

# Get all headers and format
curl -s -D - https://api.example.com/users -o /dev/null
```

---

### 5.2 Status Codes

**Description:** HTTP status codes indicate the result of the request.

**Check Status Code:**
```bash
# Get HTTP status code only
curl -s -o /dev/null -w "%{http_code}" https://example.com
# Output: 200

# Verbose status
curl -s -o /dev/null -w "Status: %{http_code}\n" https://example.com
# Output: Status: 200

# Fail on HTTP errors (-f)
curl -f https://example.com/not-found
# Output: curl: (22) The requested URL returned error: 404
```

**Common Status Codes:**
```bash
# 200 OK - Success
curl -s -o /dev/null -w "%{http_code}" https://example.com
# 200

# 201 Created - Resource created
curl -X POST -d "name=John" https://api.example.com/users -w "\n%{http_code}\n"
# 201

# 204 No Content - Success with no response body
curl -X DELETE https://api.example.com/users/123 -w "%{http_code}\n"
# 204

# 400 Bad Request - Invalid request
curl -X POST https://api.example.com/users -w "%{http_code}\n"
# 400

# 401 Unauthorized - Authentication required
curl https://api.example.com/protected -w "%{http_code}\n"
# 401

# 404 Not Found - Resource not found
curl https://api.example.com/nonexistent -w "%{http_code}\n"
# 404

# 500 Internal Server Error - Server error
curl https://api.example.com/broken -w "%{http_code}\n"
# 500
```

---

### 5.3 Saving Responses

**Description:** Save response data to files.

**Save to File:**
```bash
# Using -o (--output) with custom filename
curl -o users.json https://api.example.com/users

# Using -O (--remote-name) to keep original filename
curl -O https://example.com/image.jpg

# Silent save (no progress bar)
curl -s -o data.json https://api.example.com/data

# Save headers and body separately
curl -D headers.txt -o body.json https://api.example.com/users
```

**Download Multiple Files:**
```bash
# Download multiple files
curl -O https://example.com/file1.pdf \
     -O https://example.com/file2.pdf \
     -O https://example.com/file3.pdf

# Using pattern
curl -O "https://example.com/images/photo[1-10].jpg"

# Download with retry
curl --retry 3 --retry-delay 5 -O https://example.com/large-file.zip
```

---

### 5.4 Silent Mode

**Description:** Silent mode suppresses progress meter and error messages.

**Silent Requests:**
```bash
# Silent mode (-s)
curl -s https://api.example.com/users

# Silent with show errors (-sS)
curl -sS https://api.example.com/users

# Silent save to file
curl -s -o output.json https://api.example.com/data

# Completely silent (no output)
curl -s -o /dev/null https://example.com
```

---

### 5.5 Verbose Mode

**Description:** Verbose mode shows detailed information about the request and response.

**Verbose Output:**
```bash
# Verbose mode (-v)
curl -v https://api.example.com/users

# Output includes:
# * Trying 93.184.216.34:443...
# * Connected to api.example.com
# * SSL connection details
# > GET /users HTTP/2
# > Host: api.example.com
# > User-Agent: curl/7.81.0
# > Accept: */*
# <
# < HTTP/2 200
# < content-type: application/json
# ...

# Trace request details
curl --trace trace.txt https://api.example.com/users

# ASCII trace
curl --trace-ascii trace-ascii.txt https://api.example.com/users
```

---

### 5.6 Response Formatting

**Description:** Format and process response data.

**Using Write-Out Format:**
```bash
# Display custom information
curl -w "\nStatus: %{http_code}\nTime: %{time_total}s\n" \
     -o response.json \
     https://api.example.com/users

# Multiple metrics
curl -w @- -o /dev/null -s https://example.com << 'EOF'
    HTTP Status: %{http_code}\n
    Total Time: %{time_total}s\n
    Download Speed: %{speed_download} bytes/sec\n
    Size: %{size_download} bytes\n
EOF
```

**Format JSON with jq:**
```bash
# Pretty print JSON
curl -s https://api.github.com/users/octocat | jq '.'

# Extract fields
curl -s https://api.github.com/users/octocat | jq '.name, .login, .public_repos'

# Filter arrays
curl -s https://api.github.com/users/octocat/repos | jq '.[0:5] | .[] | .name'
```

---

## Phase 6: Advanced Options

### 6.1 Timeouts & Retries

**Description:** Control request timing and retry behavior.

**Connection Timeout:**
```bash
# Connection timeout in seconds
curl --connect-timeout 10 https://slow-server.com

# Maximum time for operation
curl --max-time 30 https://api.example.com/data

# Both timeouts
curl --connect-timeout 5 --max-time 30 https://example.com
```

**Retry Options:**
```bash
# Retry on failure (default: transient errors only)
curl --retry 3 https://unreliable-api.com

# Retry with delay between attempts
curl --retry 5 --retry-delay 2 https://api.example.com

# Maximum retry time
curl --retry 5 --retry-max-time 60 https://api.example.com

# Retry on all errors (including HTTP errors)
curl --retry 3 --retry-all-errors https://api.example.com
```

**Example with Timeouts:**
```bash
# Production-ready request
curl --connect-timeout 10 \
     --max-time 30 \
     --retry 3 \
     --retry-delay 5 \
     -s -o data.json \
     https://api.example.com/data

# Check if successful
if [ $? -eq 0 ]; then
    echo "Download successful"
else
    echo "Download failed after retries"
fi
```

---

### 6.2 Following Redirects

**Description:** Handle HTTP redirects automatically.

**Follow Redirects:**
```bash
# Don't follow redirects (default)
curl https://example.com/redirect

# Follow redirects (-L or --location)
curl -L https://example.com/redirect

# Limit redirect count
curl -L --max-redirs 5 https://example.com/redirect

# Show redirect chain
curl -L -v https://example.com/redirect 2>&1 | grep -i location
```

**POST with Redirects:**
```bash
# POST data and follow redirects
curl -L -X POST -d "key=value" https://api.example.com/submit

# Preserve POST method through redirects
curl -L -X POST --post301 --post302 --post303 \
     -d "data=value" \
     https://api.example.com/submit
```

---

### 6.3 Cookies Management

**Description:** Handle cookies for session management.

**Save Cookies:**
```bash
# Save cookies to file (-c)
curl -c cookies.txt https://example.com/login

# Send cookies from file (-b)
curl -b cookies.txt https://example.com/dashboard

# Save and send cookies in same request
curl -b cookies.txt -c cookies.txt https://example.com/page
```

**Set Cookie Manually:**
```bash
# Send cookie in request
curl -b "sessionid=abc123; user=john" https://example.com/api

# Multiple cookies
curl -b "token=xyz789" -b "user=john" https://example.com/api

# Cookie with domain and path
curl -b "sessionid=abc123; Domain=.example.com; Path=/" https://example.com
```

**Login Session Example:**
```bash
# Step 1: Login and save cookies
curl -c cookies.txt \
     -X POST \
     -d "username=john&password=secret" \
     https://example.com/login

# Step 2: Use session for authenticated request
curl -b cookies.txt https://example.com/profile

# Step 3: Logout
curl -b cookies.txt -c cookies.txt https://example.com/logout
```

---

### 6.4 User Agent

**Description:** Set custom User-Agent header to identify your client.

**Custom User Agent:**
```bash
# Default User-Agent (curl/7.81.0)
curl https://example.com

# Custom User-Agent
curl -A "MyApp/1.0" https://api.example.com

# Mimic browser
curl -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
     https://example.com

# Common user agents
curl -A "MyBot/1.0 (+http://mybot.com)" https://example.com
curl -A "Mobile App iOS/14.0" https://api.example.com
```

**Empty User Agent:**
```bash
# Remove User-Agent header
curl -A "" https://example.com

# Or use header method
curl -H "User-Agent:" https://example.com
```

---

### 6.5 Rate Limiting

**Description:** Control request rate to avoid overwhelming servers.

**Delay Between Requests:**
```bash
# Bash script with delay
for i in {1..10}; do
    curl -s "https://api.example.com/data?page=$i" -o "page$i.json"
    sleep 1  # 1 second delay between requests
done

# Download files with rate limit
for url in $(cat urls.txt); do
    curl -O "$url"
    sleep 2
done
```

**Bandwidth Limiting:**
```bash
# Limit download speed (bytes/second)
curl --limit-rate 100K https://example.com/largefile.zip -O

# Limit to 1MB/s
curl --limit-rate 1M https://example.com/file.iso -O

# Limit upload speed
curl --limit-rate 50K -T upload.zip ftp://ftp.example.com/
```

---

### 6.6 Compression

**Description:** Enable compression for faster transfers.

**Accept Compressed Response:**
```bash
# Accept gzip, deflate, br compression
curl --compressed https://example.com

# Explicitly request compression
curl -H "Accept-Encoding: gzip, deflate, br" https://example.com

# Save compressed data
curl --compressed -o data.json https://api.example.com/data
```

**Upload Compressed Data:**
```bash
# Compress before upload
gzip -c largefile.txt | curl -X POST \
     -H "Content-Encoding: gzip" \
     --data-binary @- \
     https://api.example.com/upload
```

---

## Phase 7: Security & SSL/TLS

### 7.1 HTTPS Requests

**Description:** Secure communication using HTTPS protocol.

**Basic HTTPS:**
```bash
# HTTPS request (default: verify certificate)
curl https://example.com

# Show SSL/TLS handshake details
curl -v https://example.com 2>&1 | grep -i ssl

# Display certificate information
curl -v https://example.com 2>&1 | grep -A 20 "Server certificate"
```

---

### 7.2 SSL Certificates

**Description:** Work with SSL/TLS certificates.

**Certificate Information:**
```bash
# Show certificate details
curl -vI https://example.com 2>&1 | grep -A 10 "certificate"

# Save certificate
echo | openssl s_client -connect example.com:443 2>/dev/null | \
    openssl x509 -text > certificate.txt

# Check certificate expiration
curl -vI https://example.com 2>&1 | grep -i expire
```

---

### 7.3 Certificate Verification

**Description:** Control SSL certificate verification.

**Verify Certificates:**
```bash
# Default: verify certificate (secure)
curl https://example.com

# Skip certificate verification (INSECURE - use only for testing)
curl -k https://self-signed.example.com
curl --insecure https://self-signed.example.com

# Specify CA certificate
curl --cacert /path/to/ca-cert.pem https://example.com

# Use system CA bundle
curl --capath /etc/ssl/certs https://example.com
```

---

### 7.4 Client Certificates

**Description:** Authenticate using client certificates.

**Client Certificate Authentication:**
```bash
# Use client certificate and key
curl --cert client.pem --key client-key.pem https://api.example.com

# Certificate with password
curl --cert client.pem:password --key client-key.pem https://api.example.com

# Combined certificate and key file
curl --cert client-combined.pem https://api.example.com

# Specify certificate type
curl --cert client.p12:password --cert-type P12 https://api.example.com
```

---

### 7.5 Secure Options

**Description:** Additional security options.

**Security Best Practices:**
```bash
# Use TLS 1.2 or higher
curl --tlsv1.2 https://api.example.com

# Specify TLS version
curl --tlsv1.3 https://api.example.com

# Verify SSL hostname
curl --ssl-reqd https://example.com

# Disable weak ciphers (use modern ciphers only)
curl --ciphers ECDHE-RSA-AES256-GCM-SHA384 https://example.com

# Complete secure example
curl --tlsv1.2 \
     --cacert ca-bundle.crt \
     --cert client.pem \
     --key client-key.pem \
     https://secure-api.example.com
```

---

## Phase 8: Advanced Techniques

### 8.1 Parallel Requests

**Description:** Make multiple concurrent requests for better performance.

**GNU Parallel:**
```bash
# Install GNU parallel
sudo apt install parallel

# Parallel downloads
cat urls.txt | parallel -j 5 curl -O {}

# Parallel API requests
seq 1 100 | parallel -j 10 \
    'curl -s "https://api.example.com/users/{}" -o "user{}.json"'

# With custom function
parallel_curl() {
    curl -s "https://api.example.com/data?id=$1" -o "data$1.json"
}
export -f parallel_curl
seq 1 50 | parallel -j 10 parallel_curl
```

**xargs Method:**
```bash
# Parallel requests with xargs
cat urls.txt | xargs -P 5 -I {} curl -O {}

# Download with progress
cat urls.txt | xargs -P 5 -I {} sh -c 'curl -O {} && echo "Downloaded: {}"'
```

**Bash Background Jobs:**
```bash
# Simple parallel execution
for i in {1..10}; do
    curl -s "https://api.example.com/page?num=$i" -o "page$i.html" &
done
wait  # Wait for all background jobs to complete

echo "All downloads complete"
```

---

### 8.2 HTTP/2 & HTTP/3

**Description:** Use modern HTTP protocols for better performance.

**HTTP/2:**
```bash
# Use HTTP/2 (enabled by default for HTTPS)
curl --http2 https://example.com

# Force HTTP/2 (fail if not supported)
curl --http2-prior-knowledge https://example.com

# Show HTTP version used
curl -I --http2 https://example.com | grep -i HTTP
```

**HTTP/3 (QUIC):**
```bash
# Use HTTP/3 (if curl supports it)
curl --http3 https://cloudflare.com

# Try HTTP/3, fallback to HTTP/2
curl --http3-only https://example.com
```

---

### 8.3 Proxy Configuration

**Description:** Route requests through proxy servers.

**HTTP Proxy:**
```bash
# Use HTTP proxy
curl -x http://proxy.example.com:8080 https://api.example.com

# Proxy with authentication
curl -x http://user:pass@proxy.example.com:8080 https://api.example.com

# SOCKS proxy
curl -x socks5://proxy.example.com:1080 https://api.example.com

# SOCKS5 with authentication
curl -x socks5://user:pass@proxy.example.com:1080 https://api.example.com
```

**Proxy Environment Variables:**
```bash
# Set proxy via environment variable
export http_proxy="http://proxy.example.com:8080"
export https_proxy="http://proxy.example.com:8080"
export no_proxy="localhost,127.0.0.1,.local"

# Use proxies
curl https://api.example.com

# Ignore proxy for specific request
curl --noproxy "*" https://api.example.com
```

---

### 8.4 DNS Resolution

**Description:** Control DNS resolution behavior.

**Custom DNS Resolution:**
```bash
# Resolve host to specific IP
curl --resolve example.com:443:93.184.216.34 https://example.com

# Multiple resolutions
curl --resolve example.com:443:1.2.3.4 \
     --resolve api.example.com:443:5.6.7.8 \
     https://example.com

# Use specific DNS server (requires c-ares)
curl --dns-servers 8.8.8.8,8.8.4.4 https://example.com

# IPv4 only
curl -4 https://example.com

# IPv6 only
curl -6 https://example.com
```

---

### 8.5 Performance Optimization

**Description:** Optimize curl for better performance.

**Connection Reuse:**
```bash
# Keep-alive connections (default for HTTP/1.1)
curl -H "Connection: keep-alive" https://api.example.com

# Disable keep-alive
curl -H "Connection: close" https://api.example.com
```

**Performance Metrics:**
```bash
# Detailed timing information
curl -w "\n\
    DNS Lookup: %{time_namelookup}s\n\
    TCP Connect: %{time_connect}s\n\
    TLS Handshake: %{time_appconnect}s\n\
    Pre-transfer: %{time_pretransfer}s\n\
    Start Transfer: %{time_starttransfer}s\n\
    Total Time: %{time_total}s\n\
    Download Speed: %{speed_download} bytes/sec\n\
" -o /dev/null -s https://example.com

# Save metrics to file
curl -w "@curl-format.txt" -o /dev/null -s https://example.com
```

**Optimize Downloads:**
```bash
# Resume interrupted download
curl -C - -O https://example.com/large-file.zip

# Parallel download segments (if server supports range)
# (requires aria2 or similar tool)

# Download with progress bar
curl --progress-bar -O https://example.com/file.zip

# Disable progress meter
curl -s -O https://example.com/file.zip
```

---

### 8.6 Debugging & Troubleshooting

**Description:** Debug and troubleshoot curl requests.

**Verbose Debugging:**
```bash
# Verbose output
curl -v https://api.example.com

# Very verbose (includes data)
curl -vv https://api.example.com

# Trace to file
curl --trace trace.log https://api.example.com

# ASCII trace (readable)
curl --trace-ascii trace-ascii.log https://api.example.com

# Show timing
curl --trace-time https://api.example.com
```

**Error Analysis:**
```bash
# Show errors
curl --show-error https://api.example.com

# Fail on HTTP errors (4xx, 5xx)
curl -f https://api.example.com/not-found

# Write stderr to file
curl https://api.example.com 2> errors.log

# Get error code
curl -s -o /dev/null -w "%{http_code}" https://api.example.com
EXIT_CODE=$?
if [ $EXIT_CODE -ne 0 ]; then
    echo "curl failed with exit code: $EXIT_CODE"
fi
```

**Common curl Exit Codes:**
```bash
# 0   - Success
# 1   - Unsupported protocol
# 3   - URL malformed
# 5   - Couldn't resolve proxy
# 6   - Couldn't resolve host
# 7   - Failed to connect
# 22  - HTTP error (4xx, 5xx with -f)
# 28  - Timeout
# 35  - SSL connect error
# 52  - Empty response from server
# 56  - Failure in receiving network data
```

---

## Phase 9: Real-world Use Cases

### 9.1 API Testing

**Description:** Test RESTful APIs with curl.

**Complete API Testing Example:**
```bash
# Set base URL and token
BASE_URL="https://api.example.com"
TOKEN="your-api-token-here"

# Test authentication
echo "Testing authentication..."
curl -s -H "Authorization: Bearer $TOKEN" \
     "$BASE_URL/auth/verify" | jq '.'

# Create resource (POST)
echo "Creating user..."
USER_ID=$(curl -s -X POST \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
       "name": "John Doe",
       "email": "john@example.com",
       "role": "developer"
     }' \
     "$BASE_URL/users" | jq -r '.id')

echo "Created user ID: $USER_ID"

# Read resource (GET)
echo "Fetching user..."
curl -s -H "Authorization: Bearer $TOKEN" \
     "$BASE_URL/users/$USER_ID" | jq '.'

# Update resource (PUT)
echo "Updating user..."
curl -s -X PUT \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
       "name": "John Updated",
       "email": "john.updated@example.com"
     }' \
     "$BASE_URL/users/$USER_ID" | jq '.'

# Partial update (PATCH)
echo "Patching user..."
curl -s -X PATCH \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"role":"senior-developer"}' \
     "$BASE_URL/users/$USER_ID" | jq '.'

# Delete resource (DELETE)
echo "Deleting user..."
curl -s -X DELETE \
     -H "Authorization: Bearer $TOKEN" \
     "$BASE_URL/users/$USER_ID" \
     -w "\nHTTP Status: %{http_code}\n"
```

**API Test Script:**
```bash
#!/bin/bash

# API endpoint testing script

API_URL="https://api.example.com"
TOKEN="your-token"

# Function to test endpoint
test_endpoint() {
    local method=$1
    local endpoint=$2
    local data=$3
    
    echo "Testing: $method $endpoint"
    
    if [ -z "$data" ]; then
        response=$(curl -s -X "$method" \
                       -H "Authorization: Bearer $TOKEN" \
                       -w "\n%{http_code}" \
                       "$API_URL$endpoint")
    else
        response=$(curl -s -X "$method" \
                       -H "Authorization: Bearer $TOKEN" \
                       -H "Content-Type: application/json" \
                       -d "$data" \
                       -w "\n%{http_code}" \
                       "$API_URL$endpoint")
    fi
    
    status=$(echo "$response" | tail -n 1)
    body=$(echo "$response" | head -n -1)
    
    if [ "$status" -ge 200 ] && [ "$status" -lt 300 ]; then
        echo "✓ PASS (Status: $status)"
        echo "$body" | jq '.' 2>/dev/null || echo "$body"
    else
        echo "✗ FAIL (Status: $status)"
        echo "$body"
    fi
    
    echo "---"
}

# Run tests
test_endpoint "GET" "/users"
test_endpoint "POST" "/users" '{"name":"Test User","email":"test@example.com"}'
test_endpoint "GET" "/users/1"
test_endpoint "PUT" "/users/1" '{"name":"Updated User"}'
test_endpoint "DELETE" "/users/1"
```

---

### 9.2 Web Scraping

**Description:** Extract data from websites.

**Basic Web Scraping:**
```bash
# Download HTML page
curl -s https://example.com -o page.html

# Extract specific data with grep
curl -s https://example.com | grep -oP '(?<=<title>).*(?=</title>)'

# Get all links
curl -s https://example.com | grep -oP 'href="https?://[^"]*"'

# Download images
curl -s https://example.com | \
    grep -oP 'src="https?://[^"]*\.(jpg|png|gif)"' | \
    sed 's/src="//;s/"$//' | \
    xargs -I {} curl -O {}
```

**Advanced Scraping:**
```bash
# Scrape with cookies and headers
curl -s \
     -H "User-Agent: Mozilla/5.0" \
     -H "Accept: text/html" \
     -b "session=abc123" \
     https://example.com | \
     grep -oP '<h1>.*</h1>'

# Follow pagination
for page in {1..10}; do
    curl -s "https://example.com/articles?page=$page" \
         -H "User-Agent: Mozilla/5.0" \
         -o "page$page.html"
    sleep 2  # Be polite, don't hammer the server
done

# Extract JSON data from HTML
curl -s https://example.com/data | \
    grep -oP '(?<=var data = ).*(?=;)' | \
    jq '.'
```

---

### 9.3 File Downloads

**Description:** Download files efficiently.

**Single File Download:**
```bash
# Download and save with original name
curl -O https://example.com/file.zip

# Download with custom name
curl -o myfile.zip https://example.com/file.zip

# Download with progress bar
curl --progress-bar -O https://example.com/file.zip

# Resume interrupted download
curl -C - -O https://example.com/large-file.iso
```

**Batch Downloads:**
```bash
# Download multiple files
curl -O https://example.com/file1.pdf \
     -O https://example.com/file2.pdf \
     -O https://example.com/file3.pdf

# Download from list
while read url; do
    curl -O "$url"
done < urls.txt

# Download with pattern
curl -O "https://example.com/images/photo[1-100].jpg"
curl -O "https://example.com/files/doc[a-z].pdf"
```

**Monitored Download:**
```bash
#!/bin/bash

# Download with progress monitoring
URL="https://example.com/large-file.zip"
OUTPUT="download.zip"

curl -# -o "$OUTPUT" "$URL" 2>&1 | \
while IFS= read -r line; do
    echo "Progress: $line"
done

if [ $? -eq 0 ]; then
    echo "Download completed: $OUTPUT"
    ls -lh "$OUTPUT"
else
    echo "Download failed"
    exit 1
fi
```

---

### 9.4 FTP Operations

**Description:** Upload and download files via FTP/FTPS.

**FTP Download:**
```bash
# Download file
curl ftp://ftp.example.com/file.txt -o file.txt

# With authentication
curl ftp://user:password@ftp.example.com/file.txt -o file.txt

# List directory
curl ftp://ftp.example.com/directory/

# Download directory recursively (list files then download)
curl ftp://user:pass@ftp.example.com/dir/ | \
    grep -oP '[\w.-]+\.txt' | \
    xargs -I {} curl -O ftp://user:pass@ftp.example.com/dir/{}
```

**FTP Upload:**
```bash
# Upload file
curl -T upload.txt ftp://ftp.example.com/

# Upload with authentication
curl -T upload.txt ftp://user:password@ftp.example.com/remote-file.txt

# Upload multiple files
curl -T "{file1.txt,file2.txt}" ftp://user:pass@ftp.example.com/

# Upload with create directory
curl -T file.txt --ftp-create-dirs ftp://user:pass@ftp.example.com/newdir/file.txt
```

**FTPS (FTP over SSL):**
```bash
# Secure FTP
curl -k -T upload.txt ftps://ftp.example.com/

# With certificate verification
curl --cacert ca-cert.pem -T file.txt ftps://ftp.example.com/
```

---

### 9.5 Automation Scripts

**Description:** Automate tasks with curl in shell scripts.

**Health Check Script:**
```bash
#!/bin/bash

# Website health check script

URLS=(
    "https://example.com"
    "https://api.example.com/health"
    "https://app.example.com/status"
)

for url in "${URLS[@]}"; do
    echo "Checking: $url"
    
    status=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 "$url")
    
    if [ "$status" -eq 200 ]; then
        echo "✓ OK (Status: $status)"
    else
        echo "✗ FAILED (Status: $status)"
        # Send alert (email, Slack, etc.)
        curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
             -H "Content-Type: application/json" \
             -d "{\"text\":\"⚠️ $url is down! Status: $status\"}"
    fi
    
    echo "---"
done
```

**API Data Sync:**
```bash
#!/bin/bash

# Sync data from API to local files

API_BASE="https://api.example.com"
TOKEN="your-api-token"
DATA_DIR="./data"

mkdir -p "$DATA_DIR"

# Fetch updated records
echo "Fetching updates..."
UPDATES=$(curl -s \
    -H "Authorization: Bearer $TOKEN" \
    "$API_BASE/updates?since=$(date -d '1 hour ago' -Iseconds)")

# Process each update
echo "$UPDATES" | jq -c '.[]' | while read -r item; do
    id=$(echo "$item" | jq -r '.id')
    
    echo "Syncing item: $id"
    
    # Fetch full data
    curl -s \
        -H "Authorization: Bearer $TOKEN" \
        "$API_BASE/items/$id" \
        -o "$DATA_DIR/item-$id.json"
    
    # Validate JSON
    if jq empty "$DATA_DIR/item-$id.json" 2>/dev/null; then
        echo "✓ Synced: $id"
    else
        echo "✗ Failed: $id"
        rm -f "$DATA_DIR/item-$id.json"
    fi
done

echo "Sync complete"
```

**Backup Script:**
```bash
#!/bin/bash

# Automated backup using API

BACKUP_API="https://backup.example.com/api"
API_KEY="your-api-key"
BACKUP_FILE="backup-$(date +%Y%m%d-%H%M%S).tar.gz"

# Create backup archive
echo "Creating backup..."
tar -czf "$BACKUP_FILE" /path/to/data

# Upload to backup server
echo "Uploading backup..."
response=$(curl -s -w "\n%{http_code}" \
    -X POST \
    -H "X-API-Key: $API_KEY" \
    -F "file=@$BACKUP_FILE" \
    -F "metadata={\"timestamp\":\"$(date -Iseconds)\",\"size\":$(stat -f%z "$BACKUP_FILE")}" \
    "$BACKUP_API/upload")

status=$(echo "$response" | tail -n 1)
body=$(echo "$response" | head -n -1)

if [ "$status" -eq 200 ]; then
    echo "✓ Backup uploaded successfully"
    echo "$body" | jq '.'
    rm "$BACKUP_FILE"  # Remove local copy after successful upload
else
    echo "✗ Backup upload failed (Status: $status)"
    echo "$body"
fi
```

---

### 9.6 CI/CD Integration

**Description:** Integrate curl in CI/CD pipelines.

**GitHub Actions Example:**
```yaml
# .github/workflows/api-test.yml
name: API Tests

on: [push, pull_request]

jobs:
  test-api:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Test API endpoints
      env:
        API_TOKEN: ${{ secrets.API_TOKEN }}
      run: |
        # Test health endpoint
        status=$(curl -s -o /dev/null -w "%{http_code}" \
          https://api.example.com/health)
        
        if [ "$status" -ne 200 ]; then
          echo "Health check failed: $status"
          exit 1
        fi
        
        # Test authenticated endpoint
        response=$(curl -s -w "\n%{http_code}" \
          -H "Authorization: Bearer $API_TOKEN" \
          https://api.example.com/users)
        
        status=$(echo "$response" | tail -n 1)
        
        if [ "$status" -ne 200 ]; then
          echo "API test failed: $status"
          exit 1
        fi
        
        echo "All API tests passed"
```

**Deployment Webhook:**
```bash
#!/bin/bash

# Trigger deployment via webhook

DEPLOY_WEBHOOK="https://deploy.example.com/webhook"
SECRET="your-webhook-secret"
ENVIRONMENT="production"
VERSION="v1.2.3"

echo "Triggering deployment..."

response=$(curl -s -w "\n%{http_code}" \
    -X POST \
    -H "Content-Type: application/json" \
    -H "X-Webhook-Secret: $SECRET" \
    -d "{
      \"environment\": \"$ENVIRONMENT\",
      \"version\": \"$VERSION\",
      "timestamp": "$(date -Iseconds)",
      "triggered_by": "$USER"
    }" \
    "$DEPLOY_WEBHOOK")

status=$(echo "$response" | tail -n 1)
body=$(echo "$response" | head -n -1)

if [ "$status" -eq 200 ]; then
    echo "✓ Deployment triggered successfully"
    echo "$body" | jq '.'
else
    echo "✗ Deployment failed (Status: $status)"
    echo "$body"
    exit 1
fi
```

**Docker Build & Push:**
```bash
#!/bin/bash

# Build Docker image and push to registry

REGISTRY="registry.example.com"
IMAGE_NAME="myapp"
VERSION="$(git describe --tags)"
REGISTRY_TOKEN="your-registry-token"

echo "Building image: $IMAGE_NAME:$VERSION"
docker build -t "$IMAGE_NAME:$VERSION" .

echo "Tagging image"
docker tag "$IMAGE_NAME:$VERSION" "$REGISTRY/$IMAGE_NAME:$VERSION"
docker tag "$IMAGE_NAME:$VERSION" "$REGISTRY/$IMAGE_NAME:latest"

echo "Logging in to registry"
echo "$REGISTRY_TOKEN" | docker login "$REGISTRY" -u user --password-stdin

echo "Pushing image"
docker push "$REGISTRY/$IMAGE_NAME:$VERSION"
docker push "$REGISTRY/$IMAGE_NAME:latest"

# Notify via webhook
curl -X POST \
    -H "Content-Type: application/json" \
    -d "{
      \"action\": \"image_pushed\",
      \"image\": \"$REGISTRY/$IMAGE_NAME:$VERSION\",
      \"timestamp\": \"$(date -Iseconds)\"
    }" \
    https://notifications.example.com/webhook

echo "Deployment complete"
```

---

## Advanced cURL Techniques

### Custom Configuration File

**Description:** Create a configuration file for commonly used options.

**Create ~/.curlrc:**
```bash
# ~/.curlrc - curl configuration file

# Always follow redirects
--location

# Show errors
--show-error

# Retry on transient errors
--retry 3
--retry-delay 2

# Connection timeout
--connect-timeout 10

# Maximum time
--max-time 30

# Custom user agent
user-agent = "MyCurlClient/1.0"

# Compressed responses
--compressed

# Use HTTP/2
--http2
```

**Use config file:**
```bash
# Uses ~/.curlrc by default
curl https://example.com

# Ignore .curlrc
curl -q https://example.com

# Use custom config file
curl -K myconfig.txt https://example.com
```

---

### cURL with JSON Processing

**Complete API Workflow:**
```bash
#!/bin/bash

# Complete API workflow with JSON processing

API_URL="https://api.example.com"
TOKEN="your-api-token"

# Function to make API request
api_request() {
    local method="$1"
    local endpoint="$2"
    local data="$3"
    
    if [ -z "$data" ]; then
        curl -s -X "$method" \
            -H "Authorization: Bearer $TOKEN" \
            -H "Accept: application/json" \
            "$API_URL$endpoint"
    else
        curl -s -X "$method" \
            -H "Authorization: Bearer $TOKEN" \
            -H "Content-Type: application/json" \
            -H "Accept: application/json" \
            -d "$data" \
            "$API_URL$endpoint"
    fi
}

# Get all users
echo "Fetching users..."
users=$(api_request "GET" "/users")
user_count=$(echo "$users" | jq 'length')
echo "Total users: $user_count"

# Create new user
echo "Creating user..."
new_user=$(api_request "POST" "/users" '{
    "name": "Jane Doe",
    "email": "jane@example.com",
    "role": "developer"
}')

user_id=$(echo "$new_user" | jq -r '.id')
user_name=$(echo "$new_user" | jq -r '.name')
echo "Created: $user_name (ID: $user_id)"

# Get user details
echo "Fetching user details..."
user_details=$(api_request "GET" "/users/$user_id")
echo "$user_details" | jq '.'

# Update user
echo "Updating user..."
updated_user=$(api_request "PUT" "/users/$user_id" '{
    "name": "Jane Smith",
    "email": "jane.smith@example.com"
}')

echo "Updated: $(echo "$updated_user" | jq -r '.name')"

# List all users and save to file
echo "Saving all users to file..."
api_request "GET" "/users" | jq '.' > users.json
echo "Saved to users.json"

# Filter and process
echo "Active developers:"
jq '[.[] | select(.role == "developer" and .active == true)]' users.json
```

---

### Error Handling Best Practices

**Robust Error Handling:**
```bash
#!/bin/bash

# Function with comprehensive error handling
api_call_safe() {
    local url="$1"
    local output_file="${2:-/dev/null}"
    local max_retries=3
    local retry_delay=5
    
    for ((i=1; i<=max_retries; i++)); do
        echo "Attempt $i/$max_retries..."
        
        # Make request and capture HTTP code
        http_code=$(curl -s -w "%{http_code}" \
            --connect-timeout 10 \
            --max-time 30 \
            -o "$output_file" \
            "$url")
        
        curl_exit_code=$?
        
        # Check curl exit code
        if [ $curl_exit_code -ne 0 ]; then
            echo "✗ curl failed with exit code: $curl_exit_code"
            
            case $curl_exit_code in
                6) echo "Could not resolve host" ;;
                7) echo "Failed to connect" ;;
                28) echo "Timeout" ;;
                35) echo "SSL connection error" ;;
                *) echo "Unknown error" ;;
            esac
            
            if [ $i -lt $max_retries ]; then
                echo "Retrying in $retry_delay seconds..."
                sleep $retry_delay
                continue
            else
                return $curl_exit_code
            fi
        fi
        
        # Check HTTP status code
        if [ "$http_code" -ge 200 ] && [ "$http_code" -lt 300 ]; then
            echo "✓ Success (HTTP $http_code)"
            return 0
        elif [ "$http_code" -ge 400 ] && [ "$http_code" -lt 500 ]; then
            echo "✗ Client error (HTTP $http_code)"
            return 1
        elif [ "$http_code" -ge 500 ]; then
            echo "✗ Server error (HTTP $http_code)"
            
            if [ $i -lt $max_retries ]; then
                echo "Retrying in $retry_delay seconds..."
                sleep $retry_delay
                continue
            else
                return 1
            fi
        fi
    done
    
    return 1
}

# Usage
if api_call_safe "https://api.example.com/data" "output.json"; then
    echo "Data saved to output.json"
    # Process data
    jq '.' output.json
else
    echo "Failed to fetch data"
    exit 1
fi
```

---

## cURL Cheat Sheet

### Quick Reference

**Basic Requests:**
```bash
# GET
curl https://api.example.com/users

# POST JSON
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' URL

# PUT
curl -X PUT -d "data" URL

# DELETE
curl -X DELETE URL
```

**Headers & Auth:**
```bash
# Custom header
curl -H "Key: Value" URL

# Basic auth
curl -u user:pass URL

# Bearer token
curl -H "Authorization: Bearer TOKEN" URL
```

**Output:**
```bash
# Save to file
curl -o file.txt URL
curl -O URL  # Use remote filename

# Include headers
curl -i URL

# Only headers
curl -I URL

# Silent
curl -s URL

# Verbose
curl -v URL
```

**Options:**
```bash
# Follow redirects
curl -L URL

# Timeout
curl --max-time 30 URL

# Retry
curl --retry 3 URL

# Cookie
curl -b cookies.txt -c cookies.txt URL

# User agent
curl -A "MyAgent" URL

# Compressed
curl --compressed URL
```

**Upload:**
```bash
# Form data
curl -F "field=value" URL

# File upload
curl -F "file=@path/to/file" URL

# Binary upload
curl -T file.zip URL
```

---

## Common Use Cases

### 1. Download File with Progress
```bash
curl -# -o file.zip https://example.com/file.zip
```

### 2. POST JSON Data
```bash
curl -X POST https://api.example.com/data \
  -H "Content-Type: application/json" \
  -d '{"name":"John","age":30}'
```

### 3. Upload File
```bash
curl -F "file=@document.pdf" https://api.example.com/upload
```

### 4. API Authentication
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" https://api.example.com/secure
```

### 5. Follow Redirects
```bash
curl -L https://short.url/abc123
```

### 6. Download Multiple Files
```bash
curl -O https://example.com/file[1-10].txt
```

### 7. Resume Download
```bash
curl -C - -O https://example.com/large-file.iso
```

### 8. Test API Endpoint
```bash
curl -X GET https://api.example.com/health -w "\nStatus: %{http_code}\n"
```

### 9. Save Response Headers
```bash
curl -D headers.txt -o body.html https://example.com
```

### 10. Silent Download
```bash
curl -s -o data.json https://api.example.com/data
```

---

## Troubleshooting Guide

### Common Errors and Solutions

**1. SSL Certificate Error**
```bash
# Error: SSL certificate problem
# Solution: Skip verification (dev only) or provide CA cert
curl -k https://self-signed.example.com
curl --cacert ca-bundle.crt https://example.com
```

**2. Connection Timeout**
```bash
# Error: Connection timed out
# Solution: Increase timeout or check network
curl --connect-timeout 30 https://slow-server.com
```

**3. Could Not Resolve Host**
```bash
# Error: Could not resolve host
# Solution: Check DNS or specify IP
curl --resolve example.com:443:93.184.216.34 https://example.com
```

**4. HTTP 401 Unauthorized**
```bash
# Error: 401 Unauthorized
# Solution: Add authentication
curl -u user:pass https://api.example.com
curl -H "Authorization: Bearer TOKEN" https://api.example.com
```

**5. HTTP 403 Forbidden**
```bash
# Error: 403 Forbidden
# Solution: Check permissions, add headers, or change User-Agent
curl -H "User-Agent: Mozilla/5.0" https://example.com
```

**6. HTTP 404 Not Found**
```bash
# Error: 404 Not Found
# Solution: Verify URL
curl -I https://example.com/correct-path
```

**7. Too Many Redirects**
```bash
# Error: Maximum redirects followed
# Solution: Increase limit or check for redirect loops
curl -L --max-redirs 10 https://example.com
```

**8. Empty Response**
```bash
# Error: Empty reply from server
# Solution: Check if server is running, try verbose mode
curl -v https://example.com
```

---

## Best Practices

### 1. Always Handle Errors
```bash
# Check exit code
if ! curl -f https://api.example.com/data; then
    echo "Request failed"
    exit 1
fi
```

### 2. Use Timeouts
```bash
# Prevent hanging requests
curl --connect-timeout 10 --max-time 30 URL
```

### 3. Implement Retries
```bash
# Retry on transient failures
curl --retry 3 --retry-delay 5 URL
```

### 4. Be Respectful (Rate Limiting)
```bash
# Add delays between requests
for url in $(cat urls.txt); do
    curl "$url"
    sleep 1
done
```

### 5. Use Configuration Files
```bash
# Store common options in ~/.curlrc
echo "--retry 3" >> ~/.curlrc
echo "--connect-timeout 10" >> ~/.curlrc
```

### 6. Secure Sensitive Data
```bash
# Use environment variables for tokens
export API_TOKEN="your-secret-token"
curl -H "Authorization: Bearer $API_TOKEN" URL

# Don't log sensitive data
curl -s -o /dev/null -w "%{http_code}" URL
```

### 7. Validate Responses
```bash
# Validate JSON
curl -s URL | jq empty || echo "Invalid JSON"

# Check HTTP status
status=$(curl -s -o /dev/null -w "%{http_code}" URL)
[ "$status" = "200" ] || echo "Request failed: $status"
```

### 8. Use Verbose Mode for Debugging
```bash
# Debug with verbose output
curl -v https://api.example.com 2>&1 | tee debug.log
```

### 9. Monitor Performance
```bash
# Track timing
curl -w "Time: %{time_total}s\n" -o /dev/null -s URL
```

### 10. Document Your Scripts
```bash
#!/bin/bash
# Purpose: Sync data from API
# Usage: ./sync.sh
# Dependencies: curl, jq

# Well-documented curl commands
curl -X GET \
    -H "Authorization: Bearer $TOKEN" \
    "$API_URL/data" \
    | jq '.' > output.json
```

---

## Performance Tips

### 1. Enable HTTP/2
```bash
curl --http2 https://example.com
```

### 2. Use Connection Reuse
```bash
# HTTP/1.1 keep-alive is default
curl -H "Connection: keep-alive" https://api.example.com
```

### 3. Enable Compression
```bash
curl --compressed https://api.example.com
```

### 4. Parallel Downloads
```bash
cat urls.txt | xargs -P 5 -n 1 curl -O
```

### 5. Limit Bandwidth
```bash
# Prevent saturating connection
curl --limit-rate 1M -O https://example.com/file.zip
```

### 6. Resume Interrupted Downloads
```bash
curl -C - -O https://example.com/large-file.iso
```

### 7. Use Local DNS Cache
```bash
# Reduce DNS lookup time
curl --dns-servers 8.8.8.8 URL
```

---

## Security Best Practices

### 1. Always Use HTTPS
```bash
# Prefer HTTPS over HTTP
curl https://api.example.com
```

### 2. Verify SSL Certificates
```bash
# Don't use -k in production
curl https://example.com  # Good
curl -k https://example.com  # Bad (insecure)
```

### 3. Store Credentials Securely
```bash
# Use environment variables or credential files
export API_KEY=$(cat ~/.api_key)
curl -H "X-API-Key: $API_KEY" URL
```

### 4. Use Token Authentication
```bash
# Prefer tokens over basic auth
curl -H "Authorization: Bearer $TOKEN" URL
```

### 5. Validate Input
```bash
# Sanitize user input before using in URLs
query=$(echo "$USER_INPUT" | jq -sRr @uri)
curl "https://api.example.com/search?q=$query"
```

### 6. Limit Exposure
```bash
# Don't echo sensitive data
curl -s -H "Authorization: Bearer $TOKEN" URL > /dev/null
```

---

## Resources

### Official Documentation
- [cURL Official Website](https://curl.se/)
- [cURL Man Page](https://curl.se/docs/manpage.html)
- [Everything cURL Book](https://everything.curl.dev/)

### Online Tools
- [curl.trillworks.com](https://curl.trillworks.com/) - Convert curl to code
- [reqbin.com](https://reqbin.com/curl) - Online curl client
- [curlconverter.com](https://curlconverter.com/) - Convert curl to various languages

### Related Tools
- **httpie** - User-friendly HTTP client
- **wget** - File download utility
- **aria2** - Multi-protocol download utility
- **jq** - JSON processor
- **xmllint** - XML processor

### Testing APIs
- [JSONPlaceholder](https://jsonplaceholder.typicode.com/) - Fake API for testing
- [httpbin.org](https://httpbin.org/) - HTTP request & response service
- [ReqRes](https://reqres.in/) - Test API with realistic data

---

## Conclusion

Congratulations! You've completed the comprehensive cURL mastery tutorial. You now have the knowledge to:

✅ **Make HTTP requests** with all methods (GET, POST, PUT, PATCH, DELETE)
✅ **Handle authentication** (Basic, Bearer, OAuth, API Keys)
✅ **Work with various data formats** (JSON, XML, Form Data, Multipart)
✅ **Manage responses** (headers, status codes, output)
✅ **Use advanced options** (timeouts, retries, redirects, cookies)
✅ **Implement security** (HTTPS, SSL/TLS, certificates)
✅ **Apply advanced techniques** (parallel requests, HTTP/2, proxies)
✅ **Solve real-world problems** (API testing, web scraping, automation)
✅ **Integrate with CI/CD** (GitHub Actions, deployment webhooks)
✅ **Write production-ready scripts** with error handling

### Next Steps:

1. **Practice Daily** - Use curl for everyday tasks
2. **Build Projects** - Create automation scripts
3. **Contribute** - Share your curl scripts with the community
4. **Explore** - Learn related tools (httpie, wget, aria2)
5. **Stay Updated** - Follow curl releases for new features

### Key Takeaways:

- cURL is a powerful, versatile tool for data transfer
- Always handle errors and implement retries
- Security should be a priority (HTTPS, certificate verification)
- Automation saves time and reduces errors
- Proper documentation makes scripts maintainable

**Happy curling! 🚀**

---

**End of cURL Complete Mastery Tutorial**

*Version: 1.0*  
*Last Updated: December 2025*  
*Tutorial Size: Complete Guide*  
*Skill Level: Beginner to Advanced*  
*License: MIT*

---

## Appendix

### cURL Exit Codes Reference

| Code | Meaning |
|------|----------|
| 0 | Success |
| 1 | Unsupported protocol |
| 3 | URL malformed |
| 5 | Couldn't resolve proxy |
| 6 | Couldn't resolve host |
| 7 | Failed to connect to host |
| 18 | Partial file transfer |
| 22 | HTTP error (with -f flag) |
| 23 | Write error |
| 26 | Read error |
| 27 | Out of memory |
| 28 | Operation timeout |
| 35 | SSL connect error |
| 51 | Server certificate verification failed |
| 52 | Empty response from server |
| 55 | Failed sending network data |
| 56 | Failed receiving network data |

### HTTP Status Codes Reference

**1xx Informational**
- 100 Continue
- 101 Switching Protocols

**2xx Success**
- 200 OK
- 201 Created
- 202 Accepted
- 204 No Content

**3xx Redirection**
- 301 Moved Permanently
- 302 Found
- 304 Not Modified
- 307 Temporary Redirect

**4xx Client Errors**
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 429 Too Many Requests

**5xx Server Errors**
- 500 Internal Server Error
- 502 Bad Gateway
- 503 Service Unavailable
- 504 Gateway Timeout

### Glossary

- **API**: Application Programming Interface
- **Bearer Token**: Authentication token passed in Authorization header
- **CA**: Certificate Authority
- **CRUD**: Create, Read, Update, Delete
- **DNS**: Domain Name System
- **FTP**: File Transfer Protocol
- **HTTP**: Hypertext Transfer Protocol
- **HTTPS**: HTTP Secure (over SSL/TLS)
- **JSON**: JavaScript Object Notation
- **JWT**: JSON Web Token
- **OAuth**: Open Authorization framework
- **REST**: Representational State Transfer
- **SSL**: Secure Sockets Layer
- **TLS**: Transport Layer Security
- **URL**: Uniform Resource Locator
- **XML**: Extensible Markup Language

---

*Thank you for completing this tutorial! Keep practicing and exploring the amazing world of cURL!*