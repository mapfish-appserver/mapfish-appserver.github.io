---
layout: architect
title: Tipps and hints
---

Layer filters
-------------

UMN mapfile:

    LAYER
      NAME "aktuelle-neophyten-art"
      ...
      METADATA
        ...
        "default_filter_aktuelle_neophyten_art" "*" # Wert, falls Parameter filter_aktuelle_neophyten_art fehlt
      END
      ...
      FILTER "'%filter_aktuelle_neophyten_art%' = '*' or symb4 = '%filter_aktuelle_neophyten_art%'"
      VALIDATION
        "filter_aktuelle_neophyten_art" '^[0-9]+$' # String mit mindestens einer Zahl
      END
      ...
    END

<!--
Mapserver Run-time Substitution http://mapserver.org/cgi/runsub.html

WMS GetMap mit zusätzlichem Parameter "filter_<layername>"
z.B. http://web.wms.zh.ch/FnsLWeditZH?SERVICE=WMS&REQUEST=GetMap&filter_aktuelle_neophyten_art=2110&...


    '%filter_aktuelle_neophyten_art%' = '*': Alle Features anzeigen, wenn Parameter filter_aktuelle_neophyten_art fehlt
    symb4 = '%filter_aktuelle_neophyten_art%': Features nach Attribut symb4 filtern

Konfiguration für Layer mit Filtern je nach Topic
public/javascripts/layer_filter_config.js

Serverseite benutzt Mapserver als CGI, wenn WMS Parameter "filter_*" vorhanden ist.
-->


<!--
Massstabsunabhaengige Selektion

Damit die Markierung eines Features in der Karte immer angezeigt wird (unabhängig vom Massstabsbereich des Layers), wie folgt vorgehen:

- Darstellungsmassstab des Layers festsetzen
- Rake Task durchführen -> Masstabsbereich für Infoabfrage wird in Layers Tabelle eingefüllt
- nach Rake Task in Map-File Massstabsbereich auskommentieren -> Markierung eines Features wird in der Karte masstabsunabhängig angezeigt
-->
