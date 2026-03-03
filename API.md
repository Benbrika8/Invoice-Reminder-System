# API Documentation - Form to Multi-Channel Notifications

Complete API reference for integrating with the workflow webhook.

## Overview

The workflow exposes a webhook endpoint that accepts form submissions via POST requests. Each submission triggers notifications across email, Telegram, and Google Sheets.

## Base URL

```
https://your-n8n-instance.com/webhook/multi-channel-form
```

## Authentication

Currently, no authentication is required. For production use, consider implementing:
- API keys
- JWT tokens
- OAuth2
- IP whitelisting

## Endpoints

### Submit Form

Submits form data for multi-channel notification delivery.

**Endpoint**: `/webhook/multi-channel-form`  
**Method**: `POST`  
**Content-Type**: `application/json`

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Full name of the submitter |
| email | string | Yes | Email address (validated format) |
| phone | string | Yes | Phone number with country code |
| company | string | Yes | Company or organization name |
| service | string | Yes | Requested service or inquiry type |

#### Example Request

```json
{
  "name": "John Smith",
  "email": "john.smith@example.com",
  "phone": "+1234567890",
  "company": "Tech Innovations Inc",
  "service": "Automation Consulting"
}
```

#### Success Response

**Code**: `200 OK`

```json
{
  "success": true,
  "message": "Form submitted successfully"
}
```

#### Error Responses

**Code**: `400 Bad Request`

```json
{
  "success": false,
  "error": "Invalid request format",
  "details": "Missing required field: email"
}
```

**Code**: `500 Internal Server Error`

```json
{
  "success": false,
  "error": "Processing failed",
  "details": "Gmail notification delivery failed"
}
```

## Integration Examples

### JavaScript (Vanilla)

```javascript
async function submitForm(formData) {
  try {
    const response = await fetch('https://your-n8n-instance.com/webhook/multi-channel-form', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(formData)
    });

    const result = await response.json();

    if (result.success) {
      console.log('Form submitted successfully');
    } else {
      console.error('Submission failed:', result.error);
    }
  } catch (error) {
    console.error('Network error:', error);
  }
}

// Usage
submitForm({
  name: "Jane Doe",
  email: "jane.doe@example.com",
  phone: "+19876543210",
  company: "Digital Solutions LLC",
  service: "Web Development"
});
```

### React Hook

```jsx
import { useState } from 'react';

function useFormSubmission() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const submitForm = async (formData) => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch('https://your-n8n-instance.com/webhook/multi-channel-form', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(formData),
      });

      const result = await response.json();

      if (!result.success) {
        throw new Error(result.error);
      }

      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  return { submitForm, loading, error };
}

// Usage in component
function ContactForm() {
  const { submitForm, loading, error } = useFormSubmission();

  const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = {
      name: e.target.name.value,
      email: e.target.email.value,
      phone: e.target.phone.value,
      company: e.target.company.value,
      service: e.target.service.value,
    };

    try {
      await submitForm(formData);
      alert('Form submitted successfully!');
    } catch (error) {
      alert('Submission failed: ' + error.message);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      <button type="submit" disabled={loading}>
        {loading ? 'Submitting...' : 'Submit'}
      </button>
      {error && <p className="error">{error}</p>}
    </form>
  );
}
```

### Python (requests)

```python
import requests

def submit_form(form_data):
    url = "https://your-n8n-instance.com/webhook/multi-channel-form"
    headers = {"Content-Type": "application/json"}

    try:
        response = requests.post(url, json=form_data, headers=headers)
        response.raise_for_status()

        result = response.json()
        if result.get("success"):
            print("Form submitted successfully")
            return True
        else:
            print(f"Submission failed: {result.get('error')}")
            return False

    except requests.exceptions.RequestException as e:
        print(f"Network error: {e}")
        return False

# Usage
form_data = {
    "name": "Alice Johnson",
    "email": "alice.j@example.com",
    "phone": "+14445556666",
    "company": "Innovation Labs",
    "service": "AI Integration"
}

submit_form(form_data)
```

### PHP (cURL)

```php
<?php

function submitForm($formData) {
    $url = "https://your-n8n-instance.com/webhook/multi-channel-form";

    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Content-Type: application/json'
    ]);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($formData));

    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    if ($httpCode === 200) {
        $result = json_decode($response, true);
        return $result['success'] ?? false;
    }

    return false;
}

// Usage
$formData = [
    "name" => "Bob Wilson",
    "email" => "bob.w@example.com",
    "phone" => "+15551234567",
    "company" => "StartupCo",
    "service" => "Cloud Migration"
];

if (submitForm($formData)) {
    echo "Form submitted successfully!";
} else {
    echo "Submission failed!";
}
?>
```

### cURL Command

```bash
curl -X POST https://your-n8n-instance.com/webhook/multi-channel-form \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sarah Martinez",
    "email": "sarah.m@example.com",
    "phone": "+16667778888",
    "company": "Enterprise Corp",
    "service": "Security Audit"
  }'
```

## Webhook Behavior

### Data Processing Flow

1. **Webhook Trigger**: Receives POST request
2. **Field Extraction**: Extracts form fields from request body
3. **Data Enrichment**: Adds timestamp and source metadata
4. **Parallel Distribution**:
   - Gmail: Sends HTML-formatted email
   - Telegram: Sends instant message
   - Google Sheets: Appends row with status "New"
5. **Response**: Returns success/failure status

### Processed Fields

The workflow automatically adds:

- `timestamp`: ISO 8601 format submission time
- `source`: Fixed value "web" (customizable)
- `status`: Default "New" in Google Sheets

## Rate Limiting

Currently no rate limiting is enforced. For production:

- Implement per-IP rate limiting
- Use authentication with quota management
- Monitor webhook execution metrics

## Error Handling

The workflow handles errors gracefully:

- Invalid JSON returns 400 error
- Missing required fields returns validation error
- Channel failures are logged but don't block other channels
- Google Sheets failures are retried automatically

## Monitoring

Track webhook performance:

- Execution count
- Success rate
- Average processing time
- Channel-specific delivery rates

## Security Best Practices

1. **Use HTTPS**: Always use secure connections
2. **Validate Input**: Sanitize all form inputs
3. **Implement Authentication**: Add API keys or OAuth
4. **Rate Limiting**: Prevent abuse
5. **IP Whitelisting**: Restrict to known sources
6. **Monitor Logs**: Track suspicious activity

## Support

For API-related questions:

- GitHub Issues: [Report a bug](https://github.com/Benbrika8/issues)
- Email: salahbenbrika2@gmail.com
- n8n Community: [Ask questions](https://community.n8n.io/)

---

**Version**: 1.0.0  
**Last Updated**: March 3, 2026
