---
title: Difference Between IDOR and Authorization Bypass
date: 2023-02-14 12:00:00 -500
categories: [blog]
tags: [auth_bypass, idor]     #   TAG should always be lowercase

---

## Understanding the Difference Between IDOR and Authorization Bypass

When it comes to web application security, it's important to understand the various vulnerabilities that can be exploited by attackers to gain unauthorized access to sensitive information or perform malicious actions. Two common types of vulnerabilities that are often confused are Insecure Direct Object Reference (IDOR) and Authorization Bypass. In addition, there are two types of Authorization Bypass, Horizontal and Vertical Privilege Escalation, that can further complicate the issue.

### What is Insecure Direct Object Reference (IDOR)?

Insecure Direct Object Reference (IDOR) is a vulnerability that occurs when an application exposes a reference to an internal object, such as a file, database record, or resource, in a way that allows an attacker to manipulate the reference to access unauthorized data or functionality.

For example, imagine a web application that allows users to view and edit their personal information by navigating to a URL like https://example.com/profile/1234. In this case, the number 1234 is used to identify the user's profile in the application's database. If the application doesn't properly check that the user is authorized to access that profile, an attacker could simply change the number in the URL to access the profile of a different user, or even a system administrator.


*Here are some steps to follow when testing for IDORs:*

1. Identify sensitive resources: Start by identifying any resources that contain sensitive data or perform privileged actions.

2. Authenticate: Attempt to access the sensitive resource by authenticating using valid credentials.

3. Test for direct object references: Test the application for direct object references by manipulating the parameters of requests. Look for any parameters that reference specific objects, such as user IDs or order IDs.

4. Test for privilege escalation: Look for ways to escalate privileges by accessing sensitive information or performing actions that should only be available to privileged users.

5. Automate testing: Use automated testing tools like Burp Suite to test for common IDOR vulnerabilities.

*Here is a Burp request example of an IDOR vulnerability:*

```vbnet
GET /order?id=123 HTTP/1.1
Host: vulnerable-app.com
Cookie: sessionid=abcdef123456

```
*To exploit this vulnerability, the attacker could simply change the ID parameter to a different value, such as:*

```vbnet
GET /order?id=456 HTTP/1.1
Host: vulnerable-app.com
Cookie: sessionid=abcdef123456
```

### What is Authorization Bypass?

Authorization Bypass is a vulnerability that occurs when an attacker is able to perform actions that they should not have access to due to a failure in the web application's access control mechanisms. The attacker can bypass the authorization checks and gain unauthorized access to sensitive data or perform privileged actions.

### There are two types of Authorization Bypass: Horizontal and Vertical Privilege Escalation.


*Horizontal Privilege Escalation* occurs when an attacker is able to access resources or perform actions that are at the same level of privilege as the user they are impersonating. For example, if a user has access to a certain set of files, an attacker could use an Authorization Bypass vulnerability to access those same files, but not any others.

*Vertical Privilege Escalation* occurs when an attacker is able to access resources or perform actions that are at a higher level of privilege than the user they are impersonating. For example, if a user has limited access to a certain set of files, an attacker could use an Authorization Bypass vulnerability to access files that only an administrator should be able to access.

*Here are some steps to follow when testing for Authorization Bypasses:*

1. Identify restricted resources: Start by identifying any resources that require authentication or authorization to access. This may include certain pages, directories, or API endpoints.

2. Authenticate: Attempt to access the restricted resource by authenticating using valid credentials.

3. Manipulate authentication: Manipulate the authentication mechanism to gain access to the restricted resource. This may include modifying session IDs, cookies, or other authentication parameters.

4. Test for access control vulnerabilities: Test the application for common access control vulnerabilities, such as insecure direct object reference (IDOR), vertical and horizontal privilege escalation, parameter tampering, and business logic vulnerabilities.

5. Test for privilege escalation: Look for ways to escalate privileges by accessing sensitive information or performing actions that should only be available to privileged users.

6. Automate testing: Use automated testing tools like Burp Suite to test for common authorization bypass vulnerabilities.

### Example of Authorization Bypass
For example, consider a web application that requires a user to be logged in to access certain resources. The application checks the user's authentication status by validating a session ID parameter in the request. If the application does not validate this parameter properly, an attacker could manipulate the session ID parameter to gain access to the restricted resources.

*Burp Request Example of Authorization Bypass:* 

```vbnet
GET /admin HTTP/1.1
Host: example.com
Cookie: session_id=abcd1234
```
In the above example, an attacker could manipulate the session ID parameter to access the administrative resources.

## Conclusion

In conclusion, IDOR and Authorization Bypass are two common web application vulnerabilities that can lead to unauthorized access or manipulation of data. IDOR occurs when a web application fails to validate user input, allowing an attacker to access resources they should not have access to. Authorization Bypass occurs when an attacker is able to perform actions they should not have access to due to a failure in the web application's access control mechanisms. It's important to note that every application is different, so there is no single method that will work for all applications. However, the steps provided can be used as a starting point for testing for Authorization Bypasses and IDORs. In addition, testers should stay up-to-date with the latest techniques and tools for finding these vulnerabilities.

*References:*

https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema

https://portswigger.net/web-security/access-control

https://portswigger.net/web-security/access-control/idor

![gif](/assets/img/gif.gif)

Post written by: *Coppertop*