---
title: Usando IndexedDB
slug: Web/API/IndexedDB_API/Using_IndexedDB
tags:
  - API
  - Almacenamiento
  - Avanzado
  - Base de datos
  - Guía
  - IndexedDB
  - Tutorial
  - jsstore
translation_of: Web/API/IndexedDB_API/Using_IndexedDB
original_slug: Web/API/IndexedDB_API/Usando_IndexedDB
---
<div class="summary">
<p>IndexedDB es una manera de almacenar datos dentro del navegador del usuario. Debido a que permite la creación de aplicaciones con habilidades de consulta enriquecidas, con independencia de la disponibilidad de la red, sus aplicaciones pueden trabajar tanto en línea como fuera de línea.</p>
</div>

<h2 id="Acerca_de_este_documento">Acerca de este documento</h2>

<p>Este tutorial es una guía sobre el uso de la API asíncrona de IndexedDB. Si no está familiarizado con IndexedDB, por favor consulte primero <a href="/en/IndexedDB/Basic_Concepts_Behind_IndexedDB" style="line-height: 1.5;" title="en/IndexedDB/Basic Concepts Behind IndexedDB">Conceptos Básicos Acerca de IndexedDB</a><span style="line-height: 1.5;">.</span></p>

<p>Para la documentación de referencia sobre la API de IndexedDB, vea el artículo <a href="/es/docs/IndexedDB" style="line-height: 1.5;" title="https://developer.mozilla.org/en/IndexedDB">IndexedDB</a><span style="line-height: 1.5;"> y sus subpaginas, que documentan los tipos de objetos usados por IndexedDB, así como los métodos síncronos y asíncronos. </span></p>

<h2 id="pattern" name="pattern">Patrones Básicos</h2>

<p>El patrón básico que indexedDB propone es:</p>

<ol>
 <li>Abrir una base de datos.</li>
 <li>Crear un objeto de almacenamiento en la base de datos.</li>
 <li>Iniciar una transacción y hacer una petición para hacer alguna operación de la base de datos, tal como añadir o recuperar datos.</li>
 <li>
  <div><span id="result_box" lang="es"><span class="hps">Espere a que</span> <span class="hps">se complete la operación</span> <span class="hps">por</span> <span class="hps">la escucha de la</span> <span class="hps">clase correcta de</span> <span class="hps">eventos DOM</span></span> .</div>
 </li>
 <li>
  <div>Hacer algo con el resultado (El cual puede ser encontrado en el objeto de la petición).</div>
 </li>
</ol>

<p>Con esos grandes rasgos en mente, seremos más concretos.</p>

<h2 id="open" name="open">Creando y estructurando el almacenamiento</h2>

<p>Como las especificaciones están todavía elaborandose, las implementaciones actuales de indexedDB dependen de los navegadores. H<span id="result_box" lang="es"><span class="hps">asta que la</span> <span class="hps">especificación</span> <span class="hps">se haya consolidado, l</span></span>os proveedores de navegadores pueden tener diferentes implementaciones de los estandares de indexedDB<span id="result_box" lang="es"><span>.</span></span> Una vez se alcance el consenso en el estandar, los proveedores implementarán la API sin prefijos. En algunas implementaciones ya fueron removidos los prefijos: Internet Explorer 10, Firefox 16, Chrome 24. Cuando utilizan un prefijo, los navegadores basados en gecko usan el prefijo <code>moz</code> , mientras que los navegadores basados en WebKit usan el prefijo <code>webkit</code>.</p>

<h3 id="Usando_una_versión_experimental_de_IndexedDB">Usando una versión experimental de IndexedDB</h3>

<p>En caso que usted quiera probar su código en navegadores que todavía usen un prefijo, puede usar el siguiente codigo:  </p>

<pre class="brush: js">// En la siguiente línea, puede incluir prefijos de implementación que quiera probar.
window.indexedDB = window.indexedDB || window.mozIndexedDB || window.webkitIndexedDB || window.msIndexedDB;
// No use "var indexedDB = ..." Si no está en una función.
// Por otra parte, puedes necesitar referencias a algun objeto window.IDB*:
window.IDBTransaction = window.IDBTransaction || window.webkitIDBTransaction || window.msIDBTransaction;
window.IDBKeyRange = window.IDBKeyRange || window.webkitIDBKeyRange || window.msIDBKeyRange;
// (Mozilla nunca ha prefijado estos objetos, por lo tanto no necesitamos window.mozIDB*)</pre>

<p>Hay que tener cuidado con las implementaciones que usan un prefijo ya que puede ser inestables, incompletas, o usen una versión antigua de la especificación. En producción se recomienda usar el código sin prefijos. Es preferible no tener soporte para un navegador a decir que lo tiene y fallar en ello :</p>

<pre class="brush: js">if (!window.indexedDB) {
    window.alert("Su navegador no soporta una versión estable de indexedDB. Tal y como las características no serán validas");
}
</pre>

<h3 id="Abriendo_una_base_de_datos">Abriendo una base de datos</h3>

<p>Iniciamos todo el proceso así:</p>

<pre class="brush: js">// dejamos abierta nuestra base de datos
var request = window.indexedDB.open("MyTestDatabase", 3);
</pre>

<p>¿Lo has visto? Abrir una base de datos es igual que cualquier otra operación — solo tienes que "solicitarla" (request).</p>

<p>La solicitud de apertura no abre la base de datos o inicia la transacción de inmediato. La llamada a la función <code>open()</code> retornan unos objetos <code><a href="/en-US/docs/IndexedDB/IDBOpenDBRequest" title="/en-US/docs/IndexedDB/IDBOpenDBRequest">IDBOpenDBRequest</a>,</code> cuyo resultado, correcto o erróneo, se maneja en un evento.  Alguna otra función asincrónica en indexedDB hace lo mismo - Devolver un objeto <a href="/en-US/docs/IndexedDB/IDBRequest" title="/en-US/docs/IndexedDB/IDBRequest"><code style="font-size: 14px; color: rgb(51, 51, 51);">IDBRequest</code></a> que disparará un evento con el resultado o el error. El resultado para la función de abrir es una instancia de un <code style="font-size: 14px; color: rgb(51, 51, 51);"><a href="/en-US/docs/IndexedDB/IDBDatabase" title="/en-US/docs/IndexedDB/IDBDatabase">IDBDatabase</a>.</code></p>

<p>El segundo parámetro para el método open es la versión de la base de datos. La versión de la base de datos determina el esquema - El almacen de objectos en la base de datos y su estructura. Si la base de datos no existe, es creada y se dispara un evento <code>onupgradeneeded</code> de inmediato, permitiéndote proveer una actualización de la estructura e índices en la función que capture dicho evento. Se verá más adelante en  <a href="#Updating_the_version_of_the_database">Actualizando la versión de la base de datos</a>. </p>

<div class="warning">
<p><strong>Importante</strong>: El número de versión es un <code>unsigned long</code>. Por lo tanto significa que puede ser un entero muy grande. También significa que si usas un flotante será convertido en un entero más cercano y la transacción puede no ser iniciada, el evento <code>upgradeneeded </code>no se desencadenará. Por ejemplo no use 2.4 como un número de versión ya que será igual que la 2:</p>

<pre class="brush: js">var request = indexedDB.open("MyTestDatabase", 2.4); // Esto no se hace, la versión será redondeada a 2</pre>
</div>

<h4 id="Generando_manipuladores">Generando manipuladores</h4>

<p>La primera cosa que usted querrá hacer con la totalidad de las peticiones que usted genera es agregar controladores de éxito y de error:</p>

<pre class="brush: js">request.onerror = function(event) {
  // Hacer algo con request.errorCode!
};
request.onsuccess = function(event) {
  // Hacer algo con request.result!
};</pre>

<p>¿Cuál de las dos funciones, onSuccess () o onerror (), se vuelve a llamar? Si todo tiene éxito, un evento de éxito (es decir, un evento DOM cuya propiedad tipo se establece en el "éxito") se dispara con la solicitud como su objetivo. Una vez que se dispara, la función onSuccess () a petición se activa con el evento de éxito como argumento. De lo contrario, si había algún problema, un evento de error (es decir, un evento DOM cuyo tipo de propiedad se establece en "error") se dispara a petición. Esto desencadena la función onerror () con el evento de error como argumento.</p>

<p>La API IndexedDB está diseñada para minimizar la necesidad de control de errores, por lo que no es probable que veamos muchos eventos de error (al menos, no una vez que estás acostumbrado a la API). En el caso de la apertura de una base de datos, sin embargo, hay algunas condiciones comunes que generan eventos de error. El problema más común se produce cuando el usuario ha decidido no dar, a su aplicación web, el permiso para crear una base de datos. Uno de los principales objetivos de diseño de IndexedDB es permitir que grandes cantidades de datos se almacenen para su uso sin conexión a internet. (Para obtener más información sobre la cantidad de almacenamiento que puede tener para cada navegador, consulte <a href="https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API#Storage_limits">Límites de almacenamiento</a>.)  </p>

<p>Obviamente, los navegadores no permitirán que alguna red de publicidad o sitio web malicioso pueda contaminar su ordenador, por ello los navegadores utilizan un diálogo para indicar al usuario la primera vez que cualquier aplicación web determinada intente abrir una IndexedDB para el almacenamiento. El usuario puede optar por permitir o denegar el acceso. Además, el almacenamiento IndexedDB en los modos de privacidad navegadores sólo dura en memoria hasta que la sesión de incógnito haya sido cerrada (modo de navegación privada para el modo de Firefox e Incognito para Chrome, pero en Firefox <a href="https://bugzilla.mozilla.org/show_bug.cgi?id=781982">no está implementado</a> a partir de noviembre 2015 por lo que no puede utilizar IndexedDB en Firefox navegación privada en absoluto).</p>

<p>Ahora, asumiendo que el usuario acepta su solicitud para crear una base de datos, y que ha recibido un evento de éxito para activar la devolución de llamada de éxito; ¿Que sigue? La solicitud aquí se generó con una llamada a indexedDB.open (), por lo request.result es una instancia de IDBDatabase, y que sin duda quieren ahorrar para más adelante. Su código podría ser algo como esto:</p>

<pre class="brush: js">var db;
var request = indexedDB.open("MyTestDatabase");
request.onerror = function(event) {
  alert("Why didn't you allow my web app to use IndexedDB?!");
};
request.onsuccess = function(event) {
  db = request.result;
};
</pre>

<h4 id="Manejando_errores">Manejando errores</h4>

<p>Como se mencionó anteriormente, los eventos de error de burbujas. Eventos de error se dirigen a la solicitud que generó el error, entonces el evento se propaga a la operación, y finalmente con el objeto de base de datos. Si desea evitar la adición de controladores de errores a cada solicitud, en su lugar puede añadir un solo controlador de errores en el objeto de base de datos, así:</p>

<pre class="brush: js">db.onerror = function(event) {
  // Generic error handler for all errors targeted at this database's
  // requests!
  alert("Database error: " + event.target.errorCode);
};
</pre>

<p>Uno de los errores más comunes posibles al abrir una base de datos es <code>VER_ERR</code>. Indica que la versión de la base de datos almacenada en el disco es mayor que la versión que está intentando abrir. Este es un caso de error que siempre debe ser manejado por el gestor de errores.</p>

<h3 id="Creación_o_actualización_de_la_versión_de_la_base_de_datos">Creación o actualización de la versión de la base de datos</h3>

<p>Cuando se crea una nueva base de datos o se aumenta el número de versión de una base de datos existente (mediante la especificación de un número de versión más alto de lo que hizo antes, en <a href="#cómo_abrir_una_base_de_datos">Cómo abrir una base de datos</a>), el evento onupgradeneeded se activará y un objeto <a href="https://developer.mozilla.org/en-US/docs/Web/API/IDBVersionChangeEvent">IDBVersionChangeEvent</a> será pasado a cualquier controlador de eventos <code>onversionchange</code> establecido en <code>request.result</code> (es decir, db en el ejemplo). En el controlador para el evento <code>upgradeneeded</code>, se debe crear los almacenes de objetos necesarios para esta versión de la base de datos:</p>

<pre class="brush:js;">// Este evento solamente está implementado en navegadores recientes
request.onupgradeneeded = function(event) {
  var db = event.target.result;

  // Crea un almacén de objetos (objectStore) para esta base de datos
  var objectStore = db.createObjectStore("name", { keyPath: "myKey" });
};</pre>

<p>En este caso, la base de datos ya tendrá los almacenes de objetos de la versión anterior de la base de datos, por lo que no tiene que crear estos almacenes de objetos de nuevo. Sólo es necesario crear nuevos almacenes de objetos, o eliminar las tiendas de objetos de la versión anterior que ya no son necesarios. Si necesita cambiar un almacén de objetos existentes (por ejemplo, para cambiar la ruta de acceso clave <code>keyPath</code>), entonces se debe eliminar el antiguo almacén de objetos y crear de nuevo con las nuevas opciones. (Tenga en cuenta que esto borrará la información en el almacén de objetos Si usted necesita guardar esa información, usted debe leerlo y guardarlo en otro lugar antes de actualizar la base de datos.)</p>

<p>Tratar de crear un almacén de objetos con un nombre que ya existe (o tratando de eliminar un almacén de objetos con un nombre que no existe) lanzará un error.</p>

<p>Si el evento <code>onupgradeneeded</code> retorna éxito, entonces se activará el manejador <code>onsuccess</code> de la solicitud de base de datos abierta.</p>

<p>Blink / Webkit soportan la versión actual de la especificación, tal como fue liberado en Chrome 23+ y Opera 17+ ; IE10+ también lo soporta. Implementaciones mas viejas o distintas no implementan la versión actual de la especificación, y por lo tanto no son compatibles todavía con el <code>indexedDB.open (nombre, versión).onupgradeneeded</code> . Para obtener más información sobre cómo actualizar la versión de la base de datos en Webkit/Blink mas viejos, consulte el artículo de referencia <a href="/es/docs/Web/API/IDBDatabase" title="https://developer.mozilla.org/en/IndexedDB/IDBDatabase#setVersion()_.0A.0ADeprecated">IDBDatabase</a>.</p>

<h3 id="Estructuración_de_la_base_de_datos">Estructuración de la base de datos</h3>

<p>Ahora para estructurar la base de datos. IndexedDB usa almacenes de datos (object stores) en lugar de tablas, y una base de datos puede contener cualquier número de almacenes. Cuando un valor es almacenado, se le asocia con una clave. Existen diversas maneras en que una clave pude ser indicada dependiendo de si el almacén usa una <a href="https://developer.mozilla.org/en/IndexedDB/Basic_Concepts_Behind_IndexedDB#gloss_keypath">ruta de clave</a> o <a href="https://developer.mozilla.org/en/IndexedDB/Basic_Concepts_Behind_IndexedDB#gloss_keygenerator">generador</a>.</p>

<p>La siguiente table muetra las distintas formas en que las claves pueden ser indicadas:</p>

<table class="standard-table">
 <thead>
  <tr>
   <th scope="col">Ruta de clave(<code>keyPath</code>)</th>
   <th scope="col">Generador de clave (<code>autoIncrement</code>)</th>
   <th scope="col">Descripción</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>No</td>
   <td>No</td>
   <td>Este almacén puede contener cualquier tipo de valor, incluso valores primitivos como números y cadenas. Se debe indicar un argumento de clave distinto cada vez que se agregue un nuevo valor.</td>
  </tr>
  <tr>
   <td>Si</td>
   <td>No</td>
   <td>Este almacén de objetos solo puede contener objetos de JavaScript. Los objetos deben tener una propiedad con el mismo nombre que la ruta de clave.</td>
  </tr>
  <tr>
   <td>No</td>
   <td>Si</td>
   <td>Este objeto puede contener cualquier tipo de valor. La clave es generada automáticamente, o se puede indicar un argumento de clave distinto si se quiere usar una clave específica.</td>
  </tr>
  <tr>
   <td>Si</td>
   <td>Si</td>
   <td>
    <p>Este almacén de objetos solo puede contener objetos de JavaScript. Usualmente una clave es generada y dicho valor es almacenado en el objeto en una propiedad con el mismo nombre que la ruta de clave. Sin embargo, si dicha propiedad ya existe en el objeto, el valor de esa propuiedad es usado como clave en lugar de generar una nueva.</p>
   </td>
  </tr>
 </tbody>
</table>

<p>También se puede crear índices en cualquer almacén de objetos, siempre y cuando el almacén contenga objets, y no primitivos. Un índice permite buscar valores contenidos en el almacén usando el valor de una propiedad del objeto almacenado, en lugar de la clave del mismo.</p>

<p>Adicionalmente, los índices tienen la habilidad para hacer cumplir restricciones simples en los datos almacendos. Al indicar la bandera <code>unique</code> al crear el índice, el índice asegurará que no se puedan almacenar dos objetos que tengan el mismo valor para la clave del índice. Así, por ejemplo si se tiene un almacén de objetos que contiene un set de personas, y se desea asegurar que no existan dos personas con el mismo email, se puede usar un índice con la bandera <code>unique</code> activada para forzar esto.</p>

<p>Esto puede sonar confuso, pero un ejemplo simple debe ilustrar el concepto. Primero, definiremos alguna información de clientes para usar en nuestro ejemplo:</p>

<pre class="brush: js">// Así se ve nuestra información de clientes.
const customerData = [
  { ssn: "444-44-4444", name: "Bill", age: 35, email: "bill@company.com" },
  { ssn: "555-55-5555", name: "Donna", age: 32, email: "donna@home.org" }
];
</pre>

<p>Ahora, creemos una IndexedDB para almacenar los datos:</p>

<pre class="brush: js">const dbName = "the_name";

var request = indexedDB.open(dbName, 2);

request.onerror = function(event) {
  // Manejar errores.
};
request.onupgradeneeded = function(event) {
  var db = event.target.result;

  // Se crea un almacén para contener la información de nuestros cliente
  // Se usará "ssn" como clave ya que es garantizado que es única
  var objectStore = db.createObjectStore("customers", { keyPath: "ssn" });

  // Se crea un índice para buscar clientes por nombre. Se podrían tener duplicados
  // por lo que no se puede usar un índice único.
  objectStore.createIndex("name", "name", { unique: false });

  // Se crea un índice para buscar clientespor email. Se quiere asegurar que
  // no puedan haberdos clientes con el mismo email, asi que se usa un índice único.
  objectStore.createIndex("email", "email", { unique: true });

  // Se usa transaction.oncomplete para asegurarse que la creación del almacén
  // haya finalizado antes de añadir los datos en el.
  objectStore.transaction.oncomplete = function(event) {
    // Guarda los datos en el almacén recién creado.
    var customerObjectStore = db.transaction("customers", "readwrite").objectStore("customers");
    for (var i in customerData) {
      customerObjectStore.add(customerData[i]);
    }
  }
};
</pre>

<p>Como se indicó previamente, <code>onupgradeneeded</code> es el único lugar donde se puede alterar la estructura de la base de datos. En el, se puede crear y borrar almacenes de objetos y construir y remover índices.</p>

<div>Los almacenes de datos son creados con una llamada a  <code>createObjectStore()</code>. El método toma como parámetros el nombre del almacén y un objeto.  A pesar de que el segundo parámetro es opcional, es muy importante, porque permite definir propiedades opcionales importantes y refinar el tipo de almacén que se desea crear. En este caso, se pregunta por un almacén llamado "customers" y se define la clave, que es la propiedad que indica que un objeto en el almacén es único. La propiedad en este ejemplo es "ssn" (Social Security Number) ya que los números de seguridad social está garantizado que sea único. "ssn" debe estar presente en cada objeto que se guarda al almacén.</div>

<p>También se solicitó crear un índice llamado "name" que se fija en la propiedad <code>name </code>de los objetos almacenados. Así como con <code>createObjectStore()</code>, <code>createIndex()</code> toma un objeto opcional <code>options</code> que refina el tipo de índice que se desea crear. Agregar objetos que no tengan una propiedad <code>name</code> funcionará, pero los objetos no aparecerán en el índice "name"</p>

<div>Ahora se pueden obtener los clientes almacenados usando su <code>ssn</code> directamente del almacen, o usando su nombre a través del índice. Para aprender como hacer esto, vea la sección <a href="#El_uso_de_un_índice">El uso de un índice</a></div>

<h3 id="El_uso_de_un_generador_de_claves">El uso de un generador de claves</h3>

<p>Indicar la bandera <code>autoIncrement </code> cuando se crea el almacén habilitará el generador de claves para dicho almacén. Por defecto esta bandera no está marcada.</p>

<p>Con el generador de claves, la clave será generada automáticamente a medida que se agreguen valores al almacén. El número actual de un generador de claves siempre se establece en 1 cuando se creal el almacén por primera vez. Básicamente la nueva clave autogenerada es incrementada en 1 basada en la llave anterior. El numero actual para un generador de claves nunca disminuye, salvo como resultado de operaciones de base de datos  que sean revertidos, por ejemplo, cuando la transacción de base de datos es abortada. Por lo tanto borrar un registro o incluso borrar todos los registros de un almacén nunca afecta al generador de claves</p>

<p>Se puede crear otro almacén de objetos con generador de claves como se muestra abajo:</p>

<pre class="brush: js">// Abrir la indexedDB.
var request = indexedDB.open(dbName, 3);

request.onupgradeneeded = function (event) {

    var db = event.target.result;

    // Create another object store called "names" with the autoIncrement flag set as true.
    var objStore = db.createObjectStore("names", { autoIncrement : true });

    // Because the "names" object store has the key generator, the key for the name value is generated automatically.
    // The added records would be like:
    // key : 1 =&gt; value : "Bill"
    // key : 2 =&gt; value : "Donna"
    for (var i in customerData) {
        objStore.add(customerData[i].name);
    }
}</pre>

<p>Para más detalles acerca del generador de claves, por favor ver <a href="http://www.w3.org/TR/IndexedDB/#key-generator-concept">"W3C Key Generators"</a>.</p>

<h2 id="Añadir_recuperación_y_eliminación_de_datos">Añadir, recuperación y eliminación de datos</h2>

<p>Antes que haga algo con su nueva base de datos , necesita comenzar una transacción. Transactions come from the database object, and you have to specify which object stores you want the transaction to span. Once you are inside the transaction, you can access the object stores that hold your data and make your requests. Next, you need to decide if you're going to make changes to the database or if you just need to read from it. Transactions have three available modes: <code>readonly</code>, <code>readwrite</code>, and <code>versionchange</code>.</p>

<p>To change the "schema" or structure of the database—which involves creating or deleting object stores or indexes—the transaction must be in <code>versionchange</code> mode. This transaction is opened by calling the {{domxref("IDBFactory.open")}} method with a <code>version</code> specified. (In WebKit browsers, which have not implemented the latest specifcation, the {{domxref("IDBFactory.open")}} method takes only one parameter, the <code>name</code> of the database; then you must call {{domxref("IDBVersionChangeRequest.setVersion")}} to establish the <code>versionchange</code> transaction.)</p>

<p>To read the records of an existing object store, the transaction can either be in <code>readonly</code> or <code>readwrite</code> mode. To make changes to an existing object store, the transaction must be in <code>readwrite</code> mode. You open such transactions with {{domxref("IDBDatabase.transaction")}}. The method accepts two parameters: the <code>storeNames</code> (the scope, defined as an array of object stores that you want to access) and the <code>mode</code> (<code>readonly</code> or <code>readwrite</code>) for the transaction. The method returns a transaction object containing the {{domxref("IDBIndex.objectStore")}} method, which you can use to access your object store. By default, where no mode is specified, transactions open in <code>readonly</code> mode.</p>

<p>You can speed up data access by using the right scope and mode in the transaction. Here are a couple of tips:</p>

<ul>
 <li>When defining the scope, specify only the object stores you need. This way, you can run multiple transactions with non-overlapping scopes concurrently.</li>
 <li>Only specify a <code>readwrite</code> transaction mode when necessary. You can concurrently run multiple <code>readonly</code> transactions with overlapping scopes, but you can have only one <code>readwrite</code> transaction for an object store. To learn more, see the definition for <dfn><a href="/en-US/docs/IndexedDB/Basic_Concepts_Behind_IndexedDB#Database">transactions</a></dfn> in the <a href="/en-US/docs/IndexedDB/Basic_Concepts_Behind_IndexedDB">Basic Concepts</a> article.</li>
</ul>

<h3 id="Agregar_datos_a_la_base_de_datos">Agregar datos a la base de datos</h3>

<p>If you've just created a database, then you probably want to write to it. Here's what that looks like:</p>

<pre class="brush:js;">var transaction = db.transaction(["customers"], "readwrite");
// Note: Older experimental implementations use the deprecated constant IDBTransaction.READ_WRITE instead of "readwrite".
// In case you want to support such an implementation, you can write:
// var transaction = db.transaction(["customers"], IDBTransaction.READ_WRITE);</pre>

<p>The <code>transaction()</code> function takes two arguments (though one is optional) and returns a transaction object. The first argument is a list of object stores that the transaction will span. You can pass an empty array if you want the transaction to span all object stores, but don't do it because the spec says an empty array should generate an InvalidAccessError. If you don't specify anything for the second argument, you get a read-only transaction. Since you want to write to it here you need to pass the <code>"readwrite"</code> flag.</p>

<p>Now that you have a transaction you need to understand its lifetime. Transactions are tied very closely to the event loop. If you make a transaction and return to the event loop without using it then the transaction will become inactive. The only way to keep the transaction active is to make a request on it. When the request is finished you'll get a DOM event and, assuming that the request succeeded, you'll have another opportunity to extend the transaction during that callback. If you return to the event loop without extending the transaction then it will become inactive, and so on. As long as there are pending requests the transaction remains active. Transaction lifetimes are really very simple but it might take a little time to get used to. A few more examples will help, too. If you start seeing <code>TRANSACTION_INACTIVE_ERR</code> error codes then you've messed something up.</p>

<p>Transactions can receive DOM events of three different types: <code>error</code>, <code>abort</code>, and <code>complete</code>. We've talked about the way that <code>error</code> events bubble, so a transaction receives error events from any requests that are generated from it. A more subtle point here is that the default behavior of an error is to abort the transaction in which it occurred. Unless you handle the error by first calling <code>preventDefault()</code> on the error event then doing something else, the entire transaction is rolled back. This design forces you to think about and handle errors, but you can always add a catchall error handler to the database if fine-grained error handling is too cumbersome. If you don't handle an error event or if you call <code>abort()</code> on the transaction, then the transaction is rolled back and an <code>abort</code> event is fired on the transaction. Otherwise, after all pending requests have completed, you'll get a <code>complete</code> event. If you're doing lots of database operations, then tracking the transaction rather than individual requests can certainly aid your sanity.</p>

<p>Now that you have a transaction, you'll need to get the object store from it. Transactions only let you have an object store that you specified when creating the transaction. Then you can add all the data you need.</p>

<pre class="brush: js">// Do something when all the data is added to the database.
transaction.oncomplete = function(event) {
  alert("All done!");
};

transaction.onerror = function(event) {
  // Don't forget to handle errors!
};

var objectStore = transaction.objectStore("customers");
for (var i in customerData) {
  var request = objectStore.add(customerData[i]);
  request.onsuccess = function(event) {
    // event.target.result == customerData[i].ssn;
  };
}</pre>

<p>The <code>result</code> of a request generated from a call to <code>add()</code> is the key of the value that was added. So in this case, it should equal the <code>ssn</code> property of the object that was added, since the object store uses the <code>ssn</code> property for the key path. Note that the <code>add()</code> function requires that no object already be in the database with the same key. If you're trying to modify an existing entry, or you don't care if one exists already, you can use the <code>put()</code> function, as shown below in the <a href="#updating_an_entry_in_the_database">Updating an entry in the database</a> section.</p>

<h3 id="Extracción_de_datos_de_la_base_de_datos">Extracción de datos de la base de datos</h3>

<p>Removing data is very similar:</p>

<pre class="brush: js">var request = db.transaction(["customers"], "readwrite")
                .objectStore("customers")
                .delete("444-44-4444");
request.onsuccess = function(event) {
  // It's gone!
};</pre>

<h3 id="Obtener_datos_de_la_base_de_datos">Obtener datos de la base de datos</h3>

<p>Now that the database has some info in it, you can retrieve it in several ways. First, the simple <code>get()</code>. You need to provide the key to retrieve the value, like so:</p>

<pre class="brush: js">var transaction = db.transaction(["customers"]);
var objectStore = transaction.objectStore("customers");
var request = objectStore.get("444-44-4444");
request.onerror = function(event) {
  // Handle errors!
};
request.onsuccess = function(event) {
  // Do something with the request.result!
  alert("Name for SSN 444-44-4444 is " + request.result.name);
};</pre>

<p>That's a lot of code for a "simple" retrieval. Here's how you can shorten it up a bit, assuming that you handle errors at the database level:</p>

<pre class="brush: js">db.transaction("customers").objectStore("customers").get("444-44-4444").onsuccess = function(event) {
  alert("Name for SSN 444-44-4444 is " + event.target.result.name);
};</pre>

<p>See how this works? Since there's only one object store, you can avoid passing a list of object stores you need in your transaction and just pass the name as a string. Also, you're only reading from the database, so you don't need a <code>"readwrite"</code> transaction. Calling <code>transaction()</code> with no mode specified gives you a <code>"readonly"</code> transaction. Another subtlety here is that you don't actually save the request object to a variable. Since the DOM event has the request as its target you can use the event to get to the <code>result</code> property.</p>

<div class="note">
<p><strong>Note</strong>: You can speed up data access by limiting the scope and mode in the transaction. Here are a couple of tips:</p>

<ul>
 <li>
  <p>When defining the <a href="#scope">scope</a>, specify only the object stores you need. This way, you can run multiple transactions with non-overlapping scopes concurrently.</p>
 </li>
 <li>
  <p>Only specify a <code>readwrite</code> transaction mode when necessary. You can concurrently run multiple <code>readonly</code> transactions with overlapping scopes, but you can have only one <code>readwrite</code> transaction for an object store. To learn more, see the definition for <a href="/en-US/docs/IndexedDB/Basic_Concepts_Behind_IndexedDB#gloss_transaction"><dfn>transactions</dfn> in the Basic Concepts article</a>.</p>
 </li>
</ul>
</div>

<h3 id="Actualización_de_una_entrada_en_la_base_de_datos">Actualización de una entrada en la base de datos</h3>

<p>Now we've retrieved some data, updating it and inserting it back into the IndexedDB is pretty simple. Let's update the previous example somewhat:</p>

<pre class="brush: js">var objectStore = db.transaction(["customers"], "readwrite").objectStore("customers");
var request = objectStore.get("444-44-4444");
request.onerror = function(event) {
  // Handle errors!
};
request.onsuccess = function(event) {
  // Get the old value that we want to update
  var data = request.result;

  // update the value(s) in the object that you want to change
  data.age = 42;

  // Put this updated object back into the database.
  var requestUpdate = objectStore.put(data);
   requestUpdate.onerror = function(event) {
     // Do something with the error
   };
   requestUpdate.onsuccess = function(event) {
     // Success - the data is updated!
   };
};</pre>

<p>So here we're creating an <code>objectStore</code> and requesting a customer record out of it, identified by its ssn value (<code>444-44-4444</code>). We then put the result of that request in a variable (<code>data</code>), update the <code>age</code> property of this object, then create a second request (<code>requestUpdate</code>) to put the customer record back into the <code>objectStore</code>, overwriting the previous value.</p>

<div class="note">
<p><strong>Note</strong> that in this case we've had to specify a <code>readwrite</code> transaction because we want to write to the database, not just read out of it.</p>
</div>

<h3 id="El_uso_de_un_cursor">El uso de un cursor</h3>

<p>Using <code>get()</code> requires that you know which key you want to retrieve. If you want to step through all the values in your object store, then you can use a cursor. Here's what it looks like:</p>

<pre class="brush: js">var objectStore = db.transaction("customers").objectStore("customers");

objectStore.openCursor().onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    alert("Name for SSN " + cursor.key + " is " + cursor.value.name);
    cursor.continue();
  }
  else {
    alert("No more entries!");
  }
};</pre>

<p>The<code> openCursor()</code> function takes several arguments. First, you can limit the range of items that are retrieved by using a key range object that we'll get to in a minute. Second, you can specify the direction that you want to iterate. In the above example, we're iterating over all objects in ascending order. The success callback for cursors is a little special. The cursor object itself is the <code>result</code> of the request (above we're using the shorthand, so it's <code>event.target.result</code>). Then the actual key and value can be found on the <code>key</code> and <code>value</code> properties of the cursor object. If you want to keep going, then you have to call <code>continue()</code> on the cursor. When you've reached the end of the data (or if there were no entries that matched your <code>openCursor()</code> request) you still get a success callback, but the <code>result</code> property is <code>undefined</code>.</p>

<p>One common pattern with cursors is to retrieve all objects in an object store and add them to an array, like this:</p>

<pre class="brush: js">var customers = [];

objectStore.openCursor().onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    customers.push(cursor.value);
    cursor.continue();
  }
  else {
    alert("Got all customers: " + customers);
  }
};</pre>

<div class="note">
<p>Note: Mozilla has also implemented <code>getAll()</code> to handle this case (and <code>getAllKeys()</code>, which is currently hidden behind the <code>dom.indexedDB.experimental</code> preference in about:config). these aren't part of the IndexedDB standard, so may disappear in the future. We've included them because we think they're useful. The following code does precisely the same thing as above:</p>

<pre class="brush: js">objectStore.getAll().onsuccess = function(event) {
  alert("Got all customers: " + event.target.result);
};</pre>

<p>There is a performance cost associated with looking at the <code>value</code> property of a cursor, because the object is created lazily. When you use <code>getAll()</code> for example, Gecko must create all the objects at once. If you're just interested in looking at each of the keys, for instance, it is much more efficient to use a cursor than to use <code>getAll()</code>. If you're trying to get an array of all the objects in an object store, though, use <code>getAll()</code>.</p>
</div>

<h3 id="El_uso_de_un_índice">El uso de un índice</h3>

<p>Storing customer data using the SSN as a key is logical since the SSN uniquely identifies an individual. (Whether this is a good idea for privacy is a different question, and outside the scope of this article.) If you need to look up a customer by name, however, you'll need to iterate over every SSN in the database until you find the right one. Searching in this fashion would be very slow, so instead you can use an index.</p>

<pre class="brush: js">var index = objectStore.index("name");
index.get("Donna").onsuccess = function(event) {
  alert("Donna's SSN is " + event.target.result.ssn);
};</pre>

<p>The "name" cursor isn't unique, so there could be more than one entry with the <code>name</code> set to <code>"Donna"</code>. In that case you always get the one with the lowest key value.</p>

<p>If you need to access all the entries with a given <code>name</code> you can use a cursor. You can open two different types of cursors on indexes. A normal cursor maps the index property to the object in the object store. A key cursor maps the index property to the key used to store the object in the object store. The differences are illustrated here:</p>

<pre class="brush: js">// Using a normal cursor to grab whole customer record objects
index.openCursor().onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // cursor.key is a name, like "Bill", and cursor.value is the whole object.
    alert("Name: " + cursor.key + ", SSN: " + cursor.value.ssn + ", email: " + cursor.value.email);
    cursor.continue();
  }
};

// Using a key cursor to grab customer record object keys
index.openKeyCursor().onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // cursor.key is a name, like "Bill", and cursor.value is the SSN.
    // No way to directly get the rest of the stored object.
    alert("Name: " + cursor.key + ", SSN: " + cursor.value);
    cursor.continue();
  }
};</pre>

<h3 id="Especificación_de_la_gama_y_la_dirección_de_los_cursores">Especificación de la gama y la dirección de los cursores</h3>

<p>If you would like to limit the range of values you see in a cursor, you can use an <code>IDBKeyRange</code> object and pass it as the first argument to <code>openCursor()</code> or <code>openKeyCursor()</code>. You can make a key range that only allows a single key, or one that has a lower or upper bound, or one that has both a lower and upper bound. The bound may be "closed" (i.e., the key range includes the given value(s)) or "open" (i.e., the key range does not include the given value(s)). Here's how it works:</p>

<pre class="brush: js">// Only match "Donna"
var singleKeyRange = IDBKeyRange.only("Donna");

// Match anything past "Bill", including "Bill"
var lowerBoundKeyRange = IDBKeyRange.lowerBound("Bill");

// Match anything past "Bill", but don't include "Bill"
var lowerBoundOpenKeyRange = IDBKeyRange.lowerBound("Bill", true);

// Match anything up to, but not including, "Donna"
var upperBoundOpenKeyRange = IDBKeyRange.upperBound("Donna", true);

// Match anything between "Bill" and "Donna", but not including "Donna"
var boundKeyRange = IDBKeyRange.bound("Bill", "Donna", false, true);

// To use one of the key ranges, pass it in as the first argument of openCursor()/openKeyCursor()
index.openCursor(boundKeyRange).onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // Do something with the matches.
    cursor.continue();
  }
};</pre>

<p>Sometimes you may want to iterate in descending order rather than in ascending order (the default direction for all cursors). Switching direction is accomplished by passing <code>prev</code> to the <code>openCursor()</code> function as the second argument:</p>

<pre class="brush: js">objectStore.openCursor(boundKeyRange, "prev").onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // Do something with the entries.
    cursor.continue();
  }
};</pre>

<p>If you just want to specify a change of direction but not constrain the results shown, you can just pass in null as the first argument:</p>

<pre class="brush: js">objectStore.openCursor(null, "prev").onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // Do something with the entries.
    cursor.continue();
  }
};</pre>

<p>Since the "name" index isn't unique, there might be multiple entries where <code>name</code> is the same. Note that such a situation cannot occur with object stores since the key must always be unique. If you wish to filter out duplicates during cursor iteration over indexes, you can pass <code>nextunique</code> (or <code>prevunique</code> if you're going backwards) as the direction parameter. When <code>nextunique</code> or <code>prevunique</code> is used, the entry with the lowest key is always the one returned.</p>

<pre class="brush: js">index.openKeyCursor(null, "nextunique").onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // Do something with the entries.
    cursor.continue();
  }
};</pre>

<p>Please see "<a href="https://developer.mozilla.org/en-US/docs/Web/API/IDBCursor?redirectlocale=en-US&amp;redirectslug=IndexedDB%2FIDBCursor#Constants">IDBCursor Constants</a>" for the valid direction arguments.</p>

<h2 id="Cambios_Versión_mientras_que_una_aplicación_web_está_abierto_en_otra_pestaña">Cambios Versión mientras que una aplicación web está abierto en otra pestaña</h2>

<p>When your web app changes in such a way that a version change is required for your database, you need to consider what happens if the user has the old version of your app open in one tab and then loads the new version of your app in another. When you call <code>open()</code> with a greater version than the actual version of the database, all other open databases must explicitly acknowledge the request before you can start making changes to the database (an <code>onblocked</code> event is fired until tey are closed or reloaded). Here's how it works:</p>

<pre class="brush: js">var openReq = mozIndexedDB.open("MyTestDatabase", 2);

openReq.onblocked = function(event) {
  // If some other tab is loaded with the database, then it needs to be closed
  // before we can proceed.
  alert("Please close all other tabs with this site open!");
};

openReq.onupgradeneeded = function(event) {
  // All other databases have been closed. Set everything up.
  db.createObjectStore(/* ... */);
  useDatabase(db);
}

openReq.onsuccess = function(event) {
  var db = event.target.result;
  useDatabase(db);
  return;
}

function useDatabase(db) {
  // Make sure to add a handler to be notified if another page requests a version
  // change. We must close the database. This allows the other page to upgrade the database.
  // If you don't do this then the upgrade won't happen until the user closes the tab.
  db.onversionchange = function(event) {
    db.close();
    alert("A new version of this page is ready. Please reload!");
  };

  // Do stuff with the database.
}
</pre>

<h2 id="Seguridad">Seguridad</h2>

<p>IndexedDB uses the same-origin principle, which means that it ties the store to the origin of the site that creates it (typically, this is the site domain or subdomain), so it cannot be accessed by any other origin.</p>

<p>It's important to note that IndexedDB doesn't work for content loaded into a frame from another site (either {{ HTMLElement("frame") }} or {{ HTMLElement("iframe") }}. This is a security and privacy measure and can be considered analogous the blocking of third-party cookies. For more details, see {{ bug(595307) }}.</p>

<h2 id="Warning_About_Browser_Shutdown">Warning About Browser Shutdown</h2>

<p>When the browser shuts down (e.g., when the user selects Exit or clicks the Close button), any pending IndexedDB transactions are (silently) aborted — they will not complete, and they will not trigger the error handler. Since the user can exit the browser at any time, this means that you cannot rely upon any particular transaction to complete or to know that it did not complete. There are several implications of this behavior.</p>

<p>First, you should take care to always leave your database in a consistent state at the end of every transaction. For example, suppose that you are using IndexedDB to store a list of items that you allow the user to edit. You save the list after the edit by clearing the object store and then writing out the new list. If you clear the object store in one transaction and write the new list in another transaction, there is a danger that the browser will close after the clear but before the write, leaving you with an empty database. To avoid this, you should combine the clear and the write into a single transaction. </p>

<p>Second, you should never tie database transactions to unload events. If the unload event is triggered by the browser closing, any transactions created in the unload event handler will never complete. An intuitive approach to maintaining some information across browser sessions is to read it from the database when the browser (or a particular page) is opened, update it as the user interacts with the browser, and then save it to the database when the browser (or page) closes. However, this will not work. The database transactions will be created in the unload event handler, but because they are asynchronous they will be aborted before they can execute.</p>

<p>In fact, there is no way to guarantee that IndexedDB transactions will complete, even with normal browser shutdown. See {{ bug(870645) }}.</p>

<h2 id="Full_IndexedDB_example" name="Full_IndexedDB_example">Full IndexedDB example</h2>

<h3 id="HTML_Content">HTML Content</h3>

<pre class="brush: html">&lt;script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.8.3/jquery.min.js"&gt;&lt;/script&gt;

    &lt;h1&gt;IndexedDB Demo: storing blobs, e-publication example&lt;/h1&gt;
    &lt;div class="note"&gt;
      &lt;p&gt;
        Works and tested with:
      &lt;/p&gt;
      &lt;div id="compat"&gt;
      &lt;/div&gt;
    &lt;/div&gt;

    &lt;div id="msg"&gt;
    &lt;/div&gt;

    &lt;form id="register-form"&gt;
      &lt;table&gt;
        &lt;tbody&gt;
          &lt;tr&gt;
            &lt;td&gt;
              &lt;label for="pub-title" class="required"&gt;
                Title:
              &lt;/label&gt;
            &lt;/td&gt;
            &lt;td&gt;
              &lt;input type="text" id="pub-title" name="pub-title" /&gt;
            &lt;/td&gt;
          &lt;/tr&gt;
          &lt;tr&gt;
            &lt;td&gt;
              &lt;label for="pub-biblioid" class="required"&gt;
                Bibliographic ID:&lt;br/&gt;
                &lt;span class="note"&gt;(ISBN, ISSN, etc.)&lt;/span&gt;
              &lt;/label&gt;
            &lt;/td&gt;
            &lt;td&gt;
              &lt;input type="text" id="pub-biblioid" name="pub-biblioid"/&gt;
            &lt;/td&gt;
          &lt;/tr&gt;
          &lt;tr&gt;
            &lt;td&gt;
              &lt;label for="pub-year"&gt;
                Year:
              &lt;/label&gt;
            &lt;/td&gt;
            &lt;td&gt;
              &lt;input type="number" id="pub-year" name="pub-year" /&gt;
            &lt;/td&gt;
          &lt;/tr&gt;
        &lt;/tbody&gt;
        &lt;tbody&gt;
          &lt;tr&gt;
            &lt;td&gt;
              &lt;label for="pub-file"&gt;
                File image:
              &lt;/label&gt;
            &lt;/td&gt;
            &lt;td&gt;
              &lt;input type="file" id="pub-file"/&gt;
            &lt;/td&gt;
          &lt;/tr&gt;
          &lt;tr&gt;
            &lt;td&gt;
              &lt;label for="pub-file-url"&gt;
                Online-file image URL:&lt;br/&gt;
                &lt;span class="note"&gt;(same origin URL)&lt;/span&gt;
              &lt;/label&gt;
            &lt;/td&gt;
            &lt;td&gt;
              &lt;input type="text" id="pub-file-url" name="pub-file-url"/&gt;
            &lt;/td&gt;
          &lt;/tr&gt;
        &lt;/tbody&gt;
      &lt;/table&gt;

      &lt;div class="button-pane"&gt;
        &lt;input type="button" id="add-button" value="Add Publication" /&gt;
        &lt;input type="reset" id="register-form-reset"/&gt;
      &lt;/div&gt;
    &lt;/form&gt;

    &lt;form id="delete-form"&gt;
      &lt;table&gt;
        &lt;tbody&gt;
          &lt;tr&gt;
            &lt;td&gt;
              &lt;label for="pub-biblioid-to-delete"&gt;
                Bibliographic ID:&lt;br/&gt;
                &lt;span class="note"&gt;(ISBN, ISSN, etc.)&lt;/span&gt;
              &lt;/label&gt;
            &lt;/td&gt;
            &lt;td&gt;
              &lt;input type="text" id="pub-biblioid-to-delete"
                     name="pub-biblioid-to-delete" /&gt;
            &lt;/td&gt;
          &lt;/tr&gt;
          &lt;tr&gt;
            &lt;td&gt;
              &lt;label for="key-to-delete"&gt;
                Key:&lt;br/&gt;
                &lt;span class="note"&gt;(for example 1, 2, 3, etc.)&lt;/span&gt;
              &lt;/label&gt;
            &lt;/td&gt;
            &lt;td&gt;
              &lt;input type="text" id="key-to-delete"
                     name="key-to-delete" /&gt;
            &lt;/td&gt;
          &lt;/tr&gt;
        &lt;/tbody&gt;
      &lt;/table&gt;
      &lt;div class="button-pane"&gt;
        &lt;input type="button" id="delete-button" value="Delete Publication" /&gt;
        &lt;input type="button" id="clear-store-button"
               value="Clear the whole store" class="destructive" /&gt;
      &lt;/div&gt;
    &lt;/form&gt;

    &lt;form id="search-form"&gt;
      &lt;div class="button-pane"&gt;
        &lt;input type="button" id="search-list-button"
               value="List database content" /&gt;
      &lt;/div&gt;
    &lt;/form&gt;

    &lt;div&gt;
      &lt;div id="pub-msg"&gt;
      &lt;/div&gt;
      &lt;div id="pub-viewer"&gt;
      &lt;/div&gt;
      &lt;ul id="pub-list"&gt;
      &lt;/ul&gt;
    &lt;/div&gt;
</pre>

<h3 id="CSS_Content">CSS Content</h3>

<pre class="brush: css">body {
  font-size: 0.8em;
  font-family: Sans-Serif;
}

form {
  background-color: #cccccc;
  border-radius: 0.3em;
  display: inline-block;
  margin-bottom: 0.5em;
  padding: 1em;
}

table {
  border-collapse: collapse;
}

input {
  padding: 0.3em;
  border-color: #cccccc;
  border-radius: 0.3em;
}

.required:after {
  content: "*";
  color: red;
}

.button-pane {
  margin-top: 1em;
}

#pub-viewer {
  float: right;
  width: 48%;
  height: 20em;
  border: solid #d092ff 0.1em;
}
#pub-viewer iframe {
  width: 100%;
  height: 100%;
}

#pub-list {
  width: 46%;
  background-color: #eeeeee;
  border-radius: 0.3em;
}
#pub-list li {
  padding-top: 0.5em;
  padding-bottom: 0.5em;
  padding-right: 0.5em;
}

#msg {
  margin-bottom: 1em;
}

.action-success {
  padding: 0.5em;
  color: #00d21e;
  background-color: #eeeeee;
  border-radius: 0.2em;
}

.action-failure {
  padding: 0.5em;
  color: #ff1408;
  background-color: #eeeeee;
  border-radius: 0.2em;
}

.note {
  font-size: smaller;
}

.destructive {
  background-color: orange;
}
.destructive:hover {
  background-color: #ff8000;
}
.destructive:active {
  background-color: red;
}
</pre>

<p> </p>

<h3 id="JavaScript_Content">JavaScript Content</h3>

<pre class="brush: js">(function () {
  var COMPAT_ENVS = [
    ['Firefox', "&gt;= 16.0"],
    ['Google Chrome',
     "&gt;= 24.0 (you may need to get Google Chrome Canary), NO Blob storage support"]
  ];
  var compat = $('#compat');
  compat.empty();
  compat.append('&lt;ul id="compat-list"&gt;&lt;/ul&gt;');
  COMPAT_ENVS.forEach(function(val, idx, array) {
    $('#compat-list').append('&lt;li&gt;' + val[0] + ': ' + val[1] + '&lt;/li&gt;');
  });

  const DB_NAME = 'mdn-demo-indexeddb-epublications';
  const DB_VERSION = 1; // Use a long long for this value (don't use a float)
  const DB_STORE_NAME = 'publications';

  var db;

  // Used to keep track of which view is displayed to avoid uselessly reloading it
  var current_view_pub_key;

  function openDb() {
    console.log("openDb ...");
    var req = indexedDB.open(DB_NAME, DB_VERSION);
    req.onsuccess = function (evt) {
      // Better use "this" than "req" to get the result to avoid problems with
      // garbage collection.
      // db = req.result;
      db = this.result;
      console.log("openDb DONE");
    };
    req.onerror = function (evt) {
      console.error("openDb:", evt.target.errorCode);
    };

    req.onupgradeneeded = function (evt) {
      console.log("openDb.onupgradeneeded");
      var store = evt.currentTarget.result.createObjectStore(
        DB_STORE_NAME, { keyPath: 'id', autoIncrement: true });

      store.createIndex('biblioid', 'biblioid', { unique: true });
      store.createIndex('title', 'title', { unique: false });
      store.createIndex('year', 'year', { unique: false });
    };
  }

  /**
   * @param {string} store_name
   * @param {string} mode either "readonly" or "readwrite"
   */
  function getObjectStore(store_name, mode) {
    var tx = db.transaction(store_name, mode);
    return tx.objectStore(store_name);
  }

  function clearObjectStore(store_name) {
    var store = getObjectStore(DB_STORE_NAME, 'readwrite');
    var req = store.clear();
    req.onsuccess = function(evt) {
      displayActionSuccess("Store cleared");
      displayPubList(store);
    };
    req.onerror = function (evt) {
      console.error("clearObjectStore:", evt.target.errorCode);
      displayActionFailure(this.error);
    };
  }

  function getBlob(key, store, success_callback) {
    var req = store.get(key);
    req.onsuccess = function(evt) {
      var value = evt.target.result;
      if (value)
        success_callback(value.blob);
    };
  }

  /**
   * @param {IDBObjectStore=} store
   */
  function displayPubList(store) {
    console.log("displayPubList");

    if (typeof store == 'undefined')
      store = getObjectStore(DB_STORE_NAME, 'readonly');

    var pub_msg = $('#pub-msg');
    pub_msg.empty();
    var pub_list = $('#pub-list');
    pub_list.empty();
    // Resetting the iframe so that it doesn't display previous content
    newViewerFrame();

    var req;
    req = store.count();
    // Requests are executed in the order in which they were made against the
    // transaction, and their results are returned in the same order.
    // Thus the count text below will be displayed before the actual pub list
    // (not that it is algorithmically important in this case).
    req.onsuccess = function(evt) {
      pub_msg.append('&lt;p&gt;There are &lt;strong&gt;' + evt.target.result +
                     '&lt;/strong&gt; record(s) in the object store.&lt;/p&gt;');
    };
    req.onerror = function(evt) {
      console.error("add error", this.error);
      displayActionFailure(this.error);
    };

    var i = 0;
    req = store.openCursor();
    req.onsuccess = function(evt) {
      var cursor = evt.target.result;

      // If the cursor is pointing at something, ask for the data
      if (cursor) {
        console.log("displayPubList cursor:", cursor);
        req = store.get(cursor.key);
        req.onsuccess = function (evt) {
          var value = evt.target.result;
          var list_item = $('&lt;li&gt;' +
                            '[' + cursor.key + '] ' +
                            '(biblioid: ' + value.biblioid + ') ' +
                            value.title +
                            '&lt;/li&gt;');
          if (value.year != null)
            list_item.append(' - ' + value.year);

          if (value.hasOwnProperty('blob') &amp;&amp;
              typeof value.blob != 'undefined') {
            var link = $('&lt;a href="' + cursor.key + '"&gt;File&lt;/a&gt;');
            link.on('click', function() { return false; });
            link.on('mouseenter', function(evt) {
                      setInViewer(evt.target.getAttribute('href')); });
            list_item.append(' / ');
            list_item.append(link);
          } else {
            list_item.append(" / No attached file");
          }
          pub_list.append(list_item);
        };

        // Move on to the next object in store
        cursor.continue();

        // This counter serves only to create distinct ids
        i++;
      } else {
        console.log("No more entries");
      }
    };
  }

  function newViewerFrame() {
    var viewer = $('#pub-viewer');
    viewer.empty();
    var iframe = $('&lt;iframe /&gt;');
    viewer.append(iframe);
    return iframe;
  }

  function setInViewer(key) {
    console.log("setInViewer:", arguments);
    key = Number(key);
    if (key == current_view_pub_key)
      return;

    current_view_pub_key = key;

    var store = getObjectStore(DB_STORE_NAME, 'readonly');
    getBlob(key, store, function(blob) {
      console.log("setInViewer blob:", blob);
      var iframe = newViewerFrame();

      // It is not possible to set a direct link to the
      // blob to provide a mean to directly download it.
      if (blob.type == 'text/html') {
        var reader = new FileReader();
        reader.onload = (function(evt) {
          var html = evt.target.result;
          iframe.load(function() {
            $(this).contents().find('html').html(html);
          });
        });
        reader.readAsText(blob);
      } else if (blob.type.indexOf('image/') == 0) {
        iframe.load(function() {
          var img_id = 'image-' + key;
          var img = $('&lt;img id="' + img_id + '"/&gt;');
          $(this).contents().find('body').html(img);
          var obj_url = window.URL.createObjectURL(blob);
          $(this).contents().find('#' + img_id).attr('src', obj_url);
          window.URL.revokeObjectURL(obj_url);
        });
      } else if (blob.type == 'application/pdf') {
        $('*').css('cursor', 'wait');
        var obj_url = window.URL.createObjectURL(blob);
        iframe.load(function() {
          $('*').css('cursor', 'auto');
        });
        iframe.attr('src', obj_url);
        window.URL.revokeObjectURL(obj_url);
      } else {
        iframe.load(function() {
          $(this).contents().find('body').html("No view available");
        });
      }

    });
  }

  /**
   * @param {string} biblioid
   * @param {string} title
   * @param {number} year
   * @param {string} url the URL of the image to download and store in the local
   *   IndexedDB database. The resource behind this URL is subjected to the
   *   "Same origin policy", thus for this method to work, the URL must come from
   *   the same origin as the web site/app this code is deployed on.
   */
  function addPublicationFromUrl(biblioid, title, year, url) {
    console.log("addPublicationFromUrl:", arguments);

    var xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);
    // Setting the wanted responseType to "blob"
    // http://www.w3.org/TR/XMLHttpRequest2/#the-response-attribute
    xhr.responseType = 'blob';
    xhr.onload = function (evt) {
                           if (xhr.status == 200) {
                             console.log("Blob retrieved");
                             var blob = xhr.response;
                             console.log("Blob:", blob);
                             addPublication(biblioid, title, year, blob);
                           } else {
                             console.error("addPublicationFromUrl error:",
                                           xhr.responseText, xhr.status);
                           }
                         };
    xhr.send();

    // We can't use jQuery here because as of jQuery 1.8.3 the new "blob"
    // responseType is not handled.
    // http://bugs.jquery.com/ticket/11461
    // http://bugs.jquery.com/ticket/7248
    // $.ajax({
    //   url: url,
    //   type: 'GET',
    //   xhrFields: { responseType: 'blob' },
    //   success: function(data, textStatus, jqXHR) {
    //     console.log("Blob retrieved");
    //     console.log("Blob:", data);
    //     // addPublication(biblioid, title, year, data);
    //   },
    //   error: function(jqXHR, textStatus, errorThrown) {
    //     console.error(errorThrown);
    //     displayActionFailure("Error during blob retrieval");
    //   }
    // });
  }

  /**
   * @param {string} biblioid
   * @param {string} title
   * @param {number} year
   * @param {Blob=} blob
   */
  function addPublication(biblioid, title, year, blob) {
    console.log("addPublication arguments:", arguments);
    var obj = { biblioid: biblioid, title: title, year: year };
    if (typeof blob != 'undefined')
      obj.blob = blob;

    var store = getObjectStore(DB_STORE_NAME, 'readwrite');
    var req;
    try {
      req = store.add(obj);
    } catch (e) {
      if (e.name == 'DataCloneError')
        displayActionFailure("This engine doesn't know how to clone a Blob, " +
                             "use Firefox");
      throw e;
    }
    req.onsuccess = function (evt) {
      console.log("Insertion in DB successful");
      displayActionSuccess();
      displayPubList(store);
    };
    req.onerror = function() {
      console.error("addPublication error", this.error);
      displayActionFailure(this.error);
    };
  }

  /**
   * @param {string} biblioid
   */
  function deletePublicationFromBib(biblioid) {
    console.log("deletePublication:", arguments);
    var store = getObjectStore(DB_STORE_NAME, 'readwrite');
    var req = store.index('biblioid');
    req.get(biblioid).onsuccess = function(evt) {
      if (typeof evt.target.result == 'undefined') {
        displayActionFailure("No matching record found");
        return;
      }
      deletePublication(evt.target.result.id, store);
    };
    req.onerror = function (evt) {
      console.error("deletePublicationFromBib:", evt.target.errorCode);
    };
  }

  /**
   * @param {number} key
   * @param {IDBObjectStore=} store
   */
  function deletePublication(key, store) {
    console.log("deletePublication:", arguments);

    if (typeof store == 'undefined')
      store = getObjectStore(DB_STORE_NAME, 'readwrite');

    // As per spec http://www.w3.org/TR/IndexedDB/#object-store-deletion-operation
    // the result of the Object Store Deletion Operation algorithm is
    // undefined, so it's not possible to know if some records were actually
    // deleted by looking at the request result.
    var req = store.get(key);
    req.onsuccess = function(evt) {
      var record = evt.target.result;
      console.log("record:", record);
      if (typeof record == 'undefined') {
        displayActionFailure("No matching record found");
        return;
      }
      // Warning: The exact same key used for creation needs to be passed for
      // the deletion. If the key was a Number for creation, then it needs to
      // be a Number for deletion.
      req = store.delete(key);
      req.onsuccess = function(evt) {
        console.log("evt:", evt);
        console.log("evt.target:", evt.target);
        console.log("evt.target.result:", evt.target.result);
        console.log("delete successful");
        displayActionSuccess("Deletion successful");
        displayPubList(store);
      };
      req.onerror = function (evt) {
        console.error("deletePublication:", evt.target.errorCode);
      };
    };
    req.onerror = function (evt) {
      console.error("deletePublication:", evt.target.errorCode);
      };
  }

  function displayActionSuccess(msg) {
    msg = typeof msg != 'undefined' ? "Success: " + msg : "Success";
    $('#msg').html('&lt;span class="action-success"&gt;' + msg + '&lt;/span&gt;');
  }
  function displayActionFailure(msg) {
    msg = typeof msg != 'undefined' ? "Failure: " + msg : "Failure";
    $('#msg').html('&lt;span class="action-failure"&gt;' + msg + '&lt;/span&gt;');
  }
  function resetActionStatus() {
    console.log("resetActionStatus ...");
    $('#msg').empty();
    console.log("resetActionStatus DONE");
  }

  function addEventListeners() {
    console.log("addEventListeners");

    $('#register-form-reset').click(function(evt) {
      resetActionStatus();
    });

    $('#add-button').click(function(evt) {
      console.log("add ...");
      var title = $('#pub-title').val();
      var biblioid = $('#pub-biblioid').val();
      if (!title || !biblioid) {
        displayActionFailure("Required field(s) missing");
        return;
      }
      var year = $('#pub-year').val();
      if (year != '') {
        // Better use Number.isInteger if the engine has EcmaScript 6
        if (isNaN(year))  {
          displayActionFailure("Invalid year");
          return;
        }
        year = Number(year);
      } else {
        year = null;
      }

      var file_input = $('#pub-file');
      var selected_file = file_input.get(0).files[0];
      console.log("selected_file:", selected_file);
      // Keeping a reference on how to reset the file input in the UI once we
      // have its value, but instead of doing that we rather use a "reset" type
      // input in the HTML form.
      //file_input.val(null);
      var file_url = $('#pub-file-url').val();
      if (selected_file) {
        addPublication(biblioid, title, year, selected_file);
      } else if (file_url) {
        addPublicationFromUrl(biblioid, title, year, file_url);
      } else {
        addPublication(biblioid, title, year);
      }

    });

    $('#delete-button').click(function(evt) {
      console.log("delete ...");
      var biblioid = $('#pub-biblioid-to-delete').val();
      var key = $('#key-to-delete').val();

      if (biblioid != '') {
        deletePublicationFromBib(biblioid);
      } else if (key != '') {
        // Better use Number.isInteger if the engine has EcmaScript 6
        if (key == '' || isNaN(key))  {
          displayActionFailure("Invalid key");
          return;
        }
        key = Number(key);
        deletePublication(key);
      }
    });

    $('#clear-store-button').click(function(evt) {
      clearObjectStore();
    });

    var search_button = $('#search-list-button');
    search_button.click(function(evt) {
      displayPubList();
    });

  }

  openDb();
  addEventListeners();

})(); // Immediately-Invoked Function Expression (IIFE)
</pre>

<p>{{ LiveSampleLink('Full_IndexedDB_example', "Test the online live demo") }}</p>

<h2 id="Next_step">Next step</h2>

<p>If you want to start tinkering with the API, jump in to the <a href="/en/IndexedDB" title="https://developer.mozilla.org/en/IndexedDB">reference documentation</a> and check out the different methods.</p>

<h2 id="See_also">See also</h2>

<p>Reference</p>

<ul>
 <li><a href="/en/IndexedDB" title="https://developer.mozilla.org/en/IndexedDB">IndexedDB API Reference</a></li>
 <li><a class="external" href="http://www.w3.org/TR/IndexedDB/">Indexed Database API Specification</a></li>
 <li><a href="/en-US/docs/IndexedDB/Using_IndexedDB_in_chrome" title="/en-US/docs/IndexedDB/Using_IndexedDB_in_chrome">Using IndexedDB in chrome</a></li>
</ul>

<p>Tutorials</p>

<ul>
 <li><a class="external" href="http://www.html5rocks.com/tutorials/indexeddb/todo/">A simple TODO list using HTML5 IndexedDB</a><span class="external">. {{Note("This tutorial is based on an old version of the specification and does not work on up-to-date browsers - it still uses the removed <code>setVersion()</code> method.") }}</span></li>
 <li><a href="http://www.html5rocks.com/en/tutorials/indexeddb/uidatabinding/">Databinding UI Elements with IndexedDB</a></li>
</ul>

<p>Related articles</p>

<ul>
 <li><a class="external" href="http://msdn.microsoft.com/en-us/scriptjunkie/gg679063.aspx">IndexedDB — The Store in Your Browser</a></li>
</ul>

<p>Firefox</p>

<ul>
 <li>Mozilla <a class="link-https" href="https://mxr.mozilla.org/mozilla-central/find?text=&amp;string=dom%2FindexedDB%2F.*%5C.idl&amp;regexp=1" title="https://mxr.mozilla.org/mozilla-central/find?text=&amp;string=dom/indexedDB/.*\.idl&amp;regexp=1">interface files</a></li>
</ul>