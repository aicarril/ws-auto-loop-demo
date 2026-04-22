# Verification Report

**Date**: 2026-04-22T20:37 UTC  
**Verifier**: verifier-1  
**Project**: ws-auto-loop-demo  

---

## Step 1: CloudFront Root URL (index.html)

**Command**: `curl -s -w "HTTP_CODE:%{http_code}" https://d25fb7z284lfov.cloudfront.net/`  
**Result**: ✅ PASS

```
HTTP_CODE:200

<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>ws-auto-loop-demo</title></head>
<body>
  <h1>Hello from S3 + CloudFront</h1>
  <p id="api-response">Loading...</p>
  <script>
    fetch('/api/hello')
      .then(r => r.json())
      .then(d => document.getElementById('api-response').textContent = d.message)
      .catch(() => document.getElementById('api-response').textContent = 'API unavailable');
  </script>
</body>
</html>
```

## Step 2: CloudFront /api/hello (Lambda via API Gateway)

**Command**: `curl -s -w "HTTP_CODE:%{http_code}" https://d25fb7z284lfov.cloudfront.net/api/hello`  
**Result**: ✅ PASS

```
HTTP_CODE:200
{"message":"Hello from Lambda!"}
```

## Step 3: Direct API Gateway Endpoint

**Command**: `curl -s -w "HTTP_CODE:%{http_code}" https://kb94vt4m67.execute-api.us-east-1.amazonaws.com/api/hello`  
**Result**: ✅ PASS

```
HTTP_CODE:200
{"message":"Hello from Lambda!"}
```

## Step 4: CloudFront Custom Error Page (404)

**Command**: `curl -s -w "HTTP_CODE:%{http_code}" https://d25fb7z284lfov.cloudfront.net/nonexistent-page`  
**Result**: ✅ PASS

```
HTTP_CODE:404

<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Error</title></head>
<body><h1>404 - Page Not Found</h1></body>
</html>
```

## Step 5: HTTP → HTTPS Redirect

**Command**: `curl -s -o /dev/null -w "HTTP_CODE:%{http_code}\nREDIRECT:%{redirect_url}" http://d25fb7z284lfov.cloudfront.net/`  
**Result**: ✅ PASS

```
HTTP_CODE:301
REDIRECT:https://d25fb7z284lfov.cloudfront.net/
```

## Step 6: S3 Direct Access Blocked

**Command**: `curl -s -w "HTTP_CODE:%{http_code}" https://ws-auto-loop-demo-779846822196.s3.us-east-1.amazonaws.com/index.html`  
**Result**: ✅ PASS

```
HTTP_CODE:403
<Error><Code>AccessDenied</Code><Message>Access Denied</Message></Error>
```

## Step 7: AWS CLI — S3 Objects

**Command**: `aws s3api list-objects-v2 --bucket ws-auto-loop-demo-779846822196`  
**Result**: ✅ PASS

```json
{
    "Contents": [
        {"Key": "error.html", "Size": 140, "StorageClass": "STANDARD"},
        {"Key": "index.html", "Size": 449, "StorageClass": "STANDARD"}
    ]
}
```

## Step 8: AWS CLI — S3 Public Access Block

**Command**: `aws s3api get-public-access-block --bucket ws-auto-loop-demo-779846822196`  
**Result**: ✅ PASS

```json
{
    "PublicAccessBlockConfiguration": {
        "BlockPublicAcls": true,
        "IgnorePublicAcls": true,
        "BlockPublicPolicy": true,
        "RestrictPublicBuckets": true
    }
}
```

## Step 9: AWS CLI — CloudFront Distribution

**Command**: `aws cloudfront get-distribution --id E26AX6321PXUK8`  
**Result**: ✅ PASS

```json
{
    "Status": "Deployed",
    "DomainName": "d25fb7z284lfov.cloudfront.net",
    "Enabled": true,
    "DefaultRootObject": "index.html"
}
```

## Step 10: AWS CLI — Lambda Function

**Command**: `aws lambda get-function-configuration --function-name ws-auto-loop-demo-api`  
**Result**: ✅ PASS

```json
{
    "FunctionName": "ws-auto-loop-demo-api",
    "Runtime": "nodejs20.x",
    "Handler": "index.handler",
    "State": "Active"
}
```

## Step 11: AWS CLI — API Gateway

**Command**: `aws apigatewayv2 get-api --api-id kb94vt4m67`  
**Result**: ✅ PASS

```json
{
    "Name": "ws-auto-loop-demo-api",
    "ProtocolType": "HTTP",
    "ApiEndpoint": "https://kb94vt4m67.execute-api.us-east-1.amazonaws.com"
}
```

## Step 12: AWS CLI — API Gateway Routes

**Command**: `aws apigatewayv2 get-routes --api-id kb94vt4m67`  
**Result**: ✅ PASS

```json
[{"RouteKey": "GET /api/hello", "Target": "integrations/w8oprkl"}]
```

---

## Summary

| # | Check | Status |
|---|-------|--------|
| 1 | CloudFront root (index.html) | ✅ HTTP 200 |
| 2 | CloudFront /api/hello | ✅ HTTP 200, correct JSON |
| 3 | Direct API Gateway /api/hello | ✅ HTTP 200, correct JSON |
| 4 | Custom error page (404) | ✅ HTTP 404, error.html |
| 5 | HTTP→HTTPS redirect | ✅ 301 redirect |
| 6 | S3 direct access blocked | ✅ HTTP 403 |
| 7 | S3 objects exist | ✅ index.html + error.html |
| 8 | S3 public access block | ✅ All 4 flags true |
| 9 | CloudFront deployed | ✅ Status: Deployed |
| 10 | Lambda active | ✅ State: Active, nodejs20.x |
| 11 | API Gateway exists | ✅ HTTP protocol |
| 12 | API Gateway routes | ✅ GET /api/hello |

**Overall Result**: ✅ ALL 12 CHECKS PASSED
