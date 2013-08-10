---
layout: architect
title: Custom searches
---

Custom searches
===============

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
