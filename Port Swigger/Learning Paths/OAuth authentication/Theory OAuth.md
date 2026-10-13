
https://portswigger.net/web-security/oauth


---

### La idea central

OAuth nació para compartir acceso a datos, pero acabó usándose para el "Iniciar sesión con Google/Facebook". El problema es que la especificación es **vaga y casi todo es opcional**, incluidas medidas de seguridad importantes. Casi toda la seguridad depende de que el desarrollador configure y valide bien las cosas, y ahí es donde aparecen los fallos.

### Cómo detectar que una web usa OAuth

Si ofrece login con otra cuenta, probablemente lo use. Se confirma interceptando el tráfico con Burp y buscando una petición a `/authorization` con parámetros como `client_id`, `redirect_uri` y `response_type`. También conviene probar los endpoints `/.well-known/openid-configuration` y `/.well-known/oauth-authorization-server`, que suelen devolver un JSON con la configuración y funciones soportadas.

### Vulnerabilidades en la aplicación cliente

**1. Implicit flow mal implementado.** En este flujo el token llega al navegador. Muchas apps luego envían al servidor un POST con el ID de usuario y el token para crear la sesión. Si el servidor no comprueba que el token realmente corresponde a ese usuario, un atacante puede cambiar el ID en la petición y **suplantar a cualquiera**.

**2. Protección CSRF deficiente (parámetro `state`).** Si falta `state`, el atacante puede iniciar un flujo OAuth con su propia cuenta social y engañar a la víctima para que lo complete. Si la web permite vincular cuentas sociales, el atacante acaba **vinculando su perfil a la cuenta de la víctima** y entrando en ella.

### Vulnerabilidades en el servicio OAuth

**3. Filtración de códigos y tokens vía `redirect_uri`.** Es la más famosa. Si el servidor no valida bien a dónde enviar el código o token, el atacante puede hacer que llegue a un dominio suyo y usarlo para entrar como la víctima. Aunque haya lista blanca de URIs, el documento describe formas de eludirla:

- Validaciones que solo comprueban el principio de la cadena, de modo que se pueden añadir rutas o parámetros.
- Diferencias de interpretación de la URL entre componentes, parámetros duplicados o dominios tipo `localhost.dominio-malo.net`.
- Cambiar otros parámetros (como `response_mode`) que alteran cómo se valida la URI.
- Usar _path traversal_ (`../`) para apuntar a otras páginas del dominio permitido.

**4. Usar páginas "proxy" del dominio legítimo.** Si no puedes sacar el token a un dominio externo, buscas otra página del sitio permitido que lo filtre por ti: un _open redirect_, un script que maneje mal los parámetros, un XSS (que además da mucho más tiempo de acceso que robar una cookie) o una inyección HTML que filtre el código por la cabecera `Referer`.

**5. Validación de scope defectuosa.** Un atacante puede intentar ampliar los permisos de un token, ya sea registrando su propia app maliciosa (authorization code) o añadiendo scopes al llamar a `/userinfo` con un token robado (implicit). Funciona si el servidor no compara con el scope aprobado originalmente.

**6. Registro de usuarios sin verificar.** Si el proveedor OAuth permite crear cuentas sin verificar el email, un atacante puede registrarse con el email de la víctima y la app cliente le dejará entrar como ella, porque confía en que los datos del proveedor son correctos.

### Qué te llevas como desarrollador

- Usa siempre `state` (valor impredecible ligado a la sesión).
- Valida `redirect_uri` con **coincidencia exacta** contra una lista blanca, tanto en la autorización como en el intercambio del código.
- Valida en el servidor que el token corresponde al usuario que dice ser.
- Comprueba que el scope solicitado no se pueda ampliar tras la aprobación.
- Verifica los datos del proveedor (por ejemplo, que el email esté verificado).
- Evita el implicit flow cuando sea posible y prefiere authorization code con PKCE.