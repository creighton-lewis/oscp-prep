


# Practice 
# Overview 

1. **Identify input reflection points** — search the response HTML for your input string
2. **Determine the injection context** — is it in HTML body, attribute, JS, or URL?
3. **Test basic payloads** — start with the simplest payload for the context
4. **Identify filtered characters** — try angle brackets, quotes, parentheses, event handler keywords
5. **Apply bypass techniques** — encoding, case variation, alternate tags/events, WAF-specific tricks
6. **Escalate** — once XSS is confirmed, escalate from alert() to cookie theft, session hijacking, or CSRF

# Resources 
https://brandonrussell.io/OSCP-Notes/XXMore.html
[Burp Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
[XSS Payloads](https://github.com/justdsn/XSSPayloads/blob/main/cookie_based_xss_payloads.txt)

https://github.com/justdsn/XSSPayloads
https://swisskyrepo.github.io/PayloadsAllTheThings/XSS%20Injection/1%20-%20XSS%20Filter%20Bypass/#bypass-quotes-for-string

```
wget https://raw.githubusercontent.com/ScriptIdiot/XSS-wordlist/refs/heads/master/XSS-WAF.txt
wget https://raw.githubusercontent.com/zapstiko/wordlists/965b4f46cc8c3a1e420e3564a571b49d5708a796/xss%20pyloads2


```

# Types 

## Stored/Persistent XSS

## Reflected XSS
- input reaches back server and gets returned to us without being filtered or sanitized
- if a GET request is what is used to submit the payload, which sends data and parameters as part of url, then it can be sent to user and when they visit the site, the xss will execute 
```
https://website.com?term=gift 

If the website shows something like 

`<p>You searched for: gift</p>`

Then you can execute an attack like this: 

`https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>`

```


## DOM XSS
- source: takes user input 
- sink: writes user input to page 
- if there is no sanitization between the source and the sink, then a website is vulnerable to DOM XSS 
- script.js 
*example of unsanitized*
```javascript
$(function () {
    $("#add").click(function () {
        if ($("#task").val().length > 0) {
            window.location.href = "#task=" + $("#task").val();
            var pos = document.URL.indexOf("task=");
            var task = document.URL.substring(pos + 5, document.URL.length);
            document.getElementById("todo").innerHTML = "<b>Next Task:</b> " + decodeURIComponent(task);
        }
    });
});
var pos = document.URL.indexOf("task=");
var task = document.URL.substring(pos + 5, document.URL.length);
if (pos > 0) {
    document.getElementById("todo").innerHTML = "<b>Next Task:</b> " + decodeURIComponent(task);
}

```

# Tools 
## Dalfox 
- Finds existing XSS vulnerabilities but not great at exploiting any from a payload set
- Takes a very long to time to run 



# Exploitation Opportunities 

## Image Metadata
- Comments can be added to image metadata to cause xss attacks. They can be added to jpg images as well as svg images
## Privilege Escalation 
==need to know for oscp==


