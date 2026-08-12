# Read a Website like a Pentester: what to test in every functionality

Instead of only asking "what does this feature do?", a pentester asks: "What could go wrong with this feature?"

Before testing for vulnerabilities, learn to read the application. 
The goal isn't to memorize a list of vulnerabilities. The goal is to develop the habit of looking at a feature and immediately thinking: "what should I test here?" 

## 1. Search Function
Normal user sees a search box, but a pentester sees an input controlled by the user.
Things to consider:
- Reflected XSS
- Stored XSS if searches are saved
- SQL/NoSQL injection depending on backend
- Command injection in unusual cases
- Paramenter manipulation
- Excessive resource consumption/ DoS risk
- Search result access-control issues

## 2. Login Page
Normal user sees: Enter username + password -> Login but a pentester thinks: what happens when I manipulate this process?
Things to consider:
- Authentication bypass
- Weak password policy
- Username enumeration
- Brute-force protection
- Rate limiting
- Account lockout behavior
- Session management

## 3. Registration
Think about:
- User enumeration
- Weak password requirements
- Duplicate account creation
- Email verification bypass
- Rate limiting
- Parameter manipulation

If registration accepts:
```
role=user
```
Ask: what happens if the role parameter is modified? This introduces privilege-related testing without going too deep.

## 4. Forgot Password/ Reset Password
Check:
- User enumeration
- Rate limiting
- Reset token predictability
- Token expiration
- Token reuse
- Token invalidation after password change
- Host header related issues where relevant
- Password reset flow manipulation

Simple scenario:
Request a password reset -> receive token -> use token
Then ask: Is the token still valid after it has already been used? or Does the token expire? 

## 5. Profile/ Account Settings
Things to consider:
- IDOR/BOLA
- Mass assignment
- Stored XSS
- CSRF
- Unauthorized modification
- Sensitive information exposure

## 6. File Upload 
Normal user sees the functionality to upload a profile picture but a pentester thinks "what exactly does the server validate?". 
Things to consider:
- File type validation
- Extension validation
- MIME validation
- File size limits
- Filename handling
- Storage location
- Access control
- Malicious file handling
- Path traversal

## 7. Download Functionality 
For Downloading invoice for example.
Things to consider:
- IDOR/BOLA
- Path traversal
- Unauthorized file access
- Sensitive information exposure
- Filename manipulation

Example:
```
/download?id=123
```
Ask: What happens if I change the ID?

## 8. Contact/ Feedback Form
Think about the following:
- Stored XSS
- Reflected XSS
- HTML injection
- Email/header injection where applicable
- CSRF
- Rate limiting
- Sensitive information leakage

## 9. Additional 
The functionalities discussed above are only a few common examples. Real-world web applications can contain dozens of other features, and each may introduce different security concerns.

- Comments / Reviews — Stored XSS, HTML Injection, Authorization
- Change Password — Authentication, CSRF, Session Management
- Logout — Session Invalidation
- Admin Panel — Privilege Escalation, Broken Access Control
- User Management — IDOR / BOLA, Privilege Escalation
- Notifications — IDOR, XSS, Information Disclosure
- Email Verification — Token Security, Workflow Bypass
- Invitation System — Authorization, Token Security, Rate Limiting
- Checkout / Cart — Business Logic, Price Manipulation, Authorization
- Payment Functions — Business Logic, Parameter Tampering, Replay Attacks
- API Documentation / Swagger — Endpoint Discovery, Authorization, Information Exposure
- WebSockets / Real-time Features — Authentication, Authorization, Input Validation
- Export / Reports — Authorization, Sensitive Data Exposure
- Password-Protected Actions — Reauthentication, Authorization
- Account Deletion — Authorization, CSRF, Business Logic

## 10. API Endpoints
This deserves its own section because modern websites heavily depend on APIs.
Pentester questions:
- Can I access another user’s object?
- Does the API enforce authorization?
- Can I modify parameters?
- Are sensitive fields exposed?
- Are HTTP methods properly restricted?
- Is rate limiting implemented?
- Can requests be replayed?
