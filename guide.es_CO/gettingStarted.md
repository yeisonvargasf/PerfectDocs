## Empezando

¿Está impaciente para empezar a programar con Swift y Perfect? Esta guía le proporcionará todo lo que necesite saber para ejecutar Perfect, y para crear su primera aplicación. 

Después de leer esta guía, sabrás:

- Cómo crear y ejecutar un servidor HTTP/HTTPS, organizar y ejecutar Perfect
- Los componentes prerrequisitos que debe instalar  para ejecutar Perfect ya sea en OS X o Linux Ubuntu
- Cómo construir, probar y administrar dependencias para proyectos Swift
- Cómo desplegar Perfect en entornos adicionales incluyendo Heroku, Amazon Web Services, Docker, Microsoft Azure, Google Cloud, IBM Bluemix CloudFoundry, y IBM Bluemix Docker

### Swift 3.0

Después de haber instalado un [toolchain](https://es.wikipedia.org/wiki/Cadena_de_herramientas) desde [Swift.org](https://swift.org/getting-started/), abra una ventana de terminal y escriba
```
swift --version
```

Este comando producirá un mensaje similar a este: 

```
Apple Swift version 3.0 (swiftlang-800.0.33.1 clang-800.0.31)
Target: x86_64-apple-macosx10.9
```
Asegúrese que está ejecutando la última versión de Swift 3.0, Perfect no compilará exisotosamente si usted está ejecutando una versión de Swift más baja que 3.0.

### Linux Ubuntu
Perfect funciona en los entornos Linux Ubuntu 14.04, 15.10 y 16.04. Perfect confía en OpenSSL, libssl-dev, y uuid-dev. Para instalarlos, en la terminal, escriba:

```
sudo apt-get install openssl libssl-dev uuid-dev
```

###Empezando con Perfect

Ahora está listo para construir su primera aplicación web proyecto inicial. 

### Construir Proyecto Inicial

Lo siguiente clonará y construirá un proyecto inicial vacío.  Esto ejecutará un servidor local que escuchará por el puerto 8181 en su computador:

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
swift build
.build/debug/PerfectTemplate
```

Debebería ver la siguiente salida:

```
Starting HTTP server on 0.0.0.0:8181 with document root ./webroot
```

Ahora el servidor está ejecutando y esperando por conexiones. Entre a [http://localhost:8181/](http://127.0.0.1:8181/) para ver el saludo. Presione "control-c" para terminar el servidor.

Puede ver el código fuente completo en [PerfectTemplate](https://github.com/PerfectlySoft/PerfectTemplate). 

### Xcode

El Administrador De Paquetes De Swift (SPM) puede generar un proyecto de Xcode el cuál puede ejecutar el servidor de PerfectTemplate y proporcionar la edición y depuración del código fuente para su proyecto. Digite lo siguiente en su terminal:

```
swift package generate-xcodeproj
```

Abra el archivo generado "PerfectTemplate.xcodeproj". Asegúrese que ha seleccionado el objetivo ejecutable y que lo ha seleccionado para ejecutar en "Mi Mac". Ahora puede ejecutar y depurar el servidor.

## Siguientes Pasos

Estos pequeños ejemplos de código muestran como lograr muchas de las tareas comunes que puede necesitar hacer cuando desarrolla una web o aplicación REST. En todos los casos, las variables ```request``` y ```response``` refieren, respectivamente, a los objetos ```HTTPRequest``` y ```HTTPResponse``` los cuales son dados para sus manejadores de URL.

Consulte la [referencia del API](http://www.perfect.org/docs/) para más detalles.

### Obtener una Cabecera de la Petición del Cliente

```swift
if let acceptEncoding = request.header(.acceptEncoding) {
	...
}
```

### Parámetros GET y POST del Cliente

```swift
if let foo = request.param(name: "foo") {
	...
}   
if let foo = request.param(name: "foo", defaultValue: "default foo") {
	...
}
let foos: [String] = request.params(named: "foo")
```

### Obtener la Ruta Actual de la Petición

```swift
let path = request.path
```

### Entrando al Directorio Raíz de Documento y Retornando un Archivo de Imagen al Cliente

```swift
let docRoot = request.documentRoot
do {
    let mrPebbles = File("\(docRoot)/mr_pebbles.jpg")
    let imageSize = mrPebbles.size
    let imageBytes = try mrPebbles.readSomeBytes(count: imageSize)
    response.setHeader(.contentType, value: MimeType.forExtension("jpg"))
    response.setHeader(.contentLength, value: "\(imageBytes.count)")
    response.setBody(bytes: imageBytes)
} catch {
    response.status = .internalServerError
    response.setBody(string: "Error handling request: \(error)")
}
response.completed()
```

### Obteniedo las Cookies del Cliente

```swift
for (cookieName, cookieValue) in request.cookies {
	...
}
```

### Configurando las Cookies del Cliente

```swift
let cookie = HTTPCookie(name: "cookie-name", value: "the value", domain: nil,
                    expires: .session, path: "/",
                    secure: false, httpOnly: false)
response.addCookie(cookie)
```

### Retornando Datos JSON al Cliente

```swift
response.setHeader(.contentType, value: "application/json")
let d: [String:Any] = ["a":1, "b":0.1, "c": true, "d":[2, 4, 5, 7, 8]]
    
do {
    try response.setBody(json: d)
} catch {
    //...
}
response.completed()
```
*Esta pieza de código usa la codificación incorporada de JSON. No dude en usar el codificador/decodificador de JSON que usted prefiera.*

### Redirigiendo el Cliente

```swift
response.status = .movedPermanently
response.setHeader(.location, value: "http://www.perfect.org/")
response.completed()
```

### Filtrando y Manejando Errores 404 de Forma Personalizada

```swift
struct Filter404: HTTPResponseFilter {
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}
	
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		if case .notFound = response.status {
			response.bodyBytes.removeAll()
			response.setBody(string: "The file \(response.request.path) was not found.")
			response.setHeader(.contentLength, value: "\(response.bodyBytes.count)")
			callback(.done)
		} else {
			callback(.continue)
		}
	}
}

try HTTPServer(documentRoot: webRoot)
	.setResponseFilters([(Filter404(), .high)])
	.start(port: 8181)
```
