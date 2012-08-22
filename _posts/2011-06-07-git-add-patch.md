---
layout: post
title: git add --patch
description: O cómo trabajar de manera incremental con Git
---

Aprovechando mi temporada de tiempo libre me entretuve leyendo algunos [artículos de Ryan Tomayko](http://tomayko.com/writings/) y hubo uno que especialmente llamó mi atención: [The Thing About Git](http://tomayko.com/writings/the-thing-about-git). En este artículo Ryan presenta una de las tantas mejoras que Git tiene respecto a otros VCS que yo desconocía y que la podría haber utilizado cientos de veces y ahora sin dudas haré.

Quienes tengan más experiencia en Git o hayan leído la [página del manual de `git-add(1)`](http://www.kernel.org/pub/software/scm/git/docs/git-add.html) seguramente estarán al tanto del modificador `--patch`, que permite resolver el caso cuando modificamos un archivo con dos cambios no relacionados entre si y queremos hacer `commit`s separados. Supongamos comenzamos con el siguiente archivo `calculator.rb`:

{% highlight ruby lineno %}
module Calculator
end
{% endhighlight %}

Lo agregamos al repositorio y hacemos el `commit` inicial:

<pre>
$ git ci -m "Example file for git add --patch article"
[drafts/git-add-patch 3f2a0f0] Example file for git add --patch article
 1 files changed, 2 insertions(+), 0 deletions(-)
 create mode 100644 examples/calculator.rb
</pre>

Luego trabajamos en el archivo y agregamos dos métodos que nos gustaría introducir en `commit`s separados:

{% highlight ruby lineno %}
module Calculator
  def add a, b
    a + b
  end

  def substract a, b
    a - b
  end
end
{% endhighlight %}

Normalmente acá mi solución era copiar los cambios, hacer un `git reset HEAD exmaples/calculator.rb` y aplicar los cambios de a uno, una solución cualquier cosa menos elegante u óptima. Acá es donde entra la magia de `git add --patch`: este modificador nos permite, de un modo interactivo, ver los cambios que se van a aplicar y elegir cuales queremos que se pongan en el staging area. Veamos el ejemplo:

{% highlight diff %}
$ git --no-pager diff examples/calculator.rb
diff --git a/examples/calculator.rb b/examples/calculator.rb
index 50a3da7..748f253 100644
--- a/examples/calculator.rb
+++ b/examples/calculator.rb
@@ -1,2 +1,9 @@
 module Calculator
+  def add a, b
+    a + b
+  end
+
+  def substract a, b
+    a - b
+  end
 end
{% endhighlight %}

Ahora yo quiero hacer un `commit` para el método `add` y luego uno para el método `substract`. Entonces ejecuto:

{% highlight diff %}
$ git add --patch examples/calculator.rb
diff --git a/examples/calculator.rb b/examples/calculator.rb
index 50a3da7..748f253 100644
--- a/examples/calculator.rb
+++ b/examples/calculator.rb
@@ -1,2 +1,9 @@
 module Calculator
+  def add a, b
+    a + b
+  end
+
+  def substract a, b
+    a - b
+  end
 end
Stage this hunk [y,n,q,a,d,/,e,?]?
{% endhighlight %}

Notar la última línea, donde nos pregunta si deseamos copiar todo ese cambio al staging, junto a otras opciones, que son las siguientes:

<pre>
Stage this hunk [y,n,q,a,d,/,e,?]?
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk nor any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk nor any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
k - leave this hunk undecided, see previous undecided hunk
K - leave this hunk undecided, see previous hunk
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
</pre>

Podríamos usar la opción `s` para automáticamente dividir los cambios en fracciones menores, pero en este caso esa opción no funciona dado que los cambios son contiguos, entonces utilizaremos la opción `e` que nos permite indicar los cambios automáticamente:

<pre>
# Manual hunk edit mode -- see bottom for a quick guide
@@ -1,2 +1,9 @@
 module Calculator
+  def add a, b
+    a + b
+  end
+
+  def substract a, b
+    a - b
+  end
 end
# ---
# To remove '-' lines, make them ' ' lines (context).
# To remove '+' lines, delete them.
# Lines starting with # will be removed.
#
# If the patch applies cleanly, the edited hunk will immediately be
# marked for staging. If it does not apply cleanly, you will be given
# an opportunity to edit again. If all lines of the hunk are removed,
# then the edit is aborted and the hunk is left unchanged.
</pre>

Borramos las líneas correspondientes al método `substract` y nos fijamos el `status` y los cambios del proyecto:

<pre>
$ git status
# On branch drafts/git-add-patch
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   examples/calculator.rb
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   examples/calculator.rb
</pre>

Si queremos podemos ver las diferencias que se van a introducir:

{% highlight diff %}
$ git --no-pager diff --cached examples/calculator.rb
diff --git a/examples/calculator.rb b/examples/calculator.rb
index 50a3da7..608a374 100644
--- a/examples/calculator.rb
+++ b/examples/calculator.rb
@@ -1,2 +1,5 @@
 module Calculator
+  def add a, b
+    a + b
+  end
 end
{% endhighlight %}

Luego hacemos el primer `commit`:

<pre>
$ git commit -m "Add add method"
[drafts/git-add-patch 208a6d8] Add add method
 1 files changed, 3 insertions(+), 0 deletions(-)
</pre>

Y por último analizamos las diferencias para el segundo `commit`:

{% highlight diff %}
$ git --no-pager diff examples/calculator.rb
diff --git a/examples/calculator.rb b/examples/calculator.rb
index 608a374..748f253 100644
--- a/examples/calculator.rb
+++ b/examples/calculator.rb
@@ -2,4 +2,8 @@ module Calculator
   def add a, b
     a + b
   end
+
+  def substract a, b
+    a - b
+  end
 end
{% endhighlight %}

Como está todo bien introducimos los cambios:

<pre>
$ git commit -m "Add substract method" examples/calculator.rb
[drafts/git-add-patch 89b9608] Add substract method
 1 files changed, 4 insertions(+), 0 deletions(-)
</pre>

¡Listo! Ya separamos los cambios en dos `commit` independientes. Una demostración más de las ventajas de Git respecto a otros VCS.
