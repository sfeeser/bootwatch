Below is a README.md file for the Go program, providing an overview, setup instructions, usage examples, and other relevant details to help users understand and run the domain status service.
markdown
# Bootwatch

A lightweight Go-based HTTP server that tracks playbook status updates by comain. It listens on port 58423 and provides RESTful endpoints to store and retrieve domain status information sent from specific tasks in our ansible playbooks. The service stores data in memory, with automatic cleanup of updates older than 48 hours, making it suitable for low-usage scenarios.

## Features

- **POST /update**: Accepts JSON payloads with `domain` and `status`, storing updates with timestamps.
- **GET /**: Returns all domain statuses in YAML format, sorted by domain (alphabetical) and timestamp (descending).
- **GET /{domain}**: Returns status updates for a specific domain in YAML, or "Does Not Exist" if the domain is not found.
- **Automatic Cleanup**: Removes updates older than 48 hours, running every hour to manage memory usage.
- **Thread-Safe**: Uses mutexes to handle concurrent requests safely.
- **Simple Validation**: Ensures valid domain names and non-empty statuses.

## Requirements

- Go 1.18 or later
- External package: `gopkg.in/yaml.v3`

## Installation

1. **Clone the Repository** (or copy the code to a directory):

   ``` bash
   git clone git@github.com:sfeeser/bootwatch.git
   cd bootwatch
   ```

2. Initialize a Go Module (if not already initialized):

    ```
    go mod init domain-status-service
    ```
    
4. Install Dependencies:

    ```
    go get gopkg.in/yaml.v3
    ```

6. Go mod

    ```
    go mod tidy
    ```

8. Build and Run:

    ```
    go run main.go
    ```

The server will start on localhost:58423.
Usage
Endpoints
- POST /update  
  Stores a domain status update.
- Request:

   ```bash
   curl -X POST http://localhost:58423/update -H "Content-Type: application/json" -d '{"domain":"example.com","status":"up"}'
   ```
      
- Response (JSON):

   ```json
   {"message":"Update stored"}
   ```
   
Errors:
- 400: Invalid JSON or input (e.g., empty status, invalid domain).
- 405: Non-POST requests.

GET /
- Returns all domain statuses in YAML format.
- Request:

    ```bash
    curl http://localhost:58423/
    ```
    
- Response (YAML):
    ```yaml
    example.com:
      - status: up
        timestamp: 2025-04-22T12:00:00Z
    another.com:
      - status: down
        timestamp: 2025-04-22T11:00:00Z
    ```
    
Errors:
- 405: Non-GET requests.
- 500: Internal server error (rare, e.g., YAML marshaling failure).

GET /{domain}
- Returns status updates for a specific domain.
- Request:
    ```bash
    curl http://localhost:58423/example.com
    ```
    
- Response (YAML):
       
    ```yaml
    - status: up
      timestamp: 2025-04-22T12:00:00Z
    ```

Not Found:
text
Does Not Exist

Errors:
- 400: Empty domain.
- 404: Domain not found.
- 405: Non-GET requests.
- 500: Internal server error.

Data Management
- Storage: In-memory, using a thread-safe map.
- Cleanup: Every hour, updates older than 48 hours are removed. Check server logs for "Cleanup completed" messages.
- Persistence: Data is not persisted; it’s cleared on restart.
- Project Structure
- main.go: The main program implementing the HTTP server, storage, and cleanup logic.

Configuration
- Port: Hardcoded to 58423.
- Cleanup Interval: Runs every 1 hour (configurable in main.go by modifying time.NewTicker(1 * time.Hour)).
- Retention Period: 48 hours (configurable in Storage.Cleanup by modifying time.Hour * 48).

Notes
- Domain Validation: Allows alphanumeric characters, dots, and hyphens. Stricter validation (e.g., DNS-compliant domains) can be added if needed.
- Performance: Designed for low usage. The cleanup scans all data, which is sufficient for small datasets.
- Security: Limits request body size to 1MB. Consider adding rate limiting or authentication for production use.
- Logging: Errors and cleanup events are logged to stderr.

Testing
- To test the service:
- Start the server:

    ```bash
    go run main.go
    Post updates:
    ```
    
    ```bash
    curl -X POST http://localhost:58423/update -H "Content-Type: application/json" -d '{"domain":"example.com","status":"up"}'
    curl -X POST http://localhost:58423/update -H "Content-Type: application/json" -d '{"domain":"another.com","status":"down"}'
    ```
- Retrieve data:
    ``` bash
    curl http://localhost:58423/
    curl http://localhost:58423/example.com
    Test non-existent domain:
    ```
    
    ```bash
    curl http://localhost:58423/nonexistent.com
    ```

Monitor cleanup: Check logs after an hour or modify main.go to trigger storage.Cleanup() manually.
Future Improvements
Add persistent storage (e.g., SQLite) for data retention across restarts.
Implement stricter domain validation (e.g., regex for DNS names).
Add rate limiting or authentication for production use.
Include unit and integration tests for reliability.
Expose metrics (e.g., via /metrics) for monitoring.
License
MIT License (or specify your preferred license).
Contact
For issues or suggestions, please open an issue on the repository or contact the maintainer.

### Explanation
- **Purpose**: The README provides a clear, concise guide for users to set up, run, and use the service.
- **Structure**:
  - **Overview**: Describes the service and its features.
  - **Setup**: Lists requirements and installation steps.
  - **Usage**: Details each endpoint with examples, including requests, responses, and errors.
  - **Configuration**: Notes on port, cleanup, and retention settings.
  - **Notes**: Covers validation, performance, and security considerations.
  - **Testing**: Guides users on testing the service.
  - **Future Improvements**: Suggests potential enhancements.
- **Clarity**: Written for both developers and non-technical users, with curl examples for easy testing.
- **Flexibility**: Includes placeholders (e.g., repository URL, license) that you can customize.

### Usage
1. Save this as `README.md` in the same directory as `main.go`.
2. If you’re using a version control system like Git, update placeholders (e.g., `<repository-url>`) with your repo’s details.
3. Customize the license or contact sections as needed.

Let me know if you need help customizing the README, adding more details, or creating additional documentation (e.g., a contributing guide)!
