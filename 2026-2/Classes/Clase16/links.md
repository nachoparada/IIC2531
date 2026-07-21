# Seguridad Web

La seguridad web es un tema amplio y en evolución. En esta clase, dividimos la seguridad web en tres partes: seguridad del lado del servidor, seguridad del lado del cliente (preocupándose por cómo aislar muchos sitios y aplicaciones web ejecutándose en el navegador del usuario), y seguridad de red.

## Mecanismos básicos de seguridad del lado del cliente

- [Lo básico del Same-Origin Policy](https://web.dev/same-origin-policy/)
- [Una explicación detallada del Same-Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [Cookies HTTP y su seguridad](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [Visión general de Cross-Origin Resource Sharing](https://web.dev/cross-origin-resource-sharing/)

## Vulnerabilidades comunes y mecanismos de mitigación

- [Ataques Cross-site scripting (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting)
- [Ataques Clickjacking](https://en.wikipedia.org/wiki/Clickjacking)
- [Ataques Cross-site request forgery (CSRF)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/CSRF)
- [Content-Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) — mecanismo del navegador que ayuda a mitigar vulnerabilidades XSS y clickjacking

Se pueden ver similitudes entre XSS en aplicaciones web y buffer overflows en código C del lado del servidor. Ambos son errores simples fáciles de cometer, que llevan a problemas de seguridad significativos al permitir que un adversario ejecute código arbitrario.

## Recursos adicionales

- [Blueprint de Google para un Framework Web de Alta Seguridad](https://bughunters.google.com/blog/secure-by-design-googles-blueprint-for-a-high-assurance-web-framework#key-security-controls) — resumen de los diferentes mecanismos de control de seguridad en la web
- [The Tangled Web](https://lcamtuf.coredump.cx/tangled/) por Michal Zalewski — libro detallado sobre seguridad web (capítulos 9-13 son los más relevantes)
