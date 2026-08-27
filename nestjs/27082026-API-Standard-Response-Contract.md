# API Standard Response Contract

## Overview

เอกสารนี้กำหนดมาตรฐานของ API Response เพื่อให้ทุก Endpoint มีรูปแบบเดียวกัน ทำให้ Frontend, Backend และ Third-party สามารถใช้งานร่วมกันได้อย่างสม่ำเสมอ

---

# Response Structure

API ทุกตัวควรตอบกลับในรูปแบบดังนี้

```json
{
  "success": true,
  "data": {},
  "meta": {},
  "error": null
}
```

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| success | boolean | ✅ | สถานะการทำงานของ API |
| data | object \| array \| null | ✅ | ข้อมูลที่ API ส่งกลับ |
| meta | object \| null | ❌ | ข้อมูลเพิ่มเติม เช่น Pagination |
| error | object \| null | ❌ | รายละเอียด Error |

---

# Success Response

## Single Resource

**HTTP Status**

```
200 OK
```

```json
{
  "success": true,
  "data": {
    "id": "usr_001",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": null,
  "error": null
}
```

---

## Create Resource

**HTTP Status**

```
201 Created
```

```json
{
  "success": true,
  "data": {
    "id": "usr_002",
    "name": "Jane Doe",
    "email": "jane@example.com"
  },
  "meta": null,
  "error": null
}
```

---

## List Response

```json
{
  "success": true,
  "data": [
    {
      "id": "usr_001",
      "name": "John"
    },
    {
      "id": "usr_002",
      "name": "Jane"
    }
  ],
  "meta": null,
  "error": null
}
```

---

## Empty List

ควรคืนค่าเป็น Array ว่าง

```json
{
  "success": true,
  "data": [],
  "meta": null,
  "error": null
}
```

ไม่ควรคืน

```json
{
  "data": null
}
```

---

# Pagination Response

```json
{
  "success": true,
  "data": [
    {}
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 125,
    "totalPages": 7,
    "hasNextPage": true,
    "hasPreviousPage": false
  },
  "error": null
}
```

### Meta Fields

| Field | Type | Description |
|--------|------|-------------|
| page | number | Current page |
| limit | number | Items per page |
| total | number | Total records |
| totalPages | number | Total pages |
| hasNextPage | boolean | Next page exists |
| hasPreviousPage | boolean | Previous page exists |

---

# Error Response

โครงสร้าง Error ควรเหมือนกันทุก API

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found"
  }
}
```

---

# Validation Error

**HTTP Status**

```
422 Unprocessable Entity
```

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Email is invalid"
      },
      {
        "field": "password",
        "message": "Password is required"
      }
    ]
  }
}
```

---

# Unauthorized

**HTTP Status**

```
401 Unauthorized
```

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication required"
  }
}
```

---

# Forbidden

**HTTP Status**

```
403 Forbidden
```

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "FORBIDDEN",
    "message": "Permission denied"
  }
}
```

---

# Not Found

**HTTP Status**

```
404 Not Found
```

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "NOT_FOUND",
    "message": "Resource not found"
  }
}
```

---

# Conflict

**HTTP Status**

```
409 Conflict
```

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "EMAIL_ALREADY_EXISTS",
    "message": "Email already exists"
  }
}
```

---

# Internal Server Error

**HTTP Status**

```
500 Internal Server Error
```

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "Unexpected server error"
  }
}
```

---

# Async Response

สำหรับงานที่ใช้ Queue หรือ Background Job

**HTTP Status**

```
202 Accepted
```

```json
{
  "success": true,
  "data": {
    "jobId": "job_123456"
  },
  "meta": null,
  "error": null
}
```

---

# Delete Response

ถ้าไม่มีข้อมูลต้องส่งกลับ

**HTTP Status**

```
204 No Content
```

Body

```
(No Response Body)
```

หรือหากต้องการส่งข้อความกลับ

```json
{
  "success": true,
  "data": {
    "message": "Deleted successfully"
  },
  "meta": null,
  "error": null
}
```

---

# Recommended Error Codes

| Code | Description |
|------|-------------|
| VALIDATION_ERROR | Request validation failed |
| BAD_REQUEST | Invalid request |
| UNAUTHORIZED | Authentication required |
| FORBIDDEN | Permission denied |
| NOT_FOUND | Resource not found |
| CONFLICT | Resource conflict |
| RATE_LIMIT_EXCEEDED | Too many requests |
| INTERNAL_SERVER_ERROR | Unexpected server error |

---

# Recommended HTTP Status Codes

| HTTP Status | Description |
|-------------|-------------|
| 200 | Success |
| 201 | Resource Created |
| 202 | Accepted (Async Job) |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Validation Error |
| 429 | Too Many Requests |
| 500 | Internal Server Error |

---

# Best Practices

- ใช้โครงสร้าง Response เดียวกันทุก Endpoint
- `success` ควรเป็น `true` เมื่อ HTTP Status เป็น 2xx และ `false` เมื่อเป็น 4xx หรือ 5xx
- `data` เป็น `null` เมื่อเกิด Error
- `meta` ใช้สำหรับข้อมูลเพิ่มเติม เช่น Pagination หรือ Cursor
- `error.code` ควรเป็นค่าคงที่ (Machine-readable) เพื่อให้ Frontend ใช้อ้างอิงได้
- `error.message` เป็นข้อความสำหรับแสดงผลหรือบันทึก Log
- สำหรับรายการข้อมูลที่ว่าง ให้ส่ง `[]` แทน `null`
- ใช้ HTTP Status Code ให้ตรงกับความหมายของผลลัพธ์ ไม่ใช้ `200 OK` สำหรับทุกกรณี