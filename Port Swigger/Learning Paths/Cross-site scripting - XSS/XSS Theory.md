https://portswigger.net/web-security/cross-site-scripting

<script>alert(1)</script>



---

Aquí van las defensas principales contra este tipo de ataque (XSS que roba cookies):

### 1. Cookies `HttpOnly`

Si la cookie de sesión se marca como `HttpOnly`, JavaScript **no puede leerla** con `document.cookie`. Aunque exista una vulnerabilidad XSS, el script malicioso no tendría nada que robar.

```
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

### 2. Content Security Policy (CSP)

Una cabecera CSP le dice al navegador qué scripts puede ejecutar y a qué dominios puede hacer peticiones. Por ejemplo:

```
Content-Security-Policy: default-src 'self'; script-src 'self'; connect-src 'self'
```

Esto bloquearía tanto la ejecución de scripts inyectados como el `fetch` hacia dominios externos, incluso si el XSS lograra colarse.

### 3. Sanitización y escape de inputs/outputs

- Nunca insertar contenido generado por el usuario directamente en el HTML sin escaparlo (usar funciones como `textContent` en vez de `innerHTML`, o librerías como DOMPurify si necesitas HTML enriquecido).
- Validar y sanitizar tanto en el cliente como en el servidor (el cliente se puede saltar).

### 4. Cookies `SameSite`

Con `SameSite=Strict` o `Lax`, la cookie no se envía en peticiones que vienen de otro sitio, lo que mitiga ataques tipo CSRF y limita el impacto de ciertos XSS.

### 5. Tokens CSRF bien implementados

El token CSRF debe:

- Ser de un solo uso o rotar frecuentemente.
- Verificarse en el servidor comparando con el que está asociado a la sesión.
- No servir de nada si el atacante ya tiene XSS (como en tu ejemplo, el propio script roba el token de la página), así que **no sustituye** a las otras defensas.

### 6. Principio de defensa en profundidad

Ninguna medida es suficiente por sí sola. Lo ideal es combinar CSP + HttpOnly + sanitización + SameSite, para que si una capa falla, las otras sigan protegiendo.

	¿Quieres que veamos cómo aplicarían estas defensas a un caso concreto (por ejemplo, una app en PHP, Node, etc.)?

---


# Cross-site scripting

In this section, we'll explain what cross-site scripting is, describe the different varieties of cross-site scripting vulnerabilities, and spell out how to find and prevent cross-site scripting.

## What is cross-site scripting (XSS)?

Cross-site scripting (also known as XSS) is a web security vulnerability that allows an attacker to compromise the interactions that users have with a vulnerable application. It allows an attacker to circumvent the same origin policy, which is designed to segregate different websites from each other. Cross-site scripting vulnerabilities normally allow an attacker to masquerade as a victim user, to carry out any actions that the user is able to perform, and to access any of the user's data. If the victim user has privileged access within the application, then the attacker might be able to gain full control over all of the application's functionality and data.

## How does XSS work?

Cross-site scripting works by manipulating a vulnerable web site so that it returns malicious JavaScript to users. When the malicious code executes inside a victim's browser, the attacker can fully compromise their interaction with the application.


## XSS proof of concept

You can confirm most kinds of XSS vulnerability by injecting a payload that causes your own browser to execute some arbitrary JavaScript. It's long been common practice to use the `alert()` function for this purpose because it's short, harmless, and pretty hard to miss when it's successfully called. In fact, you solve the majority of our XSS labs by invoking `alert()` in a simulated victim's browser.

Unfortunately, there's a slight hitch if you use Chrome. From version 92 onward (July 20th, 2021), cross-origin iframes are prevented from calling `alert()`. As these are used to construct some of the more advanced XSS attacks, you'll sometimes need to use an alternative PoC payload. In this scenario, we recommend the `print()` function. If you're interested in learning more about this change and why we like `print()`, [check out our blog post](https://portswigger.net/research/alert-is-dead-long-live-print) on the subject.

As the simulated victim in our labs uses Chrome, we've amended the affected labs so that they can also be solved using `print()`. We've indicated this in the instructions wherever relevant.

## What are the types of XSS attacks?

There are three main types of XSS attacks. These are:

- [Reflected XSS](https://portswigger.net/web-security/cross-site-scripting#reflected-cross-site-scripting), where the malicious script comes from the current HTTP request.
- [Stored XSS](https://portswigger.net/web-security/cross-site-scripting#stored-cross-site-scripting), where the malicious script comes from the website's database.
- [DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting#dom-based-cross-site-scripting), where the vulnerability exists in client-side code rather than server-side code.



---



## Reflected cross-site scripting

Reflected XSS is the simplest variety of cross-site scripting. It arises when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way.

Here is a simple example of a reflected XSS vulnerability:

`https://insecure-website.com/status?message=All+is+well. <p>Status: All is well.</p>`

The application doesn't perform any other processing of the data, so an attacker can easily construct an attack like this:

`https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script> <p>Status: <script>/* Bad stuff here... */</script></p>`

If the user visits the URL constructed by the attacker, then the attacker's script executes in the user's browser, in the context of that user's session with the application. At that point, the script can carry out any action, and retrieve any data, to which the user has access.

#### Read more
- [Cross-site scripting cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)


# Reflected XSS

In this section, we'll explain reflected cross-site scripting, describe the impact of reflected XSS attacks, and spell out how to find reflected XSS vulnerabilities.

## What is reflected cross-site scripting?

Reflected cross-site scripting (or XSS) arises when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way.

Suppose a website has a search function which receives the user-supplied search term in a URL parameter:

`https://insecure-website.com/search?term=gift`

The application echoes the supplied search term in the response to this URL:

`<p>You searched for: gift</p>`

Assuming the application doesn't perform any other processing of the data, an attacker can construct an attack like this:

`https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>`

This URL results in the following response:

`<p>You searched for: <script>/* Bad stuff here... */</script></p>`

If another user of the application requests the attacker's URL, then the script supplied by the attacker will execute in the victim user's browser, in the context of their session with the application.

## Impact of reflected XSS attacks

If an attacker can control a script that is executed in the victim's browser, then they can typically fully compromise that user. Amongst other things, the attacker can:

- Perform any action within the application that the user can perform.
- View any information that the user is able to view.
- Modify any information that the user is able to modify.
- Initiate interactions with other application users, including malicious attacks, that will appear to originate from the initial victim user.

There are various means by which an attacker might induce a victim user to make a request that they control, to deliver a reflected XSS attack. These include placing links on a website controlled by the attacker, or on another website that allows content to be generated, or by sending a link in an email, tweet or other message. The attack could be targeted directly against a known user, or could be an indiscriminate attack against any users of the application.

The need for an external delivery mechanism for the attack means that the impact of reflected XSS is generally less severe than stored XSS, where a self-contained attack can be delivered within the vulnerable application itself.

#### Read more

- [Exploiting cross-site scripting vulnerabilities](https://portswigger.net/web-security/cross-site-scripting/exploiting)

# Exploiting cross-site scripting vulnerabilities

The traditional way to prove that you've found a cross-site scripting vulnerability is to create a popup using the `alert()` function. This isn't because XSS has anything to do with popups; it's simply a way to prove that you can execute arbitrary JavaScript on a given domain. You might notice some people using `alert(document.domain)`. This is a way of making it explicit which domain the JavaScript is executing on.

Sometimes you'll want to go further and prove that an XSS vulnerability is a real threat by providing a full exploit. In this section, we'll explore three of the most popular and powerful ways to exploit an XSS vulnerability.

## Exploiting cross-site scripting to steal cookies

Stealing cookies is a traditional way to exploit XSS. Most web applications use cookies for session handling. You can exploit cross-site scripting vulnerabilities to send the victim's cookies to your own domain, then manually inject the cookies into the browser and impersonate the victim.

In practice, this approach has some significant limitations:

- The victim might not be logged in.
- Many applications hide their cookies from JavaScript using the `HttpOnly` flag.
- Sessions might be locked to additional factors like the user's IP address.
- The session might time out before you're able to hijack it.


## Exploiting cross-site scripting to capture passwords

These days, many users have password managers that auto-fill their passwords. You can take advantage of this by creating a password input, reading out the auto-filled password, and sending it to your own domain. This technique avoids most of the problems associated with stealing cookies, and can even gain access to every other account where the victim has reused the same password.

The primary disadvantage of this technique is that it only works on users who have a password manager that performs password auto-fill. (Of course, if a user doesn't have a password saved you can still attempt to obtain their password through an on-site phishing attack, but it's not quite the same.)


## Exploiting cross-site scripting to bypass CSRF protections

XSS enables an attacker to do almost anything a legitimate user can do on a website. By executing arbitrary JavaScript in a victim's browser, XSS allows you to perform a wide range of actions as if you were the victim user. For example, you might make a victim send a message, accept a friend request, commit a backdoor to a source code repository, or transfer some Bitcoin.

Some websites allow logged-in users to change their email address without re-entering their password. If you've found an XSS vulnerability on one of these sites, you can exploit it to steal a CSRF token. With the token, you can change the victim's email address to one that you control. You can then trigger a password reset to gain access to the account.

This type of exploit combines XSS (to steal the CSRF token) with the functionality typically targeted by CSRF. While traditional CSRF is a "one-way" vulnerability, where the attacker can induce the victim to send requests but cannot see the responses, XSS enables "two-way" communication. This enables the attacker to both send arbitrary requests and read the responses, resulting in a hybrid attack that bypasses anti-CSRF defenses.

#### Note

CSRF tokens are ineffective against XSS because XSS lets attackers read token values directly from responses.




## Reflected XSS in different contexts

There are many different varieties of reflected cross-site scripting. The location of the reflected data within the application's response determines what type of payload is required to exploit it and might also affect the impact of the vulnerability.

In addition, if the application performs any validation or other processing on the submitted data before it is reflected, this will generally affect what kind of XSS payload is needed.

#### Read more

- [Cross-site scripting contexts](https://portswigger.net/web-security/cross-site-scripting/contexts)

# Cross-site scripting contexts

When testing for [reflected](https://portswigger.net/web-security/cross-site-scripting/reflected) and [stored](https://portswigger.net/web-security/cross-site-scripting/stored) XSS, a key task is to identify the XSS context:

- The location within the response where attacker-controllable data appears.
- Any input validation or other processing that is being performed on that data by the application.

Based on these details, you can then select one or more candidate XSS payloads, and test whether they are effective.

#### Note

We have built a comprehensive [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) to help testing web applications and filters. You can filter by events and tags and see which vectors require user interaction. The cheat sheet also contains AngularJS sandbox escapes and many other sections to help with XSS research.

## XSS between HTML tags

When the XSS context is text between HTML tags, you need to introduce some new HTML tags designed to trigger execution of JavaScript.

Some useful ways of executing JavaScript are:

`<script>alert(document.domain)</script> <img src=1 onerror=alert(1)>`


## XSS in HTML tag attributes

When the XSS context is into an HTML tag attribute value, you might sometimes be able to terminate the attribute value, close the tag, and introduce a new one. For example:

`"><script>alert(document.domain)</script>`

More commonly in this situation, angle brackets are blocked or encoded, so your input cannot break out of the tag in which it appears. Provided you can terminate the attribute value, you can normally introduce a new attribute that creates a scriptable context, such as an event handler. For example:

`" autofocus onfocus=alert(document.domain) x="`

The above payload creates an `onfocus` event that will execute JavaScript when the element receives the focus, and also adds the `autofocus` attribute to try to trigger the `onfocus` event automatically without any user interaction. Finally, it adds `x="` to gracefully repair the following markup.

Sometimes the XSS context is into a type of HTML tag attribute that itself can create a scriptable context. Here, you can execute JavaScript without needing to terminate the attribute value. For example, if the XSS context is into the `href` attribute of an anchor tag, you can use the `javascript` pseudo-protocol to execute script. For example:

`<a href="javascript:alert(document.domain)">`

You might encounter websites that encode angle brackets but still allow you to inject attributes. Sometimes, these injections are possible even within tags that don't usually fire events automatically, such as a canonical tag. You can exploit this behavior using access keys and user interaction on Chrome. Access keys allow you to provide keyboard shortcuts that reference a specific element. The `accesskey` attribute allows you to define a letter that, when pressed in combination with other keys (these vary across different platforms), will cause events to fire. In the next lab you can experiment with access keys and exploit a canonical tag. [You can exploit XSS in hidden input fields using a technique invented by PortSwigger Research](https://portswigger.net/research/xss-in-hidden-input-fields).

## XSS into JavaScript

When the XSS context is some existing JavaScript within the response, a wide variety of situations can arise, with different techniques necessary to perform a successful exploit.

### Terminating the existing script

In the simplest case, it is possible to simply close the script tag that is enclosing the existing JavaScript, and introduce some new HTML tags that will trigger execution of JavaScript. For example, if the XSS context is as follows:

`<script> ... var input = 'controllable data here'; ... </script>`

then you can use the following payload to break out of the existing JavaScript and execute your own:

`</script><img src=1 onerror=alert(document.domain)>`

The reason this works is that the browser first performs HTML parsing to identify the page elements including blocks of script, and only later performs JavaScript parsing to understand and execute the embedded scripts. The above payload leaves the original script broken, with an unterminated string literal. But that doesn't prevent the subsequent script being parsed and executed in the normal way.


### Breaking out of a JavaScript string

In cases where the XSS context is inside a quoted string literal, it is often possible to break out of the string and execute JavaScript directly. It is essential to repair the script following the XSS context, because any syntax errors there will prevent the whole script from executing.

Some useful ways of breaking out of a string literal are:

`'-alert(document.domain)-' ';alert(document.domain)//`


Some applications attempt to prevent input from breaking out of the JavaScript string by escaping any single quote characters with a backslash. A backslash before a character tells the JavaScript parser that the character should be interpreted literally, and not as a special character such as a string terminator. In this situation, applications often make the mistake of failing to escape the backslash character itself. This means that an attacker can use their own backslash character to neutralize the backslash that is added by the application.

For example, suppose that the input:

`';alert(document.domain)//`

gets converted to:

`\';alert(document.domain)//`

You can now use the alternative payload:

`\';alert(document.domain)//`

which gets converted to:

`\\';alert(document.domain)//`

Here, the first backslash means that the second backslash is interpreted literally, and not as a special character. This means that the quote is now interpreted as a string terminator, and so the attack succeeds.


Some websites make XSS more difficult by restricting which characters you are allowed to use. This can be on the website level or by deploying a WAF that prevents your requests from ever reaching the website. In these situations, you need to experiment with other ways of calling functions which bypass these security measures. One way of doing this is to use the `throw` statement with an exception handler. This enables you to pass arguments to a function without using parentheses. The following code assigns the `alert()` function to the global exception handler and the `throw` statement passes the `1` to the exception handler (in this case `alert`). The end result is that the `alert()` function is called with `1` as an argument.

`onerror=alert;throw 1`

There are multiple ways of using this technique to call [functions without parentheses](https://portswigger.net/research/xss-without-parentheses-and-semi-colons).

The next lab demonstrates a website that filters certain characters. You'll have to use similar techniques to those described above in order to solve it.


### Making use of HTML-encoding

When the XSS context is some existing JavaScript within a quoted tag attribute, such as an event handler, it is possible to make use of HTML-encoding to work around some input filters.

When the browser has parsed out the HTML tags and attributes within a response, it will perform HTML-decoding of tag attribute values before they are processed any further. If the server-side application blocks or sanitizes certain characters that are needed for a successful XSS exploit, you can often bypass the input validation by HTML-encoding those characters.

For example, if the XSS context is as follows:

`<a href="#" onclick="... var input='controllable data here'; ...">`

and the application blocks or escapes single quote characters, you can use the following payload to break out of the JavaScript string and execute your own script:

`&apos;-alert(document.domain)-&apos;`

The `&apos;` sequence is an HTML entity representing an apostrophe or single quote. Because the browser HTML-decodes the value of the `onclick` attribute before the JavaScript is interpreted, the entities are decoded as quotes, which become string delimiters, and so the attack succeeds.


### XSS in JavaScript template literals

JavaScript template literals are string literals that allow embedded JavaScript expressions. The embedded expressions are evaluated and are normally concatenated into the surrounding text. Template literals are encapsulated in backticks instead of normal quotation marks, and embedded expressions are identified using the `${...}` syntax.

For example, the following script will print a welcome message that includes the user's display name:

``document.getElementById('message').innerText = `Welcome, ${user.displayName}.`;``

When the XSS context is into a JavaScript template literal, there is no need to terminate the literal. Instead, you simply need to use the `${...}` syntax to embed a JavaScript expression that will be executed when the literal is processed. For example, if the XSS context is as follows:

``<script> ... var input = `controllable data here`; ... </script>``

then you can use the following payload to execute JavaScript without terminating the template literal:

`${alert(document.domain)}`


## XSS via client-side template injection

Some websites use a client-side template framework, such as AngularJS, to dynamically render web pages. If they embed user input into these templates in an unsafe manner, an attacker may be able to inject their own malicious template expressions that launch an XSS attack.

#### Read more

- [Client-side template injection](https://portswigger.net/web-security/cross-site-scripting/contexts/client-side-template-injection)

# Client-side template injection

In this section, we'll look at client-side template injection vulnerabilities and how you can exploit them for XSS attacks. This attack technique was pioneered by our research team - read more in [XSS without HTML: Client-Side Template Injection with AngularJS](https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs). Although client-side template injection is a generic issue, we'll focus on examples from the AngularJS framework as this is the most common. We'll describe how you can craft exploits that escape from the AngularJS sandbox, and how you can potentially use AngularJS features to bypass content security policy (CSP).

## What is client-side template injection?

Client-side template injection vulnerabilities arise when applications using a client-side template framework dynamically embed user input in web pages. When rendering a page, the framework scans it for template expressions and executes any that it encounters. An attacker can exploit this by supplying a malicious template expression that launches a cross-site scripting (XSS) attack.

## What is the AngularJS sandbox?

The AngularJS sandbox is a mechanism that prevents access to potentially dangerous objects, such as `window` or `document`, in AngularJS template expressions. It also prevents access to potentially dangerous properties, such as `__proto__`. Despite not being considered a security boundary by the AngularJS team, the wider developer community generally thinks otherwise. Although bypassing the sandbox was initially challenging, security researchers have discovered numerous ways of doing so. As a result, it was eventually removed from AngularJS in version 1.6. However, many legacy applications still use older versions of AngularJS and may be vulnerable as a result.

## How does the AngularJS sandbox work?

The sandbox works by parsing an expression, rewriting the JavaScript, and then using various functions to test whether the rewritten code contains any dangerous objects. For example, the `ensureSafeObject()` function checks whether a given object references itself. This is one way to detect the `window` object, for example. The `Function` constructor is detected in roughly the same way, by checking whether the constructor property references itself.

The `ensureSafeMemberName()` function checks each property access of the object and, if it contains dangerous properties such as `__proto__` or `__lookupGetter__`, the object will be blocked. The `ensureSafeFunction()`function prevents `call()`, `apply()`, `bind()`, or `constructor()` from being called.

You can see the sandbox in action for yourself by visiting [this fiddle](http://jsfiddle.net/2zs2yv7o/1/) and setting a breakpoint at line 13275 of the `angular.js` file. The variable `fnString` contains your rewritten code, so you can look at how AngularJS transforms it.

## How does an AngularJS sandbox escape work?

A sandbox escape involves tricking the sandbox into thinking the malicious expression is benign. The most well-known escape uses the modified `charAt()` function globally within an expression:

`'a'.constructor.prototype.charAt=[].join`

When it was initially discovered, AngularJS did not prevent this modification. The attack works by overwriting the function using the `[].join` method, which causes the `charAt()` function to return all the characters sent to it, rather than a specific single character. Due to the logic of the `isIdent()` function in AngularJS, it compares what it thinks is a single character against multiple characters. As the first character of the longer string are within the range of the compared characters, the `isIdent()` function always returns true, as demonstrated by the following example:

`isIdent = function(ch) { return ('a' <= ch && ch <= 'z' || 'A' <= ch && ch <= 'Z' || '_' === ch || ch === '$'); } isIdent('x9=9a9l9e9r9t9(919)')`

Once the `isIdent()` function is fooled, you can inject malicious JavaScript. For example, an expression such as `$eval('x=alert(1)')` would be allowed because AngularJS treats every character as an identifier. Note that we need to use AngularJS's `$eval()` function because overwriting the `charAt()` function will only take effect once the sandboxed code is executed. This technique would then bypass the sandbox and allow arbitrary JavaScript execution. PortSwigger Research [broke the AngularJS sandbox comprehensively, multiple times](https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs).

### Constructing an advanced AngularJS sandbox escape

So you've learned how a basic sandbox escape works, but you may encounter sites that are more restrictive with which characters they allow. For example, a site may prevent you from using double or single quotes. In this situation, you need to use functions such as `String.fromCharCode()` to generate your characters. Although AngularJS prevents access to the `String` constructor within an expression, you can get round this by using the constructor property of a string instead. This obviously requires a string, so to construct an attack like this, you would need to find a way of creating a string without using single or double quotes.

In a standard sandbox escape, you would use `$eval()` to execute your JavaScript payload, but in the lab below, the `$eval()` function is undefined. Fortunately, we can use the `orderBy` filter instead. The typical syntax of an `orderBy` filter is as follows:

`[123]|orderBy:'Some string'`

Note that the `|` operator has a different meaning than in JavaScript. Normally, this is a bitwise `OR` operation, but in AngularJS it indicates a filter operation. In the code above, we are sending the array `[123]` on the left to the `orderBy` filter on the right. The colon signifies an argument to send to the filter, which in this case is a string. The `orderBy` filter is normally used to sort an object, but it also accepts an expression, which means we can use it to pass a payload.

You should now have all the tools you need to tackle the next lab.


## How does an AngularJS CSP bypass work?

Content security policy (CSP) bypasses work in a similar way to standard sandbox escapes, but usually involve some HTML injection. When the CSP mode is active in AngularJS, it parses template expressions differently and avoids using the `Function` constructor. This means the standard sandbox escape described above will no longer work.

Depending on the specific policy, the CSP will block JavaScript events. However, AngularJS defines its own events that can be used instead. When inside an event, AngularJS defines a special `$event` object, which simply references the browser event object. You can use this object to perform a CSP bypass. On Chrome, there is a special property on the `$event/event` object called `path`. This property contains an array of objects that causes the event to be executed. The last property is always the `window` object, which we can use to perform a sandbox escape. By passing this array to the `orderBy` filter, we can enumerate the array and use the last element (the `window` object) to execute a global function, such as `alert()`. The following code demonstrates this:

`<input autofocus ng-focus="$event.path|orderBy:'[].constructor.from([1],alert)'">`

Notice that the `from()` function is used, which allows you to convert an object to an array and call a given function (specified in the second argument) on every element of that array. In this case, we are calling the `alert()` function. We cannot call the function directly because the AngularJS sandbox would parse the code and detect that the `window` object is being used to call a function. Using the `from()` function instead effectively hides the `window` object from the sandbox, allowing us to inject malicious code.

PortSwigger Research [created a CSP bypass using AngularJS in 56 characters using this technique](https://portswigger.net/research/angularjs-csp-bypass-in-56-characters).

### Bypassing a CSP with an AngularJS sandbox escape

This next lab employs a length restriction, so the above vector will not work. In order to exploit the lab, you need to think of various ways of hiding the `window` object from the AngularJS sandbox. One way of doing this is to use the `array.map()` function as follows:

`[1].map(alert)`

`map()` accepts a function as an argument and will call it for each item in the array. This will bypass the sandbox because the reference to the `alert()` function is being used without explicitly referencing the `window`. To solve the lab, try various ways of executing `alert()` without triggering AngularJS's `window` detection.

## How to prevent client-side template injection vulnerabilities

To prevent client-side template injection vulnerabilities, avoid using untrusted user input to generate templates or expressions. If this is not practical, consider filtering out template expression syntax from user input prior to embedding it within client-side templates.

Note that HTML-encoding is not sufficient to prevent client-side template injection attacks, because frameworks perform an HTML-decode of relevant content prior to locating and executing template expressions.




## How to find and test for reflected XSS vulnerabilities

The vast majority of reflected cross-site scripting vulnerabilities can be found quickly and reliably using Burp Suite's web vulnerability scanner.

Testing for reflected XSS vulnerabilities manually involves the following steps:

- **Test every entry point.** Test separately every entry point for data within the application's HTTP requests. This includes parameters or other data within the URL query string and message body, and the URL file path. It also includes HTTP headers, although XSS-like behavior that can only be triggered via certain HTTP headers may not be exploitable in practice.
- **Submit random alphanumeric values.** For each entry point, submit a unique random value and determine whether the value is reflected in the response. The value should be designed to survive most input validation, so needs to be fairly short and contain only alphanumeric characters. But it needs to be long enough to make accidental matches within the response highly unlikely. A random alphanumeric value of around 8 characters is normally ideal. You can use Burp Intruder's [number payloads](https://portswigger.net/burp/documentation/desktop/tools/intruder/payloads/types#numbers) with randomly generated hex values to generate suitable random values. And you can use Burp Intruder's [grep payloads settings](https://portswigger.net/burp/documentation/desktop/tools/intruder/configure-attack/settings#grep-payloads) to automatically flag responses that contain the submitted value.
- **Determine the reflection context.** For each location within the response where the random value is reflected, determine its context. This might be in text between HTML tags, within a tag attribute which might be quoted, within a JavaScript string, etc.
- **Test a candidate payload.** Based on the context of the reflection, test an initial candidate XSS payload that will trigger JavaScript execution if it is reflected unmodified within the response. The easiest way to test payloads is to send the request to [Burp Repeater](https://portswigger.net/burp/documentation/desktop/tools/repeater), modify the request to insert the candidate payload, issue the request, and then review the response to see if the payload worked. An efficient way to work is to leave the original random value in the request and place the candidate XSS payload before or after it. Then set the random value as the search term in Burp Repeater's response view. Burp will highlight each location where the search term appears, letting you quickly locate the reflection.
- **Test alternative payloads.** If the candidate XSS payload was modified by the application, or blocked altogether, then you will need to test alternative payloads and techniques that might deliver a working XSS attack based on the context of the reflection and the type of input validation that is being performed. For more details, see [cross-site scripting contexts](https://portswigger.net/web-security/cross-site-scripting/contexts)
- **Test the attack in a browser.** Finally, if you succeed in finding a payload that appears to work within Burp Repeater, transfer the attack to a real browser (by pasting the URL into the address bar, or by modifying the request in [Burp Proxy's intercept view](https://portswigger.net/burp/documentation/desktop/tools/proxy/intercept-messages), and see if the injected JavaScript is indeed executed. Often, it is best to execute some simple JavaScript like `alert(document.domain)` which will trigger a visible popup within the browser if the attack succeeds.

## Common questions about reflected cross-site scripting

**What is the difference between reflected XSS and stored XSS?** Reflected XSS arises when an application takes some input from an HTTP request and embeds that input into the immediate response in an unsafe way. With stored XSS, the application instead stores the input and embeds it into a later response in an unsafe way.

**What is the difference between reflected XSS and self-XSS?** Self-XSS involves similar application behavior to regular reflected XSS, however it cannot be triggered in normal ways via a crafted URL or a cross-domain request. Instead, the vulnerability is only triggered if the victim themselves submits the XSS payload from their browser. Delivering a self-XSS attack normally involves socially engineering the victim to paste some attacker-supplied input into their browser. As such, it is normally considered to be a lame, low-impact issue.






---




## Stored cross-site scripting

Stored XSS (also known as persistent or second-order XSS) arises when an application receives data from an untrusted source and includes that data within its later HTTP responses in an unsafe way.

The data in question might be submitted to the application via HTTP requests; for example, comments on a blog post, user nicknames in a chat room, or contact details on a customer order. In other cases, the data might arrive from other untrusted sources; for example, a webmail application displaying messages received over SMTP, a marketing application displaying social media posts, or a network monitoring application displaying packet data from network traffic.

Here is a simple example of a stored XSS vulnerability. A message board application lets users submit messages, which are displayed to other users:

`<p>Hello, this is my message!</p>`

The application doesn't perform any other processing of the data, so an attacker can easily send a message that attacks other users:

`<p><script>/* Bad stuff here... */</script></p>`

#### Read more

- [Stored cross-site scripting](https://portswigger.net/web-security/cross-site-scripting/stored)
- [Cross-site scripting cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)






