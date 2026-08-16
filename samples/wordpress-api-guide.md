# Developer Guide: Custom WordPress REST API Endpoint

## 📋 Context
* **Goal:** Document a custom WordPress REST API endpoint used to programmatically register students into specific courses from external systems.
* **Target Audience:** Frontend Developers and Full-Stack Engineers integrating external checkout software with a WordPress LMS.
* **Tools Used:** Markdown, Postman, Open-API/Swagger specification.

---

### 📌 Overview
The Custom Enrollment API allows external applications to securely enroll existing users into specific WordPress LMS courses. 

* **Base URL:** `https://yourdomain.com`
* **Authentication:** Requires a Bearer Token passed via the HTTP Authorization header.

### 🛣️ Endpoints

#### Register a Student to a Course
`POST /enroll`

Enrolls a specific user into a specified course ID. If the user is already enrolled, the API returns a success message confirming active status.

**Headers:**
```http
Authorization: Bearer <your_jwt_token>
Content-Type: application/json
```

#### Request Body Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `user_id` | Integer | **Yes** | The unique WordPress ID of the student. |
| `course_id` | Integer | **Yes** | The unique post ID of the target LMS course. |
| `status` | String | No | The enrollment status. Allowed values: `active`, `pending`. Default is `active`. |

#### Example Request Body
```json
{
  "user_id": 402,
  "course_id": 1829,
  "status": "active"
}
```

---

### 🔄 API Responses

#### Success Response (`200 OK`)
```json
{
  "success": true,
  "message": "User 402 successfully enrolled in course 1829.",
  "data": {
    "enrollment_id": 45091,
    "date_created": "2026-08-16T09:42:00Z",
    "status": "active"
  }
}
```

#### Error Responses

##### `400 Bad Request`
```json
{
  "code": "rest_missing_callback_param",
  "message": "Missing parameter(s): course_id",
  "data": {
    "status": 400
  }
}
```

##### `401 Unauthorized`
```json
{
  "code": "jwt_auth_invalid_token",
  "message": "Expired token or signature verification failed.",
  "data": {
    "status": 401
  }
}
```

---

### 💻 Code Snippet (JavaScript Fetch Example)

```javascript
const enrollStudent = async () => {
  const url = 'https://yourdomain.com/enroll';
  const payload = {
    user_id: 402,
    course_id: 1829,
    status: 'active'
  };

  try {
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer YOUR_TOKEN_HERE'
      },
      body: JSON.stringify(payload)
    });

    const result = await response.json();
    console.log('Enrollment Result:', result);
  } catch (error) {
    console.error('API Error:', error);
  }
};
```
