# Taller: Formularios con validaciones

**Estudiantes:** Valerie Moreno Castañeda y Martin Taborda Aristizabal  
**Curso:** Frontend — Universidad Autónoma de Manizales

## Descripción

En este proyecto hicimos un formulario de registro para practicar validaciones con HTML y JavaScript, mensajes de error, envío de datos con `fetch`, campos dinámicos y algunas medidas de seguridad y accesibilidad. Hay dos versiones: una con validación nativa del navegador y otra que combina HTML, CSS y JavaScript.

## Estructura real del proyecto


Fromularios/
├── nativo/
│   └── index.html
├── registro/
│   ├── index.html
│   ├── estilos.css
│   ├── validaciones.js
│   └── app.js
└── Readme.md

## Cómo ejecutar el proyecto

Abrimos la carpeta en Visual Studio Code y usamos una extensión como **Live Server** para abrir `registro/index.html`. Se recomienda usar un servidor local porque `app.js` importa las funciones de `validaciones.js` como módulo de JavaScript. Para ver la versión de HTML con validaciones nativas, abrimos `nativo/index.html`.

El envío usa `https://jsonplaceholder.typicode.com/posts`, un servicio de pruebas. Necesita conexión a Internet y **no crea usuarios ni almacena cuentas reales**.

## Nivel 1. Formulario con HTML y validación nativa

En `nativo/index.html` construimos un formulario con nombre completo, correo, contraseña, confirmación, edad, país y aceptación de términos. Usamos `required`, `minlength`, `maxlength`, `type="email"`, `min`, `max`, `step` y `pattern` según el campo.

**¿Qué significa la expresión regular de la contraseña?**


(?=.*[A-Za-z])(?=.*\d).{8,}


- `(?=.*[A-Za-z])`: exige por lo menos una letra.
- `(?=.*\d)`: exige por lo menos un número.
- `.{8,}`: exige mínimo ocho caracteres.

En el formulario nativo HTML verifica el patrón, pero **no comprueba que la confirmación sea igual a la contraseña**. Eso sí se valida con JavaScript en la otra versión.

**¿Qué sucede si quitamos el atributo `name`?**

El campo puede seguir apareciendo en pantalla y ser validado, pero su valor no se incluye en los datos normales enviados por el formulario ni en `FormData`. El `id` identifica el elemento y lo asocia con su etiqueta, mientras que `name` identifica el dato que se envía.

## Nivel 2. Estilos y mensajes de error

En `registro/index.html` agregamos elementos `<p>` para los mensajes debajo de cada campo. En `app.js`, al salir de un campo (`focusout`) o al intentar enviar el formulario, se ejecutan las validaciones y se actualizan esos mensajes.

**¿Qué diferencia hay entre `:invalid` y `:user-invalid`?**

`:invalid` selecciona un control que no cumple sus restricciones, incluso antes de que el usuario interactúe con él. `:user-invalid` permite aplicar estilos cuando el navegador considera que el usuario ya ha interactuado con el control. Para evitar errores visuales antes de escribir, normalmente resulta más cómodo mostrar mensajes después de la interacción.

**¿Cuál usamos realmente?**

Aunque estudiamos esos dos pseudoselectores, el CSS entregado **no usa ninguno**. El proyecto marca los errores mediante el atributo `aria-invalid="true"` y este selector en `estilos.css`:


[aria-invalid="true"] {
  border: 2px solid #c0392b;
}


Los mensajes se escriben con `textContent` dentro de los párrafos de error. El formulario avanzado lleva `novalidate`, por lo que las comprobaciones al enviarlo son manejadas por nuestro JavaScript y no por los avisos nativos del navegador.

## Nivel 3. Envío con `FormData` y `fetch`

**¿Por qué usamos `submit` en el formulario y no `click` en el botón?**

Porque el formulario también puede enviarse con la tecla Enter. El evento `submit` permite controlar ambas maneras. Usamos `event.preventDefault()` para evitar el envío tradicional y la recarga de la página.

La función `leerDatos()` obtiene los valores con `new FormData(form)` y los pasa a un objeto con `Object.fromEntries()`. Como pueden existir varios teléfonos con el mismo `name="telefono"`, se recuperan con `fd.getAll('telefono')`.

Si no hay errores, `app.js` deshabilita temporalmente el botón, cambia su texto a «Enviando…» y realiza una petición `POST` a JSONPlaceholder. El cuerpo se envía como JSON usando `JSON.stringify()`. Antes de enviarlo, el código elimina el campo `confirmar`, pero **sí incluye `password` en los datos enviados al servicio de prueba**. Por eso se deben utilizar únicamente datos ficticios, nunca contraseñas reales.

**¿Qué pasa si la URL no existe o falla la conexión?**

Si el servidor responde con un estado HTTP incorrecto, `res.ok` será `false` y nuestro código genera un error. Si se pierde la conexión, `fetch` también puede fallar. En ambos casos aparece el mensaje «No se pudo enviar. Revisa tu conexión e inténtalo de nuevo.» y el botón vuelve a activarse mediante `finally`. Los datos escritos permanecen en el formulario cuando ocurre un error de envío.

Si JSONPlaceholder responde correctamente, se muestra un mensaje con el identificador devuelto por el servicio y el saludo «Hola, [nombre]». Después se reinicia el formulario y se eliminan los teléfonos agregados. **Este mensaje de éxito significa que respondió el servicio de prueba, no que se registró una cuenta de verdad.**

## Nivel 4. Validaciones avanzadas con JavaScript

### Parte A. Funciones de validación

Las validaciones están separadas en `registro/validaciones.js`. Incluyen campos obligatorios, longitud del texto, formato de correo, contraseña con letra y número, confirmación de contraseña, edad entera entre 18 y 120 años, selección de país, aceptación de términos y formato de teléfonos.

**¿Qué ventaja tienen las funciones puras?**

Reciben un dato y devuelven un mensaje de error o `null`, sin modificar el formulario. Eso ayuda a reutilizarlas, entenderlas y probarlas por separado. En `app.js` se maneja la interacción con la página, y en `validaciones.js` están las reglas.

### Parte B. Comprobar si el correo ya existe

En `app.js` tenemos una lista ficticia de correos usados: `ana@mail.com` y `admin@mail.com`. La función `correoDisponible()` simula una consulta que tarda 600 milisegundos, mediante `setTimeout`. Además usamos `debounce` de 500 milisegundos para no revisar el correo cada vez que se escribe una letra inmediatamente.

**¿Por qué no se puede resolver solo con HTML?**

Los atributos HTML pueden comprobar el formato del correo, pero no saben si ya está registrado en una base de datos. Para una aplicación real necesitaríamos preguntar al servidor.

**Limitación de esta implementación:** la lista es simulada, no hay base de datos. Además, si el usuario envía el formulario antes de que termine la comprobación del correo, el código no espera obligatoriamente a obtener el resultado. Por eso no garantiza bloquear siempre un correo repetido.

### Parte C. Teléfonos dinámicos

La versión avanzada permite agregar **hasta tres** campos de teléfono y quitarlos con el botón correspondiente. Cada teléfono tiene una etiqueta, un mensaje de error y un identificador generado por JavaScript. Después de agregar uno, el foco pasa al nuevo campo; después de quitarlo, vuelve al botón «Agregar teléfono».

Si el usuario agrega un campo, debe escribir entre 7 y 15 dígitos, sin espacios; se permite el signo `+` al comienzo. Un teléfono que no se haya agregado no es obligatorio, pero un campo agregado y dejado vacío produce un error.

## Nivel 5. Seguridad y usabilidad

### Parte A. La validación en el frontend no garantiza seguridad

**¿Qué pasa si quitamos `min` y `max` del HTML e ingresamos una edad de 5 años?**

En el formulario nativo, quitar esos atributos elimina esa restricción del navegador. En cambio, en la versión avanzada sigue existiendo la regla `rangoEntero(18, 120)` en `validaciones.js`, así que el JavaScript del formulario todavía rechaza esa edad. Sin embargo, alguien podría modificar el código, deshabilitar JavaScript o enviar una petición directamente sin utilizar nuestro formulario. Por eso un servidor real debe volver a validar los datos.

**¿Qué es una consulta parametrizada y por qué previene SQL injection?**

Es una forma de enviar instrucciones SQL separadas de los valores escritos por el usuario. Así el motor de base de datos trata esos valores como datos y no como parte del código SQL. Nuestro proyecto **no tiene una base de datos ni ejecuta consultas SQL**, por lo que este punto es una explicación teórica y no una función implementada.

### Parte B. `innerHTML` y `textContent`

`textContent` inserta texto sin interpretarlo como HTML. `innerHTML` interpreta el contenido como código HTML y puede ser peligroso si se le agrega directamente texto no confiable de los usuarios.

En este proyecto mostramos el saludo con:

saludo.textContent = `Hola, ${datos.nombre}`;


Esto evita que el nombre se interprete como HTML. El código sí usa `innerHTML` para construir los campos de teléfono, pero allí el identificador variable es generado por el propio programa y no se toma de una entrada escrita por el usuario.

### Parte C. Usabilidad y accesibilidad

El código incluye etiquetas asociadas a sus campos, mensajes relacionados mediante `aria-describedby`, notificaciones con `aria-live="polite"`, marcado de errores mediante `aria-invalid`, un indicador visible del foco en CSS y el traslado del foco al primer campo inválido cuando se intenta enviar. También se deshabilita el botón mientras se hace la petición para evitar varios envíos seguidos.

**Prueba con un compañero:** el proyecto no contiene registros de una prueba real con otra persona. Para completar esta actividad del taller hay que pedirle a un compañero que use el formulario, anotar los problemas que encuentre y describir las mejoras realizadas. No afirmamos que esa prueba ya se haya hecho.

## Resumen de lo que funciona y sus límites

| Función | Estado en el código |
| --- | --- |
| Validaciones nativas en `nativo/index.html` | Implementadas |
| Mensajes personalizados y campos con borde rojo | Implementados en la versión avanzada |
| Validación de contraseña y confirmación iguales | Implementada en JavaScript |
| Envío con `FormData` y `fetch` | Implementado hacia servicio de prueba |
| Control de errores de red o HTTP | Implementado |
| Comprobación de correo repetido | Simulada; no es una verificación real ni bloquea todos los casos |
| Hasta tres teléfonos adicionales | Implementado |
| Saludo seguro mediante `textContent` | Implementado |
| Validación en servidor y almacenamiento real | No implementados |
| Prueba documentada con un compañero | Pendiente de realizar o registrar |

## Conclusión

Con el taller aprendimos a crear formularios y a validar los datos primero con HTML y después con JavaScript. También practicamos el envío de información, las funciones de validación reutilizables, los campos que se agregan dinámicamente y algunos conceptos de seguridad y accesibilidad. El formulario es una **demostración académica**, no un sistema de registro listo para producción: para usarlo con usuarios reales harían falta un backend, almacenamiento y verificaciones de seguridad del lado del servidor.
