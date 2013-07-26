---
layout: architect
title: Legends and query results
---

Legends and query results
=========================

Template files
--------------

When importing a new layer running the rake task, default legend and query templates are generated and put into `app/views/topics/custom/auto/` and `app/views/layers/custom/<topicname>/auto/`. Images for the legend are stored in `public/images/custom/<topicname>/<layername><xx>.png`.

If you import a topic called `naturalearth` with one layer `countries` for example the following files are created:

-   `app/views/layers/custom/naturalearth/auto/_countries_legend.html.erb`: Legend partial
-   `app/views/layers/custom/naturalearth/auto/_countries_info.html.erb`: Info query result partial
-   `app/views/topics/custom/auto/_naturalearth_legend.html.erb`:  Legend template for topic
-   `app/views/topics/custom/_naturalearth_info.html.erb`: Info query header for topic

ERB
---

The generated templates are using the ERB templating language. ERB copies the text portions of the template directly to the generated document, and only processes code that is identified by markers. Most ERB templates only use a combination of two tag markers, each of which cause the enclosed code to be handled in a particular way.

A tag with an equals sign indicates that enclosed code is an expression, and that the renderer should substitute the code element with the result of the code (as a string) when it renders the template. Use an expression to embed a line of code into the template, or to display the contents of a variable:

    Hello, <%= @name %>.
    Today is <%= Time.now.strftime('%A') %>.

Comment markers use a hash sign:

    <%# This is just a comment %>

(Excerpt form http://www.stuartellis.eu/articles/erb/)


Customizing
-----------

For customizing these files, you can put templates with the same file name in `app/views/topics/custom/` resp. `app/views/layers/custom/<layername>/`. You should not edit the generated files, because these will be overwritten when running the rake task again.

<!--
More hints:

Achtung: für die Legendenbilder gibt es keinen auto-Ordner! Falls Bilder bearbeitet werden, sollen diese unter einem anderen Namen abgespeichert werden, ansonsten wird bei erneutem Ausführen des Rake-Tasks das Bild überschrieben!
Edit

Falls mit dem Rake-Task im Auto-Ordner ein Info-Abfrage File erstellt wird, dieses aber nicht angezeigt werden soll (auch nicht der Titel, falls das File leer ist), einfach ausserhalb des Auto-Ordners ein *_info_leer.html.erb File erstellen. Dies bewirkt, dass das im Auto-Ordner abgelegte File nicht angezeigt wird!

Falls in Info-Abfrage auf Felder zugegriffen werden soll, welche in map-File nicht in wms_include_items aufgeführt sind, so müssen diese in der Layers-Tabelle in den Feldern ident_fields und alias_fields angegeben sein, damit sie für Info-Abfrage verwendet werden können.

Der Layertitel in der Infoabfrage kann im DB-Feld (Tabelle Layers) "title" überschrieben werden!

-->
