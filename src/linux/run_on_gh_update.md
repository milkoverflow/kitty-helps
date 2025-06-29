# github webhooks

go to `https://github.com/{USER}/{REPO}/settings/hooks` and enable webhook

then host a server listening for push events, eg node:

```javascript
import { createServer } from 'http';
import { exec } from 'child_process';

const server = createServer((req, res) => {
  if (req.headers['x-hub-signature-256'] === undefined) {
    res.writeHead(405, { 'Content-Type': 'text/plain' });
    res.end('method not allowed');
  }
  if (req.method === 'POST') {
    req.on('data', _ => { });
    req.on('end', () => {
      console.log('received push');
      exec('git pull', (error) => {
        if (error) {
          console.error('error: ', error);
        } else {
          console.log('pulled');
        }
      });
      res.writeHead(200);
      res.end('ok');
    });
  } else {
    res.writeHead(405, { 'Content-Type': 'text/plain' });
    res.end('method not allowed');
  }
});

server.listen(4567, () => {
  console.log('listening on 4567 for push events');
});
```

eg go:
```go
package main

import (
	"crypto/hmac"
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
	"os/exec"
)

const (
	secretEnv = "WEBHOOK_SECRET"
	port      = "4567"
)
var repoDirs = map[string]string{
	"repo1": "/home/user/repo1",
}
type GitHubPayload struct {
	Repository struct {
		Name string `json:"name"`
	} `json:"repository"`
}


func main() {
	http.HandleFunc("/webhook", handleWebhook)
	fmt.Printf("Listening on :%s...\n", port)
	err := http.ListenAndServe(":"+port, nil)
	if err != nil {
		fmt.Println("Server failed:", err)
	}
}

func handleWebhook(w http.ResponseWriter, r *http.Request) {
	secret := os.Getenv(secretEnv)
	if secret == "" {
		http.Error(w, "Secret not configured", http.StatusInternalServerError)
		return
	}
	fmt.Println("Received request")

	body, err := io.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Could not read body", http.StatusBadRequest)
		return
	}
	fmt.Println("Read payload")

	signature := r.Header.Get("X-Hub-Signature-256")
	if !validateSignature(secret, body, signature) {
		http.Error(w, "Invalid signature", http.StatusUnauthorized)
		return
	}
	fmt.Println("Validated signature")
		var payload GitHubPayload
	if err := json.Unmarshal(body, &payload); err != nil {
		http.Error(w, "Invalid JSON payload", http.StatusBadRequest)
		return
	}

	repoDir := repoDirs[payload.Repository.Name]

	cmd := exec.Command("git", "-C", repoDir, "pull", "origin", "main")
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	err = cmd.Run()
	if err != nil {
		http.Error(w, "Git pull failed", http.StatusInternalServerError)
		return
	}

	w.WriteHeader(http.StatusOK)
	w.Write([]byte("Success"))
}

func validateSignature(secret string, body []byte, signature string) bool {
	const prefix = "sha256="

	if len(signature) <= len(prefix) || signature[:len(prefix)] != prefix {
		return false
	}

	expectedMAC := computeHMAC(body, []byte(secret))
	sigBytes, err := hex.DecodeString(signature[len(prefix):])
	if err != nil {
		return false
	}

	return hmac.Equal(sigBytes, expectedMAC)
}

func computeHMAC(message, key []byte) []byte {
	mac := hmac.New(sha256.New, key)
	mac.Write(message)
	return mac.Sum(nil)
}
```
