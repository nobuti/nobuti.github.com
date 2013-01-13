---
layout: post
title: "CSS en listas numeradas"
date: 2013-01-13 12:25
comments: true
categories: 
---
En un proyecto en el que estoy trabajando, me he tenido que enfrentar a una lista enumerada, donde los índices tenían un estilo distinto al contenido del item.

El enfoque tradicional siempre ha sido (aquí es cuando decís que vosotros no lo haceis así y yo quedo como un tonto) tratar esta tipología como una lista no ordenada y estilar el número a fuego dentro del elemento li. Esto realmente funciona, pero tiene el principal problema de que semánticamente deja bastante que desear. Es lo que se suele denominar «code smell».

Esta vez, con un poco más de tiempo, he sido capaz de encontrar una solución bastante más usable y semánticamente correcta, que permite modificar los estilos a nuestro antojo. [Podeís ver la demo aquí](http://codepen.io/nobuti/pen/efsjw).

<pre><code class="language-css">
  ol {
    margin: 0;
    padding: 0;
    counter-reset:li;
  }
  ol > li {
    position:relative;
    list-style:none;
    margin-bottom: 10px;
  }
  ol > li:before {
    content:counter(li); 
    counter-increment:li; 
    font-weight:bold;
    background: #141414;
    color: #fff;
    padding: 2px 10px;
    margin-right: 5px;
  }
</code></pre>

Primero reseteamos el comportamiento normal de uns lista ordenada con **list-style:none;**. 

**counter-reset** and **counter-increment**. Yo no conocía [estas propiedades](http://www.w3.org/TR/CSS21/generate.html#counters). Crean la referencia inicial para mostrar el «conteo». Ambas admiten un elemento a contar y un número. **counter-reset** establece el primer valor, a partir de donde vamos a empezar a contar. Por defecto este valor es 0. Y **counter-increment** el valor de cada iteración, que por defecto es 1. En ambos casos el elemento de referencia es **li**, que son los elementos que queremos contar.

Utilizando el pseudo elemento **:before** y [la función **counter(element)**](http://www.w3.org/TR/CSS21/generate.html#propdef-content) del elemento content, mostramos los números. En este caso, solo muestro el número, pero si quisiera mostrar el número seguido de un punto lo podríamos hacer así:

<pre><code class="language-css">
  ol > li:before {
    content:counter(li)". "; 
  }
</code></pre>

Lo dicho, aquí [podeís jugar con los distintos valores](http://codepen.io/nobuti/pen/efsjw). Espero que os haya parecido interesante.
