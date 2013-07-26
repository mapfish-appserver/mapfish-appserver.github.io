---
layout: architect
title: Mapfile Import
---

Mapfile conventions
===================

File format
-----------

-   UTF-8 encoding (without BOM)
-   WEB METADATA "wms_encoding" "UTF-8"
-   Indentation with two spaces (no tabs)


Basic structure
---------------

Mapfiles have to deliver a correct WMS service. See http://mapserver.org/ogc/wms_server.html

Topic and layer information are extracted from regular mapfile objects and additional METADATA entries.

Used Mapfile entries:

-   MAP NAME: Name of topic (without umlauts and special characters except '-')
-   LAYER NAME: Name of layer (without umlauts and special characters except '-')

<!--
Minimum Scale:

    auf Topic-Stufe in METADATA eintragen:
    "gb_minscale" "500"
-->

Queriable layers
----------------

<!--
    Nur abfragbare Layer haben einen TEMPLATE Eintrag
    wms_include_items muss anzuzeigende Attribute enthalten
    Für alle Attribute sollte ein Eintrag gml_xxx_alias gemacht werden
    soll bei einer Infoabfrage das Legendenfragment angezeigt werden, aber keine Attributabfrage, so muss das map-File die folgenden Einträge beinhalten
-->
    "wms_include_items"  "oid"
    "gml_include_items"  "oid"
    "gml_lk_text_alias"  "oid"

    TEMPLATE                "blank.html"

<!--
Das Query-Feld "oid" kann später wieder aus der Datei _LAYERNAME_info.html.erb entfernt werden, damit nur das Legendenfragment angezeigt wird.

Gruppierung/Legende:

    Layer, welche im Layerbaum gruppiert werden sollen, werden mit einem gemeinsamen wms_group_title versehen
    Darstellungsklassen werden benannt, damit eine Legende automatisch erstellt werden kann. Beispiel:
    CLASS
    NAME "Lichte Wälder"
    STYLE
    ...
    END
    END
-->


Layer properties
----------------

To set the search radius for info queries, add a METADATA entry "gb_searchdistance". Example:

    "gb_searchdistance" "5"

<!--
    Per Default werden mit dem Rake-Task die folgenden Suchradien gesetzt:
    when MS_LAYER_POINT then 50
    when MS_LAYER_LINE then 50
    when MS_LAYER_POLYGON then 0
-->


Display order in layer tree: 

    "gb_toc_sort" "0"
<!--
    falls Reihenfolge nicht analog Layerreihenfolge in Mapfile sein soll
-->

Selection style:

Default styling can be overwritten with a METADATA entry "gb_selection_style". Examples:

    "gb_selection_style" "<PolygonSymbolizer><Fill><CssParameter name='fill'>#ff0090</CssParameter></Fill></PolygonSymbolizer>"
    "gb_selection_style" "<LineSymbolizer><Stroke><CssParameter name='stroke'>#ff0090</CssParameter><CssParameter name='stroke-width'>10.00</CssParameter></Stroke></LineSymbolizer>"
    "gb_selection_style" "<PointSymbolizer><Graphic><Mark><WellKnownName>circle</WellKnownName><Fill><CssParameter name='fill'>#ff0090</CssParameter></Fill></Mark><Size>45.0</Size></Graphic></PointSymbolizer>"


Import topic from Mapfile
=========================


Import the mapfile into a new topic:

    rake mapfile:import_topic MAPFILE=mapconfig/maps.example.com/naturalearth.map SITE=maps.example.com

<!--
Mapfile Import

Für das Ausführen eines Mapfile Imports wird die Ruby MapScript Bibliothek benoetigt. Diese ist im Moment nur unter Linux binär verfügbar (Paket libmapscript-ruby).

    rake mapfile:import_topic MAPFILE=mapserver/maps/intranet/FnsLWZH.map

Achtung: Es werden alle DB-Eintraege des Topics und die Legenden- und Infofragmente auf dem produktiven Server ersetzt!

Ev. Name des neu generierten Topics am Ende der Topics-Tabelle verifizieren.


[ Topic zu Kategorie zuordnen ]

Tabelle categories: passende Kategorie suchen
Tabelle Topics: id des neuen Themas heraussuchen
Tabelle categories_topics: categorie_id und topic_id eintragen

[ Topic zu Organisation zuordnen ]

Tabelle organisations: passende Organisation suchen, id herausschreiben
Tabelle Topics: id der Organisationeinheit in Feld organisation_id eintragen

[ Berechtigungen setzen ]

in Permissions-Tabelle in DB müssen Berechtigungen gesetzt werden, damit die Karte in der Kartenauswahl angezeigt wird.
Administrationsoberfläche: web.maps.zh.ch/gbadmin (user: admin) -> Permissions anwählen.
Tabelle nach Kriterien filtern und nach Sequence ordnen

Topic freischalten für Internet-User:
Layer FaBoFFFZH/* show 1 false internet
Topic FaBoFFFZH show 1 false internet

Topic freischalten für Intranet-User:
Topic FaBoFFFZH show 1 false intranet
Layers per Default offen

WMS des Topics freischalten für Intranet-User:
per Default offen

WMS des Topics freischalten für Internet-User:
Wms kkgeo_gws_zh show 4 false internet

Passwortgeschütze Topics / WMS:
Topic avwms show 0 false avview
Wms avwms show 3 false avview

User (-> Gruppe) -> Rolle -> Topic

Applikation neu starten (damit Kartenauswahl aktualisiert wird):

-->

<!--
Symbolisieren von externen WMS mit SLD

Ein externer WMS kann mit einem SLD neu symbolisiert werden.

Dazu muss unter METADATA im entsprechenden Layer des Mapfiles folgende Zeile eingefügt werden:
"wms_sld_url" "http://URL_TO_SLD"

Das SLD im xml-Format muss an der angegebenen Stelle auf dem Web-Server abgelegt werden.
Edit
Beispiel für Smaragd-Gebiete in Bundesinvetare (BundInvZH.map):

Originale Symbolisierung (ohne SLD) -> hellgrüne Flächenfüllung, dunkelgrüne dünne Umrandung
Neue Symbolisierung (mit SLD) -> hellrote Flächenfüllung, dunkelrote dicke Umrandung
Edit
Mapfile

  LAYER
    NAME "smaragd-gebiete"
    STATUS on
    TYPE RASTER
    CONNECTION "http://wms.geo.admin.ch/?"
    CONNECTIONTYPE WMS
    METADATA
     "wms_srs"                 "EPSG:21781"
     "wms_name"                "ch.bafu.schutzgebiete-smaragd"
     "wms_server_version"      "1.1.1"
     "wms_format"              "image/png"
     "wms_sld_url"             "http://84.75.158.193/sld_polygon1_simple.xml"
     "wms_title"               "Smaragd-Gebiete"
     "wms_group_title"         "Schutzgebiete und Biotopinventare"
     "gb_toc_sort"             "92"
    END
  END

Edit
SLD unter der Adresse "http://84.75.158.193/sld_polygon1_simple.xml"

<StyledLayerDescriptor xmlns="http://www.opengis.net/sld" xmlns:se="http://www.opengis.net/se" xmlns:ogc="http://www.opengis.net/ogc" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="1.1.0" xsi:schemaLocation="http://www.opengis.net/sld http://schemas.opengis.net/sld/1.1.0/StyledLayerDescriptor.xsd">
  <NamedLayer>
    <se:Name>ch.bafu.schutzgebiete-smaragd</se:Name>
    <UserStyle>
      <se:Name>xxx</se:Name>
      <se:FeatureTypeStyle>
        <se:Rule>
          <se:polygonSymbolizer>
            <se:fill>
              <se:SvgParameter name="fill">#ff6432</se:SvgParameter>
            </se:fill>
            <se:Stroke>
              <se:SvgParameter name="stroke">#ff0000</se:SvgParameter>
              <se:SvgParameter name="stroke-width">5</se:SvgParameter>
            </se:Stroke>
          </se:polygonSymbolizer>
        </se:Rule>
      </se:FeatureTypeStyle>
    </UserStyle>
  </NamedLayer>
</StyledLayerDescriptor>
-->