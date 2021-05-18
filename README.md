# AJAX

Este repositorio tiene por propósito de ser una introducción teórica y práctica al concepto de AJAX.

## Introducción

AJAX es el acrónimo de "Asynchronous JavaScript And XML" y hace referencia a un conjunto de tecnologías y técnicas que permiten principalmente mejorar o enriquecer la experiencia de un usuario en una aplicación web. Recordemos que la web en sus inicios surgió como un mecanismo de intercambio de información que fue evolucionando progresivamente hasta lo que conocemos hoy en día, donde muchas aplicaciones web tienen iguales características o funcionalidades que sus equivalentes aplicaciones de escritorio. O incluso, muchas de las aplicaciones que usamos hoy en día en el escritorio son en realidad aplicaciones web. AJAX contribuyó bastante en este sentido. Sin embargo, en el camino, existieron otras tecnologías como Flash o Silverlight, hoy en desuso, que aportaron en el mismo sentido.

Sin el uso de AJAX, podemos decir que la comunicación del cliente con el servidor web es **sincrónica**. Esto implica que el usuario al realizar una acción determinada que implica una petición al servidor web, debe esperar que la misma se complete, para poder seguir interactuando con la aplicación. El siguiente diagrama ilustra dicha situación

![Ejemplo de interacción con peticiones sincrónicas](./images/sync.png)

Detengamonos a pensar por un momento si muchas de las aplicaciones web que usamos hoy en día serían víables haciendo uso solamente de este esquema. Rápidamente nos damos cuenta que no sería posible. Pensemos en un ejemplo en concreto. Al ingresar a Google para realizar una búsqueda, cuando ingresamos un término, la interfaz nos despliega una serie de sugerencias de términos que coinciden con lo ingresado. Dicha lista, es generada en el servidor y es obtenida sin refresca la página. Esta comunicación con el servidor web fue realizada en segundo plano por el navegador y, como mencionamos anteriormente, no implicó que la página se refresque. Evidentemente, la aplicación en algún punto está manejando los eventos que se producen sobre el cuadro de búsqueda y generando una petición al servidor para obtener sugerencias de búsqueda. Este comportamiento debe estar implementado en JavaScript y, si nos ponemos a pensar en más detalle, implica que como parte de la ejecución del código estamos generando una petición HTTP que al ser respondida implicará una determinada modificación sobre el sitio. Ahora bien, también sabemos que mientras que esta petición está en curso, podemos seguir interactuando con el sitio. Incluso si abrimos el panel de red de las herramientas de desarrollador de nuestro navegador y seguimos escribiendo, vemos como se disparan nuevas peticiones incluso antes de obtener la respuesta a algunas previas. Esto implica que podemos seguir ejecutando código JavaScript cuando hay peticiones en curso; dicho en otras palabras, las peticiones fueron generadas de manera asincrónica. Justamente, **hablamos de JavaScript asincrónico porque la página no se refresca y podemos seguir ejecutando código luego de realizar una petición**. La siguiente imagen ilustra dicha situación:

![Ejemplo de interacción con peticiones asincrónicas](./images/async.png)

Ahora bien, la siguiente pregunta que seguramente nos haremos, es qué diferencia hay entre una petición sincrónica y asincrónica. Desde el punto de vista del protocolo HTTP no hay diferencias. En otras palabras, no importa si el flujo de la aplicación queda bloqueado a la espera de la respuesta o continúa para el protocolo. Ahora bien, desde el punto de vista de la información que se transmite tanto en la petición como en la respuesta, es más interesante. Si seguimos con nuestro ejemplo, la petición generada al ingresar el término de búsqueda `unlu` es:

```
GET /complete/search?q=unlu HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:88.0) Gecko/20100101 Firefox/88.0
Accept: */*
Accept-Language: es-AR,es;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Referer: https://www.google.com/
Connection: keep-alive
Cookie: ...
```

> Algunos detalles de la petición fueron omitidos con fines didácticos

Analizando la misma, vemos que no hay diferencias con una petición `GET` de las que hemos visto hasta ahora. Vemos que el término de búsqueda viaja en el _query string_ de la URL. Si analizamos la respuesta, vemos lo siguiente:

```
HTTP/2 200 OK
date: Fri, 30 Apr 2021 20:18:13 GMT
content-type: application/json; charset=UTF-8

[[["unlu\u003cb\u003e chivilcoy\u003c\/b\u003e",0,[199,175,67]],["unlu lujan",46,[433,131],{"zh":"unlu lujan","zi":"Universidad Nacional de Luján \u2014 Universidad pública en Luján, Argentina","zp":{"gs_ssp":"eJzj4tLP1TcwMS_JyC0xYPTiKs3LKVXIKc1KzAMAWcoHuw"},"zs":"https://encrypted-tbn0.gstatic.com/images?q\u003dtbn:ANd9GcQaNHAQ2wlLO80aPzOFYH5XAh2QTWkHX5zppfk9EDbfFHrlCncpkrcZQZQ\u0026s\u003d10"}],["unlu\u003cb\u003e campana\u003c\/b\u003e",0,[199,175]],["unlu\u003cb\u003e aulas virtuales\u003c\/b\u003e",0,[433,131]],["unlu\u003cb\u003e san miguel\u003c\/b\u003e",0,[433,131,199,175]],["unlu\u003cb\u003e estudiantes\u003c\/b\u003e",0],["unlu\u003cb\u003e san fernando\u003c\/b\u003e",0,[199,175]],["unlu\u003cb\u003e calendario academico\u003c\/b\u003e",0],["unlu\u003cb\u003e perfil estudiante\u003c\/b\u003e",0],["unlu\u003cb\u003e san miguel inscripción 2021\u003c\/b\u003e",0]],{"q":"xDn1dZnwRJfTpbrX-XY9jBuH3_8"}]
```

> Algunos detalles de la petición fueron omitidos con fines didácticos

Como vemos, las sugerencias de búsqueda se están enviando estructuradas en un determinado formato. Dicho formato se denomina JSON (JavaScript Object Notation) y más adelante lo profundizaremos. Ahora bien, nada quita la posibilidad de que el servidor pueda responder a una de estas peticiones con documento HTML, una imagen o algún otro tipo de contenido. Tampoco de que la petición haya podido ser realizada empleando algún otro método HTTP como `POST`. Esto abre amplias posibilidades con diversas aplicaciones, como por ejemplo, la carga de parte de la página luego de mostrarle cierto contenido al usuario con el fin de reducir los tiempos de espera, el enviar el contenido de un formulario al servidor sin necesidad de "recargar la página", la carga en segundo plano de imágenes u otro contenido, etc.

## Formatos de intercambio de datos

Como mencionamos anteriormente, la información que recibimos desde el servidor, a priori, puede tener diversos formatos o estar estructurada de diversas formas. La opción más popular hoy en días es JSON (JavaScript Object Notation), aunque también suele utilizarse XML (eXtensible Markup Language). Probablemente te preguntes por qué se emplean estos formatos para estructurar la información, pregunta que tiene varias respuestas. Por un lado, debido que la información recibida normalmente requiere algún tipo de procesamiento, se necesita que pueda ser fácilmente *parseada* (analizada) y manejada desde un lenguaje de programación. Estos formatos intentan solucionar este problema. Por otro lado, nada quita que esta información pueda ser consumida por otra aplicación con capacidades de hacer peticiones HTTP. En consecuencia, utilizar estos mecanismos brinda cierto nivel de abstracción de las tecnologías y arquitecturas que hay detrás del servidor que está generando la respuesta. Por ejemplo, podemos tener una aplicación web implementada en PHP que corre en un servidor Apache sobre Linux que genera un JSON consumido por un sitio web mediante JavaScript y por otro lado, una aplicación Android desarrollada en Java que también lo consume.

También puede surgir la pregunta "yo solo necesito mostrar el contenido que me está retornando el servidor, ¿no sería más directo solo retornar código HTML?". Esto es posible y existen casos de uso donde se requiere. Pero para actualizar un parte del contenido de la página puede tener algunos inconvenientes. En primer lugar, la respuesta del servidor puede estar muy ajustada a una vista o página en particular, lo cual reduce las posibilidades de emplearla en otra parte de la aplicación o sitio. Por otro lado, el servidor tiene que tener cierto conocimiento acerca de cómo es la página donde se mostrará la respuesta, lo cual no siempre es deseable. Por último, también puede tener implicancias desde el punto de vista de la seguridad.

Habiendo discutido por qué utilizamos lenguajes de intercambio de datos, pasaremos a describirlos.

### JSON

Según su [sitio web oficial](https://www.json.org/json-es.html), _"JSON (JavaScript Object Notation - Notación de Objetos de JavaScript) es un formato ligero de intercambio de datos. Leerlo y escribirlo es simple para humanos, mientras que para las máquinas es simple interpretarlo y generarlo. Está basado en un subconjunto del Lenguaje de Programación JavaScript, Standard ECMA-262 3rd Edition - Diciembre 1999. JSON es un formato de texto que es completamente independiente del lenguaje pero utiliza convenciones que son ampliamente conocidas por los programadores de la familia de lenguajes C, incluyendo C, C++, C#, Java, JavaScript, Perl, Python, y muchos otros. Estas propiedades hacen que JSON sea un lenguaje ideal para el intercambio de datos"._ Veamos un ejemplo de un documento JSON:

```json
{
    "id": 44878,
    "nombre": "Kind of Blue",
    "artistas":[
        {
            "nombre": "Mile Davis",
            "fecha_nacimento": "1926-05-26",
            "fecha_muerte": "1991-09-28"
        }
    ],
    "generos": [ "Jazz" ],
    "canciones" : [
        {
            "nombre": "So What",
            "duracion": 545
        },
        {
            "nombre": "Freddie Freeloader",
            "duracion": 576
        },        
        {
            "nombre": "Blue in Green",
            "duracion": 329
        },
        {
            "nombre": "All Blues",
            "duracion": 693
        },        
        {
            "nombre": "Flamenco Sketches",
            "duracion": 566
        },          
    ],
    "fecha_ultima_reproduccion": null
}
```

Como vemos, un documento JSON, tiene una sintaxis bastante parecida a la de JavaScript (no es un subconjunto en sentido estricto como consecuencia de ciertos caracteres Unicode válidos en JSON pero no en JS). En consecuencia, podría implementarse un *parseador* en JavaScript con la función [`eval()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/eval), lo cual, está desaconsejado, por cuestiones de seguridad. Más allá, de las cuestiones estrictas del *parseo*, es importante destacar que una de sus ventajas es que, formateado de forma adecuada, puede ser leído sin demasiada dificultad por una persona. Además permite organizar de forma jerárquica la información y es capaz de representar números, *strings*, *booleanos*, valores nulos, arreglos y mapeos de *strings* a valores. Otra ventaja que tiene, es que es "compacto" desde el punto de vista de cómo representa la información. Una desventaja, es que no permite ingresar comentarios.

### XML

XML es un metalenguaje que permite definir lenguajes de marcado para representar información estructurada y es una recomendación de la W3C. Cabe destacar que su uso excede a Internet y tiene otras aplicaciones como las base de datos, los editores de texto, etc. Está complementado por otras tecnologías que lo complementan (como por ejemplo, XPath), muchas de las cuales también han sido definidas por la W3C, lo cual lo hacen muy poderoso. Veamos un ejemplo de un documento XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<album id="44878">
    <nombre>Kind of Blue</nombre>
    <artista>
      <nombre>Miles Davis</nombre>
      <nacimiento>1926-05-26</nacimiento>
      <muerte>1991-09-28</muerte>
    </artista> 
    <generos>
      <genero>Jazz</genero>
    </generos>
    <canciones>
      <cancion>
        <nombre>So What</nombre>
        <duracion>545</duracion>
      </cancion>
      <cancion>
        <nombre>Freddie Freeloader</nombre>
        <duracion>576</duracion>
      </cancion>
      <cancion>
        <nombre>Blue in Green</nombre>
        <duracion>329</duracion>
      </cancion>
      <cancion>
        <nombre>All Blues</nombre>
        <duracion>693</duracion>
      </cancion>
      <cancion>
        <nombre>Flamenco Sketches</nombre>
        <duracion>566</duracion>
      </cancion>
    </canciones>
    <!-- Fecha de la última reproducción de alguna canción del álbum -->
    <fechaUltimaReproduccion></fechaUltimaReproduccion>
</album>
```

Como podemos observar a simple vista, un documento XML es menos compacto que un documento JSON. Sin embargo, su lectura tampoco es complicada para humanos y también permite organizar la información de forma jerárquica. También soporta el uso de comentarios y su estructura puede ser validada haciendo uso de una DTD (Document Type Definition) o de un *XML Schema*, lo cual, por ejemplo, nos permite detectar la presencia de etiquetas que el autor del documento no haya definido. Sin embargo, un documento XML es más difícil de *parsear* que un documento JSON.

## Cómo realizar peticiones AJAX

JavaScript ofrece dos alternativas para efectuar peticiones:

* `XMLHttpRequest`
* Fetch API

Las principales diferencias entre ambas son las siguientes:

* `XMLHttpRequest` es anterior a Fetch API, por lo que es soportada por más navegadores
* `XMLHttpRequest` tiene una API un poco más compleja que Fetch y de un poco más bajo nivel
* Fetch API usa [promesas](https://web.dev/promises/)

A continuación haremos una introducción a cada una de las APIs y veremos cómo hacer una petición asincrónica con cada una.

### XMLHttpRequest

La primera implementación de este objeto aparece en 1999 como un componente [ActiveX](https://es.wikipedia.org/wiki/ActiveX). De ahí se fue evolucionando, y en 2006, se presentó el [primer borrador](https://www.w3.org/TR/2006/WD-XMLHttpRequest-20060405/) de la especificación. 

Básicamente, este API cuenta con el objecto [`XMLHttpRequest`](https://developer.mozilla.org/es/docs/Web/API/XMLHttpRequest) el cual nos permite enviar una petición y genera una serie de eventos en relación al estado de la misma, y en consecuencia, nos permite registrar los correspondientes manejadores de eventos.

Puedes ver un ejemplo de una petición realizada con esta API [aquí](./ejemplo-xhr.html). Recuerda abrir el inspector del navegador y revisar el código fuente.

### Fetch API

Como mencionamos anteriormente, la API fetch está basada en [promesas](https://web.dev/promises/). Una promesa es el resultado eventual de una operación asincrónica. Informalmente, podríamos decir que es un compromiso de un resultado determinado de una operación asincrónica. Dicho compromiso puede ser cumplido (fulfilled), rechazado (rejected) y puede estar pendiente (pending, lo que implica que aún no fue ni cumplido ni rechazado). Las promesas están modeladas por el objecto [`Promise`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Promise), el cual ofrece un método llamado [`then(handlerCumplimiento, handlerRechazo)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) que toma como argumento dos funciones que se ejecutan de forma asincrónica, una la promesa está cumplida y otro cuando está rechazda y que devuelve una nueva promesa. La característica de la misma depende del comportamiento de dichas funciones:

* si la función retorna un valor, la promesa retornada se resuelve con el valor retornado;
* si no retorna nada, la promesa retornada se resuelve con el valor `undefined`;
* si arroja un error, la promesa retornada se rechaza con el error arrojado como valor;
* si retorna una promesa cumplida, la promesa retornada se resuelve con el valor de la promesa cumplida como su valor;
* si retorna una promesa rechazada, la promesa retornada se rechaza con el valor de la promesa rechazada como su valor;
* si retorna una promesa pendiente, la promesa retornada será cumplida o rechazada dependiendo el resultado de la promesa retornada con el valor correspondiente al rechazo o cumplimiento de dicha promesa 

Cabe destacar que `then()` no nos obliga a definir manejadores tanto para el caso del cumplimiento como del rechazo. Uno podría pasar a uno de sus argumentos el valor `undefined`. Otro método interesante de `Promise` es `catch(handlerRechazo)`. Este método es equivalente a llamar `then(undefined, handlerRechazo)`. Esto, nos permite hacer cosas como la siguiente:

```js
promesa.then(function(valor){
  // manejador 1
}).then(function(otroValor){
  // manejador 2
}).catch(function(error){
  // manejador error
})
```

Aquí si promesa se cumple, se ejecutará `manejador 1`, caso contrario `manejador error`. Luego, de acuerdo el comportamiento de `manejador 1`, se retornará una nueva promesa de acuerdo a lo especificado anteriormente, que según si se cumple o no, determinará la ejecución del `manejador 2` o del `manejador error`.

Habiendo descrito brevemente las promesas, pasaremos a ver cómo son utilizadas en la API de *fetch*. La API de *fetch* posee un método denominado `fetch()`, el cual permite realizar la petición HTTP. El resultado de la misma, es una promesa, donde:

* si la petición pudo completarse, la promesa es resuelta con un valor de tipo [`Response`](https://developer.mozilla.org/es/docs/Web/API/Response). Este es un objeto que contiene toda la información de la respuesta, como por ejemplo, el código de estado, el cuerpo, etc. 
* En caso de no poder haberse podido realizar la petición, por ejemplo, por un problema de red, la promesa será rechaza con un [`TypeError`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError) como valor.

Puedes ver un ejemplo de una petición realizada con esta API [aquí](./ejemplo-fetch.html). Recuerda abrir el inspector del navegador y revisar el código fuente.

## Desafíos

1. Para cada uno de los ejemplos, tratá de renderizar cada tarea dentro del código HTML como un `article`. Ayuda: necesitarás *parsear* la respuesta JSON. 
2. Agrega un mensaje que le indique al usuario que una petición está en curso.
3. Agrega un formulario que le permita al usuario dar de alta una tarea (ten en cuenta que la API te responderá como que la ha creado pero en realidad no se persistirá).
4. Seguramente, haz notado que la API siempre retorna JSON. ¿Qué cambios deberíamos hacer en nuestro código en el hipotético caso que esta API pudiera manejar diversos tipos de contenido para asegurarnos de obtener contenido codificado en JSON?

## Bibliografía

* Javascript: The definitive guide. 7ma edición. David Flanagan. Capítulos: 11.6 y 13.
* [AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX). MDN. Última consulta: 15/10/2021.
* [Introduction to fetch()](https://developers.google.com/web/updates/2015/03/introduction-to-fetch). Matt Gaunt. Última consulta: 15/10/2021.
* [Fetch API](https://developer.mozilla.org/es/docs/Web/API/Fetch_API). MDN. Última consulta: 15/10/2021.
* [XMLHttpRequest](https://developer.mozilla.org/es/docs/Web/API/XMLHttpRequest). MDN. Última consulta: 15/10/2021.

