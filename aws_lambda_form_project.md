# AWS Lambda + API Gateway + S3 Form Project

## Project Objectives

1. Create a Lambda function that processes HTTP requests
2. Expose the Lambda function via API Gateway
3. Update their S3-hosted website to call the API
4. Integrate frontend forms with serverless backend logic

## Part 1: Create the Lambda Function

### Create Lambda Function in AWS Console

1. Go to **AWS Lambda** → **Create function**
2. Function name: `GreetingFunction`
3. Runtime: **Python 3.11** (or latest available)
4. Create function
5. Replace the code in the editor with:

```python
import json
import logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(f'Event received: {event}')
    
    try:
        # Parse the incoming request body
        if isinstance(event.get('body'), str):
            body = json.loads(event['body'])
        else:
            body = event.get('body', {})
    except json.JSONDecodeError:
        return {
            'statusCode': 400,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
                'Access-Control-Allow-Headers': 'Content-Type',
                'Access-Control-Allow-Methods': 'OPTIONS,POST'
            },
            'body': json.dumps({'error': 'Invalid JSON in request body'})
        }

    name = body.get('name', '').strip()

    # Validate that name was provided
    if not name:
        return {
            'statusCode': 400,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
                'Access-Control-Allow-Headers': 'Content-Type',
                'Access-Control-Allow-Methods': 'OPTIONS,POST'
            },
            'body': json.dumps({'error': 'Name is required'})
        }
    
    # Return the greeting
    response = {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Headers': 'Content-Type',
            'Access-Control-Allow-Methods': 'OPTIONS,POST'
        },
        'body': json.dumps({
            'message': f'Hello {name}!'
        })
    }

    return response
```

[Lambda Function Code Breakdown](/lambda-breakdown.md)

6. Click **Deploy**

### Test the Lambda Function

1. Go to **Test** tab
2. Create a new test event with:

```json
{
  "body": "{\"name\": \"Alice\"}"
}
```

3. Click **Test** and verify it returns `{"message": "Hello Alice!"}`

## Part 2: Create API Gateway Endpoint

### Create API Gateway

1. Go to **API Gateway** → **Create API**
2. Choose **REST API** → **Build**
3. API name: `GreetingAPI`
4. Endpoint type: **Regional**
5. Click **Create API**
6. Select root `/` and **Create resource**
7. On the right give your resource a name (e.g. `greet`)

### Create a POST Method

1. In Resources, select your resource (e.g. `greet`)
2. Click **Create Method** → **POST**
3. Integration type: **Lambda Function**
4. Lambda Function: `GreetingFunction`
5. Enable **Lambda Proxy Integration**.
6. Click **Save**
7. Accept any Lambda permission popup

### Add OPTIONS Method for CORS

1. In Resources, select your resource (e.g. `greet`)
2. Click **Create Method** → **OPTIONS**
3. Select Integration type: `Mock`
4. Click Save
5. Navigate back to your API and resource, and select **OPTIONS**
6. Add the following `Method Response Headers`:

```sh
Access-Control-Allow-Origin
Access-Control-Allow-Methods
Access-Control-Allow-Headers
Content-Type
```

7. Navigate to Integration Response
8. Add the following `header mappings`:

```sh
Access-Control-Allow-Origin: '*'
Access-Control-Allow-Methods: 'OPTIONS,POST'
Access-Control-Allow-Headers: 'Content-Type'
Content-Type: 'application/json'
```

### Enable CORS

CORS stands for Cross-Origin Resource Sharing. It's a security feature implemented by web browsers to control how web pages can request resources from a different domain (origin) than the one that served the web page.

By default, browsers block these cross-origin requests unless the server at `api.anotherdomain.com` explicitly allows it. This prevents malicious websites from making unauthorized requests to other sites on behalf of a user.

If you're using `API Gateway` + `Lambda`, you need to configure CORS in API Gateway to allow your frontend (e.g., hosted on a different domain) to communicate with the backend.

1. Select the **POST** method
2. Click **Method Request**
3. Go back and click on the root `/`
4. Click **Enable CORS and replace existing CORS headers**
5. Check all methods and codes (4XX, 5XX, POST, OPTIONS)
6. Click **Enable CORS**

### Deploy the API

1. Click **Deploy API**
2. Deployment stage: Create new stage called `prod`
3. Click **Deploy**
4. Copy your **Invoke URL** (looks like: `https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/stage/resource`)

>NOTE: You may need to add the `stage` and `resource` manually

## Part 3: Update Your Website

### Create HTML Form

Add this to your website's HTML file. Update the `API_ENDPOINT` variable with your actual API Gateway URL:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Greeting App</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 20px;
        }
        
        .form-container {
            border: 1px solid #ccc;
            padding: 20px;
            border-radius: 5px;
            background-color: #f9f9f9;
        }
        
        input {
            padding: 10px;
            width: 100%;
            max-width: 300px;
            margin: 10px 0;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }
        
        button {
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        
        button:hover {
            background-color: #45a049;
        }
        
        button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }
        
        .response {
            margin-top: 20px;
            padding: 15px;
            border-radius: 4px;
            display: none;
        }
        
        .response.success {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
            display: block;
        }
        
        .response.error {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
            display: block;
        }
        
        .loading {
            display: none;
            margin-top: 10px;
        }
        
        .loading.show {
            display: block;
        }
    </style>
</head>
<body>
    <h1>Greeting Application</h1>
    
    <div class="form-container">
        <h2>Enter Your Name</h2>
        <form id="greetingForm">
            <input 
                type="text" 
                id="nameInput" 
                placeholder="Enter your name" 
                required
            >
            <button type="submit">Submit</button>
        </form>
        
        <div class="loading" id="loading">
            Loading...
        </div>
        
        <div class="response" id="response"></div>
    </div>

    <script>
        // Replace this with your actual API Gateway endpoint
        const API_ENDPOINT = 'https://[YOUR_API_ENDPOINT]/[STAGE]/[RESOURCE]';
        
        const form = document.getElementById('greetingForm');
        const nameInput = document.getElementById('nameInput');
        const responseDiv = document.getElementById('response');
        const loadingDiv = document.getElementById('loading');
        
        form.addEventListener('submit', async (e) => {
            e.preventDefault();
            
            const name = nameInput.value.trim();
            
            if (!name) {
                showResponse('Please enter your name', 'error');
                return;
            }
            
            // Show loading state
            loadingDiv.classList.add('show');
            responseDiv.classList.remove('success', 'error');
            
            try {
                const response = await fetch(API_ENDPOINT, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ name: name })
                });
                
                const data = await response.json();
                
                if (response.ok) {
                    showResponse(data.message, 'success');
                    nameInput.value = ''; // Clear input
                } else {
                    showResponse(data.error || 'An error occurred', 'error');
                }
            } catch (error) {
                console.error('Error:', error);
                showResponse('Failed to connect to the server. Check the console.', 'error');
            } finally {
                loadingDiv.classList.remove('show');
            }
        });
        
        function showResponse(message, type) {
            responseDiv.textContent = message;
            responseDiv.className = `response ${type}`;
        }
    </script>
</body>
</html>
```

[JavaScript Code Breakdown](/javascript-breakdown.md)

### Update Your GitHub Repo

1. Replace (or add to) your existing HTML file with the code above
2. Update the `API_ENDPOINT` variable with your actual API Gateway URL
3. Commit and push to GitHub
4. Your GitHub Actions workflow will automatically deploy to S3

### Test End-to-End

1. Visit your S3 website
2. Enter a name and click Submit
3. You should see "Hello [name]!" appear below the form

## Debugging Tips for Learners

**No response or blank response:**

- Check browser console (F12) for errors
- Verify `API_ENDPOINT` URL is correct
- Confirm Lambda function is deployed
- Check API Gateway stage is correct (`/prod`)

**CORS errors:**

- Confirm CORS was enabled in API Gateway
- Error message will mention "Access-Control-Allow-Origin"

**400 Bad Request:**

- Verify the Lambda function is receiving the JSON with `name` field
- Check CloudWatch logs in Lambda console

**Check Lambda Logs:**

- Go to `Lambda` → `GreetingFunction` → `Monitor` → `Logs`
- View the most recent log stream to see what the function received

## Extension Ideas

After the basic project works, learners can:

- Add more form fields (email, message)
- Store submissions in DynamoDB
- Add authentication via Cognito
- Create different endpoints for different operations
- Add error handling and validation on the backend
- Use environment variables to make the API endpoint configurable
