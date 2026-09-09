Access control is the application of constraints on who or what is authorized to perform actions or access resources. In the context of web applications, access control is dependent on authentication and session management:

- **Authentication** confirms that the user is who they say they are.
- **Session management** identifies which subsequent HTTP requests are being made by that same user.
- **Access control** determines whether the user is allowed to carry out the action that they are attempting to perform.


# Access control security models
https://portswigger.net/web-security/access-control/security-models



### Vertical access controls

Vertical access controls are mechanisms that restrict access to sensitive functionality to specific types of users.

With vertical access controls, different types of users have access to different application functions. For example, an administrator might be able to modify or delete any user's account, while an ordinary user has no access to these actions. Vertical access controls can be more fine-grained implementations of security models designed to enforce business policies such as separation of duties and least privilege.

### Horizontal access controls

Horizontal access controls are mechanisms that restrict access to resources to specific users.

With horizontal access controls, different users have access to a subset of resources of the same type. For example, a banking application will allow a user to view transactions and make payments from their own accounts, but not the accounts of any other user.

### Context-dependent access controls

Context-dependent access controls restrict access to functionality and resources based upon the state of the application or the user's interaction with it.

Context-dependent access controls prevent a user performing actions in the wrong order. For example, a retail website might prevent users from modifying the contents of their shopping cart after they have made payment.




## Examples of broken access controls

Broken access control vulnerabilities exist when a user can access resources or perform actions that they are not supposed to be able to.

### Vertical privilege escalation

If a user can gain access to functionality that they are not permitted to access then this is vertical privilege escalation. For example, if a non-administrative user can gain access to an admin page where they can delete user accounts, then this is vertical privilege escalation.

#### Unprotected functionality

At its most basic, vertical privilege escalation arises where an application does not enforce any protection for sensitive functionality. For example, administrative functions might be linked from an administrator's welcome page but not from a user's welcome page. However, a user might be able to access the administrative functions by browsing to the relevant admin URL.

For example, a website might host sensitive functionality at the following URL:

`https://insecure-website.com/admin`

This might be accessible by any user, not only administrative users who have a link to the functionality in their user interface. In some cases, the administrative URL might be disclosed in other locations, such as the `robots.txt` file:

`https://insecure-website.com/robots.txt`

Even if the URL isn't disclosed anywhere, an attacker may be able to use a wordlist to brute-force the location of the sensitive functionality.


---

El archivo `robots.txt` es uno de los elementos más incomprendidos en el desarrollo web. Su rol principal está en la gestión del tráfico y el SEO, pero su mal uso es una fuente clásica de vulnerabilidades en ciberseguridad.

WSTG - Latest | OWASP Foundation

Para entender su relevancia, hay que partir de una regla de oro: **`robots.txt` es una petición educada, no un mecanismo de seguridad.**

## 1. Relevancia en Programación y Desarrollo Web

En el desarrollo web, el `robots.txt` es un archivo de texto plano ubicado en la raíz de un sitio (ej. `[tusitio.com/robots.txt](https://tusitio.com/robots.txt)`). Utiliza el _Protocolo de Exclusión de Robots_ para comunicarse con los "crawlers" o arañas web (como Googlebot o Bingbot).

Sus funciones principales son:

- **Optimización del SEO:** Evita que los motores de búsqueda indexen contenido duplicado, páginas de prueba o resultados de búsquedas internas.
    
- **Gestión de recursos (Crawl Budget):** Evita que los bots sobrecarguen el servidor rastreando miles de páginas irrelevantes de forma automática.
    
- **Estructura:** Utiliza directivas simples como `User-agent` (a qué bot va dirigido), `Allow` (permitir) y `Disallow` (bloquear).    

## 2. Relevancia en Ciberseguridad y Vulnerabilidades

Aquí es donde las cosas se complican. Muchos desarrolladores novatos asumen erróneamente que bloquear una carpeta en el `robots.txt` la hace invisible. En la práctica, genera una vulnerabilidad conocida como **Information Disclosure (Divulgación de Información)**.

Dado que el archivo es 100% público, cualquiera puede leerlo. Si un desarrollador escribe esto:

```
User-agent: *
Disallow: /admin-panel/
Disallow: /backups-2023/
Disallow: /api/v1/usuarios/
```

En lugar de proteger esas áreas, **le acaba de entregar un mapa del tesoro a los atacantes**.

### El problema con los "Bots Malos"

El protocolo depende enteramente de la buena voluntad del bot. Los rastreadores de Google o Bing respetarán la directiva `Disallow`. Sin embargo, los escáneres de vulnerabilidades, los scrapers abusivos y los atacantes ignorarán el archivo por completo. De hecho, lo primero que hace un atacante o una herramienta de hacking al analizar un objetivo es leer el `robots.txt` para descubrir dónde están escondidas las partes sensibles del sistema.

## Mejores Prácticas (Cómo hacerlo bien)

Si el `robots.txt` no sirve para proteger datos, ¿cómo se debe manejar la seguridad?

1. **No incluyas rutas secretas:** Si una página o directorio es confidencial, no la menciones en el `robots.txt`.
    
2. **Usa etiquetas `noindex`:** Si quieres que una página privada no aparezca en Google, pero no quieres revelar su ruta en el `robots.txt`, usa la etiqueta HTML `<meta name="robots" content="noindex">` o el encabezado HTTP `X-Robots-Tag` en esa página específica. El buscador no la indexará y los atacantes no verán la ruta publicada.
    
3. **Seguridad real (Autenticación):** La única forma de proteger un directorio es mediante controles de acceso reales: contraseñas, tokens de sesión, firewalls de aplicaciones web (WAF) y configuraciones correctas de permisos en el servidor.
    
4. **Security.txt:** En ciberseguridad moderna, existe otro archivo llamado `security.txt` (RFC 9116) que las empresas colocan en sus servidores para indicar a los hackers éticos cómo y dónde reportar vulnerabilidades de forma segura.

---



In some cases, sensitive functionality is concealed by giving it a less predictable URL. This is an example of so-called "security by obscurity". However, hiding sensitive functionality does not provide effective access control because users might discover the obfuscated URL in a number of ways.

Imagine an application that hosts administrative functions at the following URL:

`https://insecure-website.com/administrator-panel-yb556`

This might not be directly guessable by an attacker. However, the application might still leak the URL to users. The URL might be disclosed in JavaScript that constructs the user interface based on the user's role:

`<script> var isAdmin = false; if (isAdmin) { ... var adminPanelTag = document.createElement('a'); adminPanelTag.setAttribute('href', 'https://insecure-website.com/administrator-panel-yb556'); adminPanelTag.innerText = 'Admin panel'; ... } </script>`

This script adds a link to the user's UI if they are an admin user. However, the script containing the URL is visible to all users regardless of their role.

Mira el código fuente de la página (ctrl+u), a veces hay tesoros.



#### Parameter-based access control methods

Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location. This could be:

- A hidden field.
- A cookie.
- A preset query string parameter.

The application makes access control decisions based on the submitted value. For example:

`https://insecure-website.com/login/home.jsp?admin=true https://insecure-website.com/login/home.jsp?role=1`

This approach is insecure because a user can modify the value and access functionality they're not authorized to, such as administrative functions.


Las cookies pueden no estar bien protegidas, míralo siempre en el fn+F12

A veces puede ser un parámetro devuelto en el request body del JSON, y puedes incluirlo en la request como parámetro en el body (entre los corchetes {})





#### Broken access control resulting from platform misconfiguration

Some applications enforce access controls at the platform layer. they do this by restricting access to specific URLs and HTTP methods based on the user's role. For example, an application might configure a rule as follows:

`DENY: POST, /admin/deleteUser, managers`

This rule denies access to the `POST` method on the URL `/admin/deleteUser`, for users in the managers group. Various things can go wrong in this situation, leading to access control bypasses.

Some application frameworks support various non-standard HTTP headers that can be used to override the URL in the original request, such as `X-Original-URL` and `X-Rewrite-URL`. If a website uses rigorous front-end controls to restrict access based on the URL, but the application allows the URL to be overridden via a request header, then it might be possible to bypass the access controls using a request like the following:

`POST / HTTP/1.1 X-Original-URL: /admin/deleteUser ...`



An alternative attack relates to the HTTP method used in the request. The front-end controls described in the previous sections restrict access based on the URL and HTTP method. Some websites tolerate different HTTP request methods when performing an action. If an attacker can use the `GET` (or another) method to perform actions on a restricted URL, they can bypass the access control that is implemented at the platform layer.



#### Broken access control resulting from URL-matching discrepancies

Websites can vary in how strictly they match the path of an incoming request to a defined endpoint. For example, they may tolerate inconsistent capitalization, so a request to `/ADMIN/DELETEUSER` may still be mapped to the `/admin/deleteUser` endpoint. If the access control mechanism is less tolerant, it may treat these as two different endpoints and fail to enforce the correct restrictions as a result.

Similar discrepancies can arise if developers using the Spring framework have enabled the `useSuffixPatternMatch` option. This allows paths with an arbitrary file extension to be mapped to an equivalent endpoint with no file extension. In other words, a request to `/admin/deleteUser.anything` would still match the `/admin/deleteUser` pattern. Prior to Spring 5.3, this option is enabled by default.

On other systems, you may encounter discrepancies in whether `/admin/deleteUser` and `/admin/deleteUser/` are treated as distinct endpoints. In this case, you may be able to bypass access controls by appending a trailing slash to the path.







### Horizontal privilege escalation

Horizontal privilege escalation occurs if a user is able to gain access to resources belonging to another user, instead of their own resources of that type. For example, if an employee can access the records of other employees as well as their own, then this is horizontal privilege escalation.

Horizontal privilege escalation attacks may use similar types of exploit methods to vertical privilege escalation. For example, a user might access their own account page using the following URL:

`https://insecure-website.com/myaccount?id=123`

If an attacker modifies the `id` parameter value to that of another user, they might gain access to another user's account page, and the associated data and functions.

#### Note

This is an example of an insecure direct object reference (IDOR) vulnerability. This type of vulnerability arises where user-controller parameter values are used to access resources or functions directly.





