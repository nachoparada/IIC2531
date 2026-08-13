---
marp: true
theme: default
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
style: |
  section {
    font-size: 28px;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: flex-start;
    padding-top: 50px;
  }

  ul ul li, ol ol li {
    color: #666666;
    font-size: 0.9em;
  }

  ul ul, ol ol {
    opacity: 0.8;
  }

  pre {
    font-size: 0.9em;
    width: 78%;
    margin-left: 70px;
  }

  pre code {
    padding: 18px 22px;
  }

  section.tweet-slide {
    padding-top: 34px;
    align-items: center;
  }

  section.tweet-slide h1 {
    align-self: flex-start;
    font-size: 1.55em;
    margin-bottom: 10px;
  }

  .tweet-shot {
    width: 620px;
    height: 520px;
    border: 1px solid #d8dee4;
    border-radius: 12px;
    background-image: url('./assets/query-x-post-crop.png');
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center top;
    background-color: #ffffff;
    box-shadow: 0 10px 26px rgba(31, 35, 40, 0.16);
  }

  .tweet-link {
    width: 620px;
    margin-bottom: 12px;
    font-size: 0.72em;
  }

---

<!-- _class: tweet-slide -->

# Motivación: un nuevo método HTTP

<div class="tweet-link">
<a href="https://x.com/i/status/2087226776523092390">Fuente</a>
</div>

<div class="tweet-shot"></div>

---

# ¿Qué son los métodos HTTP?

* En HTTP, el método dice qué quiere hacer el cliente con un recurso
  * `GET`: obtener una representación
  * `POST`: enviar datos para que el servidor procese algo
  * `PUT`, `PATCH`, `DELETE`: modificar o eliminar estado
* Tienen distintas propiedades:
  * Pueden ser idempotentes, solo de lectura, etc.


---

# El problema que resuelve `QUERY`

* Muchas APIs hacen búsquedas complejas con:

```http
POST /feed HTTP/1.1
Content-Type: application/json

{"q":"foo","limit":10,"sort":"-published"}
```

* Eso funciona, pero `POST` no dice si la operación es solo lectura
* `GET` sí es seguro, pero pone filtros en la URL
  * URLs largas
  * Codificación incómoda
  * Más exposición en logs, historial, bookmarks, analytics

---

# `QUERY`: búsqueda con body, pero segura

```http
QUERY /feed HTTP/1.1
Content-Type: application/json

{"q":"foo","limit":10,"sort":"-published"}
```

* Definido por RFC 10008, publicado en junio de 2026
* El body describe la consulta
* La operación debe ser **solo lectura** e **idempotente**
  * Se puede reintentar sin miedo a cambios parciales de estado
* **Mejora privacidad relativa: menos datos sensibles en URLs**
