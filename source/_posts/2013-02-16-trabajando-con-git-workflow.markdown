---
layout: post
title: "Trabajando con Git: Workflow"
date: 2013-02-16 06:58
comments: true
categories: 
---
TL;TD Git mola. Y así es como lo veo yo.

##Introducción
El objetivo de este post no es convencerte de usar Git. Úsalo y punto. Ni tampoco se trata de una introducción para usarlo. [Hay muchas y muy buenas](http://rogerdudler.github.com/git-guide/index.es.html). En lugar de eso, voy a tratar de plasmar algunas ideas de como usarlo en flujos de trabajos aplicables a pequeños equipos de desarrollo. Es lo que se denomina *workflow*. 

Después de leer bastante sobre el tema, así es como yo considero que es la mejor forma de trabajar con Git cuando existen más personas aparte de ti trabajando en un mismo proyecto. Aunque en general es buena práctica aplicarlo siempre.

##Workflow
En este workflow, siempre existen en todo momento dos ramas principales: **master** y **develop**.

*master* representa la versión última y más estable de tu proyecto. Siempre siempre siempre pasas a producción desde esta rama.

*develop* contiene el desarrollo, los cambios que están en proceso, las mejoras del proyecto o las nuevas versiones. Contiene una versión de tu proyecto que no necesariamente tiene que estar lista para pasar a producción.

Desde la rama *develop*, se crean ramas para trabajar en fixes y desarrollos individuales. Es en estas ramas donde trabajan los desarrolladores. Una vez que un nuevo feature/bugfix está listo, y testeado, se hace merge con la rama *develop*. En este punto donde entran los test de compatibilidad con el resto de código de tus otros compañeros de equipo. 

Una vez que la rama *develop* esté estable y se alcance el objetivo/iteración deseado del desarrollo, se hace un **merge con la rama master**. 

Si nos fijamos bien, en todo el proceso se crean 3 niveles de ramas con diferente estabilidad. Conforme subamos de nivel, nuestro código se presupone más estable. Al hacer deploy desde la rama *master*, la rama de primer nivel, nos estamos asegurando (si hacemos las cosas medio bien) una alta probabilidad de que nuestro código no contenga errores.

Veamos un ejemplo de cómo aplicar este workflow paso a paso. Pongamos que estamos trabajando en un proyecto y en un momento determinado, detectamos un bug o el cliente pide un cambio.

Creamos una rama partiendo de la rama *develop*. En este caso la llamamos *changebugfix*.
<pre><code class="language-git">git checkout -b changebugfix
</code></pre>

Trabajas en los cambios a nivel de código, hasta que consigas resolver el problema o implementar la nueva funcionalidad. 
<pre><code class="language-git">touch file.rb && git add file.rb
…
git add file.rb
git commit -m "Adding new feature"
touch lib/gem.rb && git add lib/gem.rb
…
git add lib/gem.rb
git commit -m "Adding new lib"
</code></pre>

Merge *changebugfix* en la rama *develop*. Y testing, testing, testing.
<pre><code class="language-git">git checkout develop
git pull
git merge changebugfix
</code></pre>

En este punto, descubres que algo no va bien, probablemente no sea culpa tuya, nunca lo es. Pero te toca seguir trabajando en tu rama *changebugfix* para lidiar con este conflicto en el código. Vuelta al punto 2.

Todo funciona perfecto. Es hora de hacer el merge definitivo y borrar la rama de desarrollo:
<pre><code class="language-git">git checkout develop
git merge changebugfix
git push origin develop
git branch -d changebugfix
</code></pre>

Es tiempo de release. El cliente quiere ver lo que hay y poner su logo más grande.
<pre><code class="language-git">git checkout master
git merge develop
</code></pre>

Como podéis ver, además de tener un proceso, es fundamental tener una buena comunicación entre los miembros del equipo.

##Rebase
Como siempre trabajamos en nuestra rama local de desarrollo, eso nos permite entre otras cosas, hacer *rebase*. Que no es más que una reorganización, pudiendo editar, agrupar y reordenar commits. En última instancia estamos modificando el histórico de git. Y como se intuye, esto puede ser peligroso, así que hay que tener cuidado.

Yo hago commits continuamente, mucho. Si te pasa como a mi, probablemente termines una rama con algo parecido a esto:

<pre><code class="language-git">Init feature
Add media query mixin
Fix header link
Fix media query for devices
Add missing semi colon
Refactor
Fix typo
Bugfixing in some stuff
Fix link
</code></pre>

Cuando alguien en tu equipo de desarrollo revisa el proyecto, es bastante probable que todo le parezca carente de sentido alguno. Y pensandolo friamente, no es que me deje en buen lugar precisamente ir aireando los errores tontos que cometes al programar :)

Sería más lógico agrupar todos los commits en aquellos que sean más importantes para el desarrollo. En nuestro caso, podríamos agrupar todos los commits en uno solo: 'Add responsive request for devices'.

Para hacer esto, justo antes de hacer el *merge* de nuestra rama con *develop*, ejecutamos un rebase:

<pre><code class="language-git">git rebase develop -i
</code></pre>

El parámetro -i (interactively) nos permite de forma fácil modificar los commits individuales. Esto nos mostrará por terminal algo como esto:

<pre><code class="language-git">pick 76012fa2 Init feature changebugfix
pick 1ed9323a Add media query mixin
pick 3ae44d64 Fix header link
pick 61b1ce34 Fix media query for devices
pick 120ad36d Add missing semi colon
pick 89c0a230 Refactor
pick 19ac34e6 Fix typo
pick 239d0ae2 Bugfixing in some stuff
pick 76012fa2 Fix link
</code></pre>

La palabra *pick* la podemos cambiar por diferentes comandos, entre los que más se usan están **r (reword)** para editar el mensaje del commit y **s (squash)** para agrupar el commit con su predecesor. En nuestro caso terminamos teniendo algo parecido a esto:

<pre><code class="language-git">pick 76012fa2 Init feature changebugfix
s 1ed9323 Add media query mixin
s 3ae44d6 Fix header link
s 61b1ce3 Fix media query for devices
s 120ad36 Add missing semi colon
s 89c0a23 Refactor
s 19ac34e Fix typo
s 239d0ae Bugfixing in some stuff
s 76012fa Fix link
</code></pre>

Como el primer comando es *s* al guardar nos preguntará el mensaje para el commit general. Es entonces cuando cambiaremos el commit por 'Add responsive request for devices'. 

Si ahora hacemos

<pre><code class="language-git">git log --oneline
</code></pre>

Veremos un solo commit más amigable y comprensible, en lugar de 7 commits con etiquetas menos intuitivas. 

<pre><code class="language-git">git log --oneline
ef3c709 Add responsive request for devices
</code></pre>

Despues de hacer rebase, continuamos con el merge sobre la rama develop. Recordemos que el rebase solo lo usamos como una forma de organizar y simplificar el histórico de git.

<pre><code class="language-git">git checkout develop
git merge changebugfix
git branch -d changebugfix
</code></pre>
