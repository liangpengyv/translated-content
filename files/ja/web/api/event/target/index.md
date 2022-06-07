---
title: Event.target
slug: Web/API/Event/target
tags:
  - API
  - DOM
  - Event
  - Property
  - リファレンス
  - delegation
  - target
translation_of: Web/API/Event/target
---
{{ApiRef("DOM")}}

イベントを発生させたオブジェクトへの参照です。 イベントハンドラーがバブリング、またはキャプチャフェーズの間に呼び出されたとき、{{domxref("event.currentTarget")}} とは異なります。

<h2 id="Syntax" name="Syntax">構文</h2>

<pre class="syntaxbox">theTarget = event.target</pre>

<h2 id="Example" name="Example">例</h2>

`event.target` プロパティは、**イベントデリゲーション**を実装するために使用できます。

<pre><code>// リストを作ります
var ul = document.createElement('ul');
document.body.appendChild(ul);

var li1 = document.createElement('li');
var li2 = document.createElement('li');
ul.appendChild(li1);
ul.appendChild(li2);

function hide(e){
  // e.target はクリックされた &lt;li&gt; 要素を参照します
  // これはコンテキスト内の親 &lt;ul&gt; を参照する e.currentTarget とは異なります
  e.target.style.visibility = 'hidden';
}

// リストにリスナーを接続します
// &lt;li&gt; がクリックされた時に発火します
ul.addEventListener('click', hide, false);</code>

</pre>

 

<h2 id="Specifications" name="Specifications">仕様</h2>

 

<table class="standard-table">
 <tbody>
  <tr>
   <th>仕様</th>
   <th>状態</th>
   <th>コメント</th>
  </tr>
  <tr>
   <td>{{SpecName("DOM WHATWG", "#dom-event-target", "Event.target")}}</td>
   <td>{{Spec2("DOM WHATWG")}}</td>
   <td> </td>
  </tr>
  <tr>
   <td>{{SpecName("DOM4", "#dom-event-target", "Event.target")}}</td>
   <td>{{Spec2("DOM4")}}</td>
   <td> </td>
  </tr>
  <tr>
   <td>{{SpecName("DOM2 Events", "#Events-Event-target", "Event.target")}}</td>
   <td>{{Spec2("DOM2 Events")}}</td>
   <td>初回定義</td>
  </tr>
 </tbody>
</table>

<h2 id="Browser_compatibility" name="Browser_compatibility">ブラウザー実装状況</h2>

{{Compat("api.Event.target")}}

<h2 id="Compatibility_notes" name="Compatibility_notes">互換性のための注記</h2>

IE 6-8 では、イベントモデルが異なります。イベントリスナーは、非標準の {{domxref('EventTarget.attachEvent')}} メソッドでアタッチされます。このモデルでは、イベントオブジェクトは `target` プロパティの代わりに、{{domxref('Event.srcElement')}} プロパティを持っており、意味的には `event.target` と同じです。

<pre class="brush: js">function hide(e) {
  // IE6-8 をサポート
  var target = e.target || e.srcElement;
  target.style.visibility = 'hidden';
}
</pre>

<h2 id="See_also" name="See_also">関連項目</h2>

<ul>
 <li><a href="/ja/docs/Web/API/Event/Comparison_of_Event_Targets">イベントターゲットの比較</a></li>
</ul>