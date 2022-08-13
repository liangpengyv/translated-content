---
title: Element.ongotpointercapture
slug: Web/API/Element/gotpointercapture_event
tags:
  - API
  - Controlador
  - DOM
  - Elemento
  - Eventos Puntero
  - Propiedad
  - Referencia
translation_of: Web/API/GlobalEventHandlers/ongotpointercapture
translation_of_original: Web/API/Element/ongotpointercapture
original_slug: Web/API/GlobalEventHandlers/ongotpointercapture
---
<p>{{ ApiRef("DOM") }}</p>

<p><code>ongotpointercapture</code> es una propiedad {{event("Event_handlers", "event handler")}} de la interfaz {{domxref("Element")}}  <span id="result_box" lang="es"><span>que devuelve</span> <span>el controlador de eventos</span> <span>(función</span><span>)</span> <span>para el</span></span> evento tipo {{event("gotpointercapture")}}.</p>

<h2 id="Syntax" name="Syntax">Síntasix</h2>

<pre class="eval">var gotCaptureHandler = target.ongotpointercpature;
</pre>

<h3 id="Return_Value" name="Return_Value">Valor de Retorno</h3>

<dl>
 <dt><code>gotCaptureHandler</code></dt>
 <dd><span id="result_box" lang="es"><span>El controlador de eventos</span>  <span>gotpointercapture</span> para el <span>elemento target</span><span>.</span></span></dd>
</dl>

<h2 id="Ejemplo">Ejemplo</h2>

<pre class="brush: js">&lt;html&gt;
&lt;script&gt;
function overHandler(ev) {
 // Determine the target event's gotpointercapture handler
 var gotCaptureHandler = ev.target.ongotpointercapture;
}
function init() {
 var el=document.getElementById("target");
 el.onpointerover = overHandler;
}
&lt;/script&gt;
&lt;body onload="init();"&gt;
&lt;div id="target"&gt; Touch me ... &lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;
</pre>

<h2 id="Specifications" name="Specifications">Especificaciones</h2>

<table class="standard-table">
 <tbody>
  <tr>
   <th scope="col">Especificación</th>
   <th scope="col">Estado</th>
   <th scope="col">Comentario</th>
  </tr>
  <tr>
   <td>{{SpecName('Pointer Events 2','#widl-Element-ongotpointercapture', 'ongotpointercapture')}}</td>
   <td>{{Spec2('Pointer Events 2')}}</td>
   <td>Versión no estable.</td>
  </tr>
  <tr>
   <td>{{SpecName('Pointer Events', '#widl-Element-ongotpointercapture', 'ongotpointercapture')}}</td>
   <td>{{Spec2('Pointer Events')}}</td>
   <td>Definición inicial.</td>
  </tr>
 </tbody>
</table>

<h2 id="Browser_compatibility" name="Browser_compatibility">Compatibilidad en los Navegadores</h2>

{{Compat("api.GlobalEventHandlers.ongotpointercapture")}}

<h2 id="See_also" name="See_also">Ver también</h2>

<ul>
 <li>{{ event("gotpointercapture") }}</li>
</ul>