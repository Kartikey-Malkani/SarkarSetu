# Sarkar Setu - API Overview

Complete reference for the Sarkar Setu REST API endpoints, request/response formats, and usage examples.

## Quick Start

### Base URL
```
https://api.sarkarsetu.example.com
```

### Interactive Documentation
- **Swagger UI**: https://api.sarkarsetu.example.com/docs
- **ReDoc**: https://api.sarkarsetu.example.com/redoc
- **OpenAPI Schema**: https://api.sarkarsetu.example.com/openapi.json

### Authentication
Current version uses public endpoints. Future versions will support:
- JWT tokens
- OAuth 2.0
- API keys

---

## Public Endpoints

### 1. Get All Schemes

**Endpoint**
```
GET /schemes
```

**Parameters**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `limit` | integer | No | Items per page (default: 20) |
| `sort_by` | string | No | Sort field (default: "name") |
| `order` | string | No | Sort order: "asc" or "desc" (default: "asc") |

**Example Request**
```bash
curl -X GET "https://api.sarkarsetu.example.com/schemes?page=1&limit=10"
```

**Example Response**
```json
{
  "data": [
    {
      "id": "scheme_001",
      "name": "Pradhan Mantri Aawas Yojana",
      "ministry": "Ministry of Housing",
      "description": "Housing for all scheme",
      "benefit_amount": 500000,
      "state_specific": false,
      "portal_url": "https://pmaymis.gov.in"
    },
    {
      "id": "scheme_002",
      "name": "PMJDY",
      "ministry": "Ministry of Finance",
      "description": "Jan Dhan Yojana for financial inclusion",
      "benefit_amount": 0,
      "state_specific": true,
      "portal_url": "https://pmjdy.gov.in"
    }
  ],
  "total": 156,
  "page": 1,
  "pages": 16
}
```

**Status Codes**
- `200 OK` - Successful
- `400 Bad Request` - Invalid parameters
- `500 Server Error` - Server issue

---

### 2. Get Scheme Details

**Endpoint**
```
GET /schemes/{scheme_id}
```

**Parameters**
| Parameter | Type | Located In | Description |
|-----------|------|----------|-------------|
| `scheme_id` | string | URL path | Scheme identifier |

**Example Request**
```bash
curl -X GET "https://api.sarkarsetu.example.com/schemes/scheme_001"
```

**Example Response**
```json
{
  "id": "scheme_001",
  "name": "Pradhan Mantri Aawas Yojana",
  "ministry": "Ministry of Housing and Urban Affairs",
  "description": "Housing for all initiative providing subsidized homes",
  "launch_date": "2015-06-25",
  "benefit_amount": 500000,
  "benefit_type": "Housing Subsidy",
  "eligibility_criteria": {
    "income_limit": 600000,
    "age_limit": null,
    "education_requirement": "Literate",
    "state_specific": false,
    "required_documents": ["Aadhaar", "Bank Account", "Income Proof"]
  },
  "state_specific": false,
  "valid_states": ["All"],
  "portal_url": "https://pmaymis.gov.in",
  "contact_info": {
    "phone": "1800-11-1111",
    "email": "support@housing.gov.in",
    "website": "https://pmay.gov.in"
  },
  "status": "active"
}
```

**Status Codes**
- `200 OK` - Scheme found
- `404 Not Found` - Scheme not found
- `500 Server Error` - Server issue

---

### 3. Search Schemes

**Endpoint**
```
GET /schemes/search
```

**Parameters**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | Yes | Search query |
| `type` | string | No | Scheme type filter |
| `ministry` | string | No | Ministry filter |

**Example Request**
```bash
curl -X GET "https://api.sarkarsetu.example.com/schemes/search?q=housing&type=subsidy"
```

**Example Response**
```json
{
  "results": [
    {
      "id": "scheme_001",
      "name": "Pradhan Mantri Aawas Yojana",
      "relevance_score": 0.95,
      "matched_fields": ["name", "description"]
    },
    {
      "id": "scheme_023",
      "name": "Housing for Urban Poor",
      "relevance_score": 0.87,
      "matched_fields": ["description"]
    }
  ],
  "total_results": 2,
  "search_type": "semantic"
}
```

**Status Codes**
- `200 OK` - Search successful
- `400 Bad Request` - Missing query parameter
- `500 Server Error` - Server issue

---

## Data Processing Endpoints

### 4. Get Personalized Recommendations

**Endpoint**
```
POST /recommend
```

**Request Body**
```json
{
  "age": 28,
  "income": 600000,
  "state": "Maharashtra",
  "occupation": "Engineer",
  "education_level": "Bachelor",
  "gender": "Male",
  "family_size": 4,
  "income_source": "Salary",
  "employment_status": "Employed"
}
```

**Body Parameters**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `age` | integer | Yes | User age (18-100) |
| `income` | number | Yes | Annual income in rupees |
| `state` | string | Yes | State name (full name) |
| `occupation` | string | Yes | Current occupation |
| `education_level` | string | Yes | Education level (Literate, 10th, 12th, Bachelor, Master, PhD) |
| `gender` | string | Yes | Gender (Male, Female, Other) |
| `family_size` | integer | No | Family members |
| `income_source` | string | No | Source of income |
| `employment_status` | string | No | Employment status |

**Example Request**
```bash
curl -X POST "https://api.sarkarsetu.example.com/recommend" \
  -H "Content-Type: application/json" \
  -d '{
    "age": 28,
    "income": 600000,
    "state": "Maharashtra",
    "occupation": "Engineer",
    "education_level": "Bachelor",
    "gender": "Male",
    "family_size": 4
  }'
```

**Example Response**
```json
{
  "recommendations": [
    {
      "scheme_id": "scheme_001",
      "scheme_name": "Pradhan Mantri Aawas Yojana",
      "relevance_score": 0.92,
      "match_reason": "Income eligibility within limit",
      "priority": 1,
      "benefit_amount": 500000
    },
    {
      "scheme_id": "scheme_045",
      "scheme_name": "Startup India Scheme",
      "relevance_score": 0.88,
      "match_reason": "Education and occupation match",
      "priority": 2,
      "benefit_amount": 100000
    }
  ],
  "total_recommendations": 12,
  "search_method": "semantic"
}
```

**Status Codes**
- `200 OK` - Recommendations generated
- `400 Bad Request` - Invalid user profile
- `422 Unprocessable Entity` - Missing required fields
- `500 Server Error` - Server issue

---

### 5. Check Eligibility

**Endpoint**
```
POST /check-eligibility
```

**Request Body**
```json
{
  "scheme_id": "scheme_001",
  "user_profile": {
    "age": 28,
    "income": 600000,
    "state": "Maharashtra",
    "education_level": "Bachelor"
  }
}
```

**Example Request**
```bash
curl -X POST "https://api.sarkarsetu.example.com/check-eligibility" \
  -H "Content-Type: application/json" \
  -d '{
    "scheme_id": "scheme_001",
    "user_profile": {
      "age": 28,
      "income": 600000,
      "state": "Maharashtra",
      "education_level": "Bachelor"
    }
  }'
```

**Example Response**
```json
{
  "scheme_id": "scheme_001",
  "scheme_name": "Pradhan Mantri Aawas Yojana",
  "eligible": true,
  "match_percentage": 92,
  "criteria_matched": [
    {
      "criterion": "Income Limit",
      "required": "≤ 6,00,000",
      "user_value": "6,00,000",
      "matched": true
    },
    {
      "criterion": "Age",
      "required": "No limit",
      "user_value": "28",
      "matched": true
    }
  ],
  "criteria_missing": [],
  "next_steps": ["Apply on scheme portal", "Submit required documents"]
}
```

**Status Codes**
- `200 OK` - Eligibility checked
- `400 Bad Request` - Invalid data
- `404 Not Found` - Scheme not found
- `500 Server Error` - Server issue

---

## Application Endpoints

### 6. Submit Application

**Endpoint**
```
POST /apply
```

**Request Body**
```json
{
  "user_id": "USR123",
  "scheme_id": "scheme_001",
  "simulate_portal": false,
  "form_data": {
    "full_name": "John Doe",
    "email": "john@example.com",
    "phone": "9876543210",
    "aadhaar": "1234-XXXX-5678",
    "income": 600000,
    "address": "123 Main St"
  }
}
```

**Body Parameters**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `user_id` | string | Yes | Unique user identifier |
| `scheme_id` | string | Yes | Target scheme ID |
| `simulate_portal` | boolean | No | Disable actual automation (default: true) |
| `form_data` | object | Yes | Scheme-specific form fields |

**Example Request**
```bash
curl -X POST "https://api.sarkarsetu.example.com/apply" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "USR123",
    "scheme_id": "scheme_001",
    "simulate_portal": true,
    "form_data": {
      "full_name": "John Doe",
      "email": "john@example.com",
      "phone": "9876543210"
    }
  }'
```

**Example Response (Simulation Mode)**
```json
{
  "success": true,
  "application_id": "DEMO_ABC123XYZ",
  "scheme_id": "scheme_001",
  "scheme_name": "Pradhan Mantri Aawas Yojana",
  "submission_time": "2024-03-24T15:30:45Z",
  "status": "submitted",
  "message": "Application submitted in simulation mode",
  "next_steps": [
    "Visit scheme portal to check status",
    "Download confirmation receipt",
    "Track application progress"
  ]
}
```

**Example Response (Automation Mode)**
```json
{
  "success": true,
  "application_id": "PMAY_2024_001234",
  "scheme_id": "scheme_001",
  "scheme_name": "Pradhan Mantri Aawas Yojana",
  "submission_time": "2024-03-24T15:30:45Z",
  "status": "submitted",
  "portal_confirmation": "Submitted on official portal",
  "confirmation_reference": "REF#2024001234",
  "message": "Application successfully submitted to government portal"
}
```

**Status Codes**
- `200 OK` - Application submitted successfully
- `400 Bad Request` - Invalid form data
- `404 Not Found` - Scheme or user not found
- `422 Unprocessable Entity` - Missing required fields
- `500 Server Error` - Server issue

---

### 7. Get Application Status

**Endpoint**
```
GET /applications/{application_id}
```

**Parameters**
| Parameter | Type | Located In | Description |
|-----------|------|----------|-------------|
| `application_id` | string | URL path | Application identifier |

**Example Request**
```bash
curl -X GET "https://api.sarkarsetu.example.com/applications/DEMO_ABC123XYZ"
```

**Example Response**
```json
{
  "application_id": "DEMO_ABC123XYZ",
  "user_id": "USR123",
  "scheme_id": "scheme_001",
  "scheme_name": "Pradhan Mantri Aawas Yojana",
  "status": "submitted",
  "submission_date": "2024-03-24T15:30:45Z",
  "last_updated": "2024-03-24T15:30:45Z",
  "timeline": [
    {
      "status": "submitted",
      "timestamp": "2024-03-24T15:30:45Z",
      "message": "Application received"
    }
  ],
  "next_status_update": "2024-03-31T00:00:00Z",
  "contact_info": {
    "phone": "1800-11-1111",
    "email": "support@housing.gov.in"
  }
}
```

**Status Codes**
- `200 OK` - Status retrieved
- `404 Not Found` - Application not found
- `500 Server Error` - Server issue

---

### 8. List User Applications

**Endpoint**
```
GET /applications
```

**Parameters**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string | Yes | User identifier |
| `status` | string | No | Filter by status (submitted, approved, rejected) |
| `page` | integer | No | Page number (default: 1) |
| `limit` | integer | No | Items per page (default: 10) |

**Example Request**
```bash
curl -X GET "https://api.sarkarsetu.example.com/applications?user_id=USR123&status=submitted"
```

**Example Response**
```json
{
  "user_id": "USR123",
  "applications": [
    {
      "application_id": "DEMO_ABC123XYZ",
      "scheme_id": "scheme_001",
      "scheme_name": "Pradhan Mantri Aawas Yojana",
      "status": "submitted",
      "submission_date": "2024-03-24T15:30:45Z"
    },
    {
      "application_id": "DEMO_DEF456UVW",
      "scheme_id": "scheme_002",
      "scheme_name": "PMJDY",
      "status": "received",
      "submission_date": "2024-03-20T10:15:30Z"
    }
  ],
  "total": 2,
  "page": 1,
  "pages": 1
}
```

**Status Codes**
- `200 OK` - Applications retrieved
- `400 Bad Request` - Missing user_id
- `500 Server Error` - Server issue

---

## Error Handling

### Standard Error Response

```json
{
  "error": true,
  "code": "INVALID_REQUEST",
  "message": "User-friendly error message",
  "details": {
    "field": "income",
    "reason": "Must be a positive number"
  },
  "timestamp": "2024-03-24T15:30:45Z"
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|------------|-------------|
| `INVALID_REQUEST` | 400 | Malformed request |
| `MISSING_FIELDS` | 422 | Required fields missing |
| `INVALID_DATA_TYPE` | 422 | Data type mismatch |
| `NOT_FOUND` | 404 | Resource not found |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Permission denied |
| `SERVER_ERROR` | 500 | Internal server error |
| `SERVICE_UNAVAILABLE` | 503 | Service temporarily unavailable |

---

## Rate Limiting

### Limits (Per minute)
- Public endpoints: 60 requests/minute
- Data processing endpoints: 30 requests/minute
- Application endpoints: 20 requests/minute

### Rate Limit Headers

Response includes:
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1711270245
```

### Exceeding Limits

```json
{
  "error": true,
  "code": "RATE_LIMIT_EXCEEDED",
  "message": "You have exceeded the rate limit. Please try again later.",
  "retry_after": 60
}
```

---

## Code Examples

### Python (Requests)

```python
import requests

API_BASE = "https://api.sarkarsetu.example.com"

# Get recommendations
response = requests.post(
    f"{API_BASE}/recommend",
    json={
        "age": 28,
        "income": 600000,
        "state": "Maharashtra",
        "occupation": "Engineer",
        "education_level": "Bachelor"
    }
)

recommendations = response.json()
print(f"Found {len(recommendations['recommendations'])} schemes")
```

### JavaScript (Fetch)

```javascript
const API_BASE = "https://api.sarkarsetu.example.com";

async function getRecommendations(userProfile) {
  const response = await fetch(`${API_BASE}/recommend`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(userProfile)
  });
  
  const data = await response.json();
  console.log(`Found ${data.recommendations.length} schemes`);
  return data;
}
```

### cURL

```bash
# Search schemes
curl -X GET "https://api.sarkarsetu.example.com/schemes/search?q=housing"

# Submit application
curl -X POST "https://api.sarkarsetu.example.com/apply" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "USR123",
    "scheme_id": "scheme_001",
    "simulate_portal": true,
    "form_data": {"full_name": "John Doe"}
  }'
```

---

## Changelog

### Version 1.0.0 (Current)
- ✅ Scheme listing and search
- ✅ Personalized recommendations
- ✅ Eligibility checking
- ✅ Application submission
- ✅ Status tracking
- ✅ Simulation mode

### Version 1.1.0 (Planned)
- 🔄 Authentication (JWT)
- 🔄 Application document upload
- 🔄 Notification webhooks
- 🔄 Batch recommendations

### Version 2.0.0 (Future)
- 🔄 Mobile app API
- 🔄 Real-time status updates (WebSocket)
- 🔄 Advanced filtering and sorting
- 🔄 Third-party integrations

---

## Support

For API questions or issues:
- Email: kartikeymalkani71@gmail.com
- GitHub Issues: https://github.com/Kartikey-Malkani/SarkarSetu/issues
- Documentation: See `/docs` endpoint

---

**Last Updated**: March 2026  
**API Version**: 1.0.0
