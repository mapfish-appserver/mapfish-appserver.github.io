---
layout: architect
title: Editing
---

Editing
-------

<!--
Betroffene Dateien:

Ruby-Seite

config/routes.rb

resources-Eintrag in "namespace :geo"

app/models/geo/...

Model erstellen für Editierlayer

app/controllers/geo/...

Controller erstellen (fast leer)


JavaScript-Seite

app/view/edit/...

Panel(s) mit Architect erstellen (Formulare mit zu erfassenden Items der einzelnen Layer).
Bei mehreren Panels braucht's zudem ein übergeordnetes Formular.

app/controller/edit

Für jedes Panel braucht's einen Controller.
-->

<!--
Hinzufügen eines editierbaren Layers auf Serverseite

Models und Controller werden im jeweiligen Unterverzeichnis geo/ hinzugefügt.

z.B. für Tabelle aln_fns_apliwa_pflege_f

Model

neues Model app/models/geo/aln_fns_apliwa_pflege_f.rb

module Geo
  class AlnFnsApliwaPflegeF < GeoModel
    set_table_name 'aln_fns_apliwa_pflege_f'
    set_primary_key 'geodb_oid'
    set_default_rgeo_factory

    DATA_FIELDS=["geodb_oid", "liwaeingriff", "liwaeingriffsatus", "liwaeingriffjahr", "liwaauftrag"]
    scope :json_filter, select((DATA_FIELDS + ["geom"]).join(','))
    scope :csv_filter, select(DATA_FIELDS.join(','))

    def csv_header
      DATA_FIELDS
    end

    def csv_row
      [geodb_oid, liwaeingriff, liwaeingriffsatus, liwaeingriffjahr, liwaauftrag]
    end    
end

Controller

neuer Controller app/controllers/geo/aln_fns_apliwa_pflege_fs_controller.rb
Name des Controllers ist Modelname + "sController" (Plural beachten)

module Geo
  class AlnFnsApliwaPflegeFsController < GeoController

    def geo_model
      AlnFnsApliwaPflegeF
    end

  end
end

Config

neuer Eintrag im Geo-Namespace in config/routes.rb damit der Controller unter geo/aln_fns_apliwa_pflege_fs/ erreichbar ist

  namespace :geo do
    ...
    resources :aln_fns_apliwa_pflege_fs
  end

-->
