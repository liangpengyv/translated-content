---
title: border-left
slug: Web/CSS/border-left
translation_of: Web/CSS/border-left
---
<p>{{CSSRef}}</p>

<h2 id="Resumen" name="Resumen">Resumen</h2>

<p>El <code>borde_izquierdo</code> es una propiedad rápida para poner el ancho, estilo y color del borde izquierdo de un elemento. Esta propiedad puede ser usada para poner los valores de uno o mas de : {{ Cssxref("border-left-width") }}, {{ Cssxref("border-left-style") }}, {{ Cssxref("border-left-color") }}. Valores omitidos son puestos a su valor inicial.</p>

<p>{{cssinfo}}</p>

<h2 id="Sintaxis" name="Sintaxis">Sintaxis</h2>

<pre class="eval">border-left: [<em>border-width</em> || <em>border-style</em> || <em>border-color</em> | <em>inherit</em>] ;
</pre>

<h3 id="Valores" name="Valores">Valores</h3>

<dl>
 <dt>border-width</dt>
 <dd>See {{ Cssxref("border-width") }}.</dd>
 <dt>border-style </dt>
 <dd>See {{ Cssxref("border-style") }}.</dd>
 <dt>border-color </dt>
 <dd>See {{ Cssxref("border-color") }}.</dd>
</dl>

<h2 id="Ejemplos" name="Ejemplos">Ejemplos</h2>

<p>element {</p>

<pre class="eval">    border-left: 1px solid #000;
</pre>

<p>}</p>

<h2 id="Notas" name="Notas">Notas</h2>

<p>Si las reglas no especifican un color de borde, el borde tendrá la propiedad {{ Cssxref("color") }}</p>

<h2 id="Specifications" name="Specifications">Specifications</h2>

<table class="standard-table">
 <thead>
  <tr>
   <th scope="col">Specification</th>
   <th scope="col">Status</th>
   <th scope="col">Comment</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>{{ SpecName('CSS3 Backgrounds', '#border-left', 'border-left') }}</td>
   <td>{{ Spec2('CSS3 Backgrounds') }}</td>
   <td>No direct changes, though the modification of values for the {{ cssxref("border-left-color") }} do apply to it.</td>
  </tr>
  <tr>
   <td>{{ SpecName('CSS2.1', 'box.html#propdef-border-left', 'border-left') }}</td>
   <td>{{ Spec2('CSS2.1') }}</td>
   <td>No significant changes</td>
  </tr>
  <tr>
   <td>{{ SpecName('CSS1', '#border-left', 'border-left') }}</td>
   <td>{{ Spec2('CSS1') }}</td>
   <td>Initial definition</td>
  </tr>
 </tbody>
</table>

<h2 id="Browser_compatibility" name="Browser_compatibility">Browser compatibility</h2>

{{Compat("css.properties.border-left")}}