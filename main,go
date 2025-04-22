package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "sort"
    "strings"
    "sync"
    "time"

    "gopkg.in/yaml.v3"
)

// StatusUpdate represents a single status update with timestamp
type StatusUpdate struct {
    Status    string `yaml:"status"`
    Timestamp string `yaml:"timestamp"`
}

// Storage manages the in-memory domain status data
type Storage struct {
    mutex sync.RWMutex
    data  map[string][]StatusUpdate
}

// UpdateRequest represents the expected JSON input for POST /update
type UpdateRequest struct {
    Domain string `json:"domain"`
    Status string `json:"status"`
}

// NewStorage initializes a new Storage instance
func NewStorage() *Storage {
    return &Storage{
        data: make(map[string][]StatusUpdate),
    }
}

// AddUpdate adds a new status update for a domain
func (s *Storage) AddUpdate(domain, status string) {
    s.mutex.Lock()
    defer s.mutex.Unlock()

    update := StatusUpdate{
        Status:    status,
        Timestamp: time.Now().UTC().Format(time.RFC3339),
    }

    s.data[domain] = append(s.data[domain], update)
}

// GetAll returns all domain statuses, sorted by domain and timestamp
func (s *Storage) GetAll() map[string][]StatusUpdate {
    s.mutex.RLock()
    defer s.mutex.RUnlock()

    // Create a copy to avoid modifying original data
    result := make(map[string][]StatusUpdate)
    for domain, updates := range s.data {
        // Copy updates
        updatesCopy := make([]StatusUpdate, len(updates))
        copy(updatesCopy, updates)
        // Sort by timestamp (descending)
        sort.Slice(updatesCopy, func(i, j int) bool {
            return updatesCopy[i].Timestamp > updatesCopy[j].Timestamp
        })
        result[domain] = updatesCopy
    }

    return result
}

// GetDomain returns status updates for a specific domain
func (s *Storage) GetDomain(domain string) ([]StatusUpdate, bool) {
    s.mutex.RLock()
    defer s.mutex.RUnlock()

    updates, exists := s.data[domain]
    if !exists {
        return nil, false
    }

    // Copy and sort by timestamp (descending)
    updatesCopy := make([]StatusUpdate, len(updates))
    copy(updatesCopy, updates)
    sort.Slice(updatesCopy, func(i, j int) bool {
        return updatesCopy[i].Timestamp > updatesCopy[j].Timestamp
    })

    return updatesCopy, true
}

// Cleanup removes updates older than 48 hours
func (s *Storage) Cleanup() {
    s.mutex.Lock()
    defer s.mutex.Unlock()

    cutoff := time.Now().UTC().Add(-48 * time.Hour)
    for domain, updates := range s.data {
        // Filter updates newer than cutoff
        var newUpdates []StatusUpdate
        for _, update := range updates {
            t, err := time.Parse(time.RFC3339, update.Timestamp)
            if err != nil {
                log.Printf("Error parsing timestamp %s: %v", update.Timestamp, err)
                continue
            }
            if t.After(cutoff) {
                newUpdates = append(newUpdates, update)
            }
        }
        // Update or delete domain entry
        if len(newUpdates) > 0 {
            s.data[domain] = newUpdates
        } else {
            delete(s.data, domain)
        }
    }
    log.Println("Cleanup completed")
}

// validateDomain performs basic domain validation
func validateDomain(domain string) bool {
    domain = strings.TrimSpace(domain)
    if domain == "" || len(domain) > 255 {
        return false
    }
    // Basic validation: allow alphanumeric, dots, and hyphens
    for _, char := range domain {
        if !((char >= 'a' && char <= 'z') ||
            (char >= 'A' && char <= 'Z') ||
            (char >= '0' && char <= '9') ||
            char == '.' || char == '-') {
            return false
        }
    }
    return true
}

// handleUpdate handles POST /update requests
func handleUpdate(w http.ResponseWriter, r *http.Request, storage *Storage) {
    if r.Method != http.MethodPost {
        http.Error(w, `{"error": "Method not allowed"}`, http.StatusMethodNotAllowed)
        return
    }

    // Limit request body size to 1MB
    r.Body = http.MaxBytesReader(w, r.Body, 1<<20)

    var req UpdateRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, `{"error": "Invalid JSON"}`, http.StatusBadRequest)
        return
    }

    // Validate input
    if !validateDomain(req.Domain) || strings.TrimSpace(req.Status) == "" {
        http.Error(w, `{"error": "Invalid input"}`, http.StatusBadRequest)
        return
    }

    // Store update
    storage.AddUpdate(req.Domain, req.Status)

    // Respond with success
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    fmt.Fprintf(w, `{"message": "Update stored"}`)
}

// handleListAll handles GET / requests
func handleListAll(w http.ResponseWriter, r *http.Request, storage *Storage) {
    if r.Method != http.MethodGet {
        http.Error(w, `{"error": "Method not allowed"}`, http.StatusMethodNotAllowed)
        return
    }

    data := storage.GetAll()

    // Marshal to YAML
    yamlData, err := yaml.Marshal(data)
    if err != nil {
        log.Printf("Error marshaling YAML: %v", err)
        http.Error(w, "Internal server error", http.StatusInternalServerError)
        return
    }

    // Set response headers and write YAML
    w.Header().Set("Content-Type", "application/x-yaml")
    w.WriteHeader(http.StatusOK)
    w.Write(yamlData)
}

// handleDomainStatus handles GET /{domain} requests
func handleDomainStatus(w http.ResponseWriter, r *http.Request, storage *Storage) {
    if r.Method != http.MethodGet {
        http.Error(w, `{"error": "Method not allowed"}`, http.StatusMethodNotAllowed)
        return
    }

    // Extract domain from URL path
    domain := strings.TrimPrefix(r.URL.Path, "/")
    if domain == "" {
        http.Error(w, `{"error": "Domain required"}`, http.StatusBadRequest)
        return
    }

    updates, exists := storage.GetDomain(domain)
    if !exists {
        w.WriteHeader(http.StatusNotFound)
        fmt.Fprintf(w, "Does Not Exist")
        return
    }

    // Marshal to YAML
    yamlData, err := yaml.Marshal(updates)
    if err != nil {
        log.Printf("Error marshaling YAML: %v", err)
        http.Error(w, "Internal server error", http.StatusInternalServerError)
        return
    }

    // Set response headers and write YAML
    w.Header().Set("Content-Type", "application/x-yaml")
    w.WriteHeader(http.StatusOK)
    w.Write(yamlData)
}

func main() {
    // Initialize storage
    storage := NewStorage()

    // Start cleanup routine
    go func() {
        ticker := time.NewTicker(1 * time.Hour)
        defer ticker.Stop()
        for range ticker.C {
            storage.Cleanup()
        }
    }()

    // Set up HTTP server
    mux := http.NewServeMux()
    mux.HandleFunc("/update", func(w http.ResponseWriter, r *http.Request) {
        handleUpdate(w, r, storage)
    })
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        if r.URL.Path == "/" {
            handleListAll(w, r, storage)
        } else {
            handleDomainStatus(w, r, storage)
        }
    })

    server := &http.Server{
        Addr:    ":58423",
        Handler: mux,
    }

    // Start server
    log.Printf("Starting server on :58423")
    if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
        log.Fatalf("Server error: %v", err)
    }
}
