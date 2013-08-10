---
layout: architect
title: Custom searches
---

Serverseitige Selektion

    Ablauf:
        Query mit Attributen und/oder Punkt
        Resultat GeoJSON mit Attributen und Extent
        OpenLayers macht Update auf Extent der Geometrie
        WMS-Selektionslayer in OpenLayers mit SLD (Filter auf Objekt und Darstellung Selektion)

Custom searches
===============

Implementation von Spezialsuchen.
Schritte für die Suche nach Xxx:

    Model xxx.rb: table_name, SQL
    Mapfile für das Topic SelectionZH: Definition eines entsprechenden Layers xxx
    search_rules.rb (in config/initializers): Eintrag im Objekt SEARCHRULES. So heisst dann der Aufruf: /search/xxx.json?... und die dort definierten Felder kommen im JSON zurück. Box2d(geom) obligatorisch, da bei der Selektion verwendet
    GbXxxSearchComboBox.js: Details zur Abfrage und zum User-Interface der Abfrage
    GbXxxSearchPanel.js: kurze Definition des Suchfensters, Verweis auf die Combobox gbxxxsearchcombobox
    GbSearchTabPanel.js: Eintrag von Gb41.view.search.GbXxxSearchPanel im requires.
    query_config.js: Aufnahme des speziellen Suchpanels im Objekt search_configs
    neue Files ins git aufnehmen (xxx.rb, GbXxxSearchComboBox.js, GbXxxSearchPanel.js

Achtung: Die Sache funktioniert erst, wenn der (lokale) Server neu gestartet wird!

Custom search models
--------------------

app/models/xxx.rb: Search model class inheriting `SearchModel`


Example (`app/models/country.rb`):

    class Country < SearchModel
      self.table_name = 'countries'
      self.primary_key = 'geodb_oid'

      def self.query(fields, params)
        countries = self.select(fields)
        countries = countries.where('name ILIKE ?', "#{params[:name]}")
        {:features => countries, :quality => 0}
      end
    end


Search API
----------

Custom search rules are configured in `config/initializers/search_rules.rb`.

An entry consists of a name identifying the search and a SearchRule constructor.

Sample entry:

    'country' =>
      SearchRule.new( Country,
        %w(ogc_fid name pop_est Box2d(wkb_geometry))
      )


After restart a search query like

    http://0.0.0.0:3000/search/country.json?name=Switzerland

returns the following JSON response:

    {"success":true,"features":[{"ogc_fid":40,"name":"Switzerland","pop_est":"7604467.0","box2d":"BOX(5.95480920400016 45.820718486,10.4666268310001 47.8011660770001)"}],"quality":0}

Interface: Combobox und Panel

-   public/apps/gb41build/app/view/search/GbXxxxxSearchComboBox.js
-   public/apps/gb41build/app/view/search/GbXxxxxSearchPanel.js

Aktivierung

configs/query_config.js
Ergänzung der search_configs mit Topicname und zu aktivierender Klasse

    var search_configs = {
        "TbaSseZH": ["Gb41.view.search.GbManholeSearchPanel"],
        "XXX": ["Gb41.view.search.GbXxxxxSearchPanel"],
        ...
    };

Locate query result
-------------------

Link auf Objekt

Suche nach Parzelle(n) + BFS-Nr:

    Modell erstellen: app/models/parcelarea.rb
    In File config/initializers/search_rules.rb bei "LOCATERULES=" folgende Zeile einfügen:

    'parz' => LocateRule.new('Parcelarea'),

Aufruf: http://web.maps.zh.ch/?locate=parz&locations=261,AU4999

Suche nach GVZ-NR + BFS-Nr:

    Modell erstellen: app/models/gvz.rb
    In File config/initializers/search_rules.rb bei "LOCATERULES=" folgende Zeile einfügen:

    'gvz' => LocateRule.new('Gvz'),

Aufruf: http://web.maps.zh.ch/?locate=gvz&locations=1,1000

Suche nach 1 Attribut:

    In File config/initializers/search_rules.rb bei "LOCATERULES=" eine Zeile einfügen:
    Bsp. Sportanlagen:

    'sportanlagen' => LocateRule.new('SearchModel', 'sportanlagen', 'linkid'),

Aufruf: http://maps.zh.ch/?topic=SportanlagenZH&locate=sportanlagen&locations=30612

Bsp. Forstreviere:

'revier' => LocateRule.new('SearchModel', 'forstreviere', 'forevnr'),

Aufruf: http://maps.zh.ch/?topic=WaldEGZH&locate=revier&locations=508

Bsp. Fassungen:

Aufruf: http://web.maps.zh.ch/?topic=AwelGrundWaMWZH&locate=fassung&locations=d%2017-0007

LocateRule = Struct.new(:model, :layer, :search_field)

LOCATERULES = {
  'parz' => LocateRule.new('Grundstueck'),
  'revier' => LocateRule.new('SearchModel', 'forstreviere', 'forevnr'),
  'gvz' => LocateRule.new('Gvz'),
  'fassung' => LocateRule.new('Fassung'),
  'toponame' => LocateRule.new('Toponamesearch'),
  'sportanlagen' => LocateRule.new('SearchModel', 'sportanlagen', 'linkid'),
  'gwr-suche' => LocateRule.new('SearchModel', 'gwr-suche', 'gwr_aqua'),
  'gemeinden' => LocateRule.new('SearchModel', 'gemeindegrenzen', 'bfs')

}
