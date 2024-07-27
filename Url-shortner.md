Got it! If you want a simple URL shortener without using DynamoDB, you can store the mappings in memory. This approach is suitable for testing and local development but not for production due to the lack of persistence.

Here’s how you can implement a simple in-memory URL shortener using AWS Lambda and Go:

### Step 1: Write the Lambda Function

Create a `main.go` file:

```go
package main

import (
        "context"
        "encoding/json"
        "fmt"
        "math/rand"
        "net/http"
        "sync"
        "time"

        "github.com/aws/aws-lambda-go/events"
        "github.com/aws/aws-lambda-go/lambda"
)

type Request struct {
        URL string `json:"url"`
}

type Response struct {
        ShortURL string `json:"short_url"`
}

type URLMapping struct {
        ShortCode string `json:"short_code"`
        URL       string `json:"url"`
}

var (
        mappings = make(map[string]string)
        mutex    = &sync.Mutex{}
)

func init() {
        rand.Seed(time.Now().UnixNano())
}

func handler(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
        switch request.HTTPMethod {
        case "POST":
                return handleShortenURL(request)
        case "GET":
                return handleRedirect(request)
        default:
                return events.APIGatewayProxyResponse{
                        StatusCode: http.StatusMethodNotAllowed,
                        Body:       "Method Not Allowed",
                }, nil
        }
}

func handleShortenURL(request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
        var req Request
        err := json.Unmarshal([]byte(request.Body), &req)
        if err != nil || req.URL == "" {
                return events.APIGatewayProxyResponse{
                        StatusCode: http.StatusBadRequest,
                        Body:       "Invalid request",
                }, nil
        }

        shortCode := generateShortCode()

        mutex.Lock()
        mappings[shortCode] = req.URL
        mutex.Unlock()

        shortURL := fmt.Sprintf("https://%s/%s", request.Headers["Host"], shortCode)
        response := Response{ShortURL: shortURL}
        responseBody, _ := json.Marshal(response)

        return events.APIGatewayProxyResponse{
                StatusCode: http.StatusOK,
                Body:       string(responseBody),
        }, nil
}

func handleRedirect(request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
        shortCode := request.PathParameters["shortCode"]

        mutex.Lock()
        url, exists := mappings[shortCode]
        mutex.Unlock()

        if !exists {
                return events.APIGatewayProxyResponse{
                        StatusCode: http.StatusNotFound,
                        Body:       "URL not found",
                }, nil
        }

        return events.APIGatewayProxyResponse{
                StatusCode: http.StatusMovedPermanently,
                Headers: map[string]string{
                        "Location": url,
                },
        }, nil
}

func generateShortCode() string {
        const charset = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
        code := make([]byte, 6)
        for i := range code {
                code[i] = charset[rand.Intn(len(charset))]
        }
        return string(code)
}

func main() {
        lambda.Start(handler)
}
```

### Step 2: Build and Deploy the Lambda Function

1. **Build the Go executable for Linux:**

   ```sh
   GOOS=linux GOARCH=amd64 go build -o main main.go
   ```

2. **Create a deployment package:**

   ```sh
   zip main.zip main
   ```

3. **Deploy the Lambda function:**

   - Go to the AWS Lambda console.
   - Create a new Lambda function.
   - Upload the `main.zip` file.
   - Set the handler to `main`.

### Step 3: Set Up API Gateway

1. **Create a new API in API Gateway.**
2. **Create two routes:**
   - POST `/shorten`
   - GET `/{shortCode}`
3. **Integrate these routes with the Lambda function.**

### Step 4: Test the Function

1. **Shorten a URL:**

   ```sh
   curl -X POST -H "Content-Type: application/json" -d '{"url":"https://www.example.com"}' https://your-api-id.execute-api.your-region.amazonaws.com/shorten
   ```

2. **Redirect using the shortened URL:**

   Open the shortened URL in your browser or use `curl`:

   ```sh
   curl -L https://your-api-id.execute-api.your-region.amazonaws.com/{shortCode}
   ```

This setup provides a basic in-memory URL shortener using AWS Lambda and Go.
