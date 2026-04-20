# Security Testing Guide

This document outlines how to run the automated security tests for the Xify API. These tests ensure that endpoints remain protected against common vulnerabilities like Broken Authentication and Insecure Direct Object Reference (IDOR).

## Why We Test
When we expose endpoints that deal with project data (like `/api/projects/save-data`, `/api/projects/data/`, and `/api/projects/versions/`), we must verify that:
1. **Authentication:** The user possesses a valid, unexpired session ID (cookie).
2. **Authorization (Ownership):** The user is only accessing **their own** data, preventing IDOR exploits where one logged-in user tries to guess the email/project name to access another user's data.

## The Test Script
An automated test script is located at: `www/Home/backend/check_security.py`

This script uses FastAPI's `TestClient` to simulate malicious requests against the API endpoints without needing to run the full Docker container.

### What it Tests
- **Test 1:** Attempts to access project versions without any session cookie. Expected to be blocked (401).
- **Test 2:** Attempts to access project versions using a valid session for User A, but requesting data for User B (IDOR). Expected to be blocked (403).
- **Test 3:** Attempts to access actual project payload data using a valid session for User A, but requesting data for User B (IDOR). Expected to be blocked (403).
- **Test 4:** Attempts standard, valid access. Expected to succeed (200).

## How to Run the Tests

To run the security tests manually across the API, execute the following commands from the terminal. 

1. **Navigate to the backend directory:**
   ```bash
   cd ../www/Home/backend
   ```

2. **Ensure test dependencies are installed:**
   Ensure your virtual environment has `httpx` installed (required for testing).
   ```bash
   ./venv/bin/pip install httpx
   ```

3. **Run the script with isolated environment variables:**
   In order to prevent the tests from overriding your live production database or logging infrastructure, the script should be run with temporary local target variables.
   ```bash
   DATABASE_URL="sqlite:///./test_security.db" LOG_FILE="./test_log.txt" ./venv/bin/python check_security.py
   ```

### Expected Output
If the endpoints are successfully secured against unauthenticated access and IDOR attacks, the script will output:

```text
Setting up test data...

--- Test 1: get_project_versions without session ---
Status Code: 401 (Expected: 401)
✅ SUCCESS: Blocked access without session.

--- Test 2: get_project_versions for different user (IDOR) ---
Status Code: 403 (Expected: 403)
✅ SUCCESS: Blocked IDOR access.

--- Test 3: get_project_data for different user (IDOR) ---
Status Code: 403 (Expected: 403)
✅ SUCCESS: Blocked IDOR access.

--- Test 4: Valid access ---
Status Code: 200 (Expected: 200)
✅ SUCCESS: Valid access works.
```

If you see `❌ VULNERABILITY`, it means the specific protection logic on the route is broken and requires immediate fixing in `main.py`.

## Future Testing
When adding new endpoints to the API in `main.py` that handle sensitive project or user data, you should update the `test_vulnerabilities()` function within `check_security.py` to ensure the new routes are subjected to identical unauthenticated and IDOR test patterns.
