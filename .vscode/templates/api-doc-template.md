# API: <Name>

## Endpoint

`METHOD /path`

## Description

Short description.

## Request

### Headers

| Key           | Required | Example        |
| ------------- | -------- | -------------- |
| Authorization | Yes      | Bearer <token> |

### Query Params

| Name | Type   | Required | Description |
| ---- | ------ | -------- | ----------- |
| page | number | No       | Page index  |

### Body

```json
{
  "example": "value"
}
```

## Response

### 200 OK

```json
{
  "code": 0,
  "message": "ok",
  "data": {}
}
```

### Error Codes

| Code | Meaning       | Action        |
| ---- | ------------- | ------------- |
| 4001 | Invalid Param | Check request |
