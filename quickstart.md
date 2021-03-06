---
layout: architect
title: Quick-start guide
---

Quick-start guide
=================

To build a new Mapfish Appserver application, create a Rails application
and include mapfish\_appserver in your Gemfile:

    rails new mfdemo --database=postgresql --skip-bundle --skip-javascript
    cd mfdemo
    echo "gem 'therubyracer', :platforms => :ruby" >>Gemfile
    echo "gem 'gb_mapfish_appserver'" >>Gemfile
    echo "gem 'gb_mapfish_print'" >>Gemfile
    echo "gem 'ruby_mapscript', :platforms => :ruby" >>Gemfile
    bundle install

Also make sure, you have MapScript installed:

    sudo apt-get install libmapscript-ruby1.9.1

Setup the Mapfish project, initialize the application database:

    rake db:create
    rails generate mapfish:install --default-site-name=maps.example.com

Keep the admin password for later.

Generate a basic viewer:

    rake mapfish:viewer:create name=myviewer repo=https://github.com/mapfish-appserver/gb_mapfish_viewer_gis_browser.git
    rm public/index.html

Add a default route to your viewer in config/routes.rb:

    root :to => "apps#show", :app => "myviewer"

Setup a PostGIS database and load some data:

    export PGPORT=5432
    createdb geodb
    psql -d geodb -c "CREATE EXTENSION postgis"
    psql -d geodb -c "GRANT ALL ON geometry_columns TO PUBLIC; GRANT ALL ON geography_columns TO PUBLIC; GRANT ALL ON spatial_ref_sys TO PUBLIC"
    wget http://www.naturalearthdata.com/http//www.naturalearthdata.com/download/10m/cultural/ne_10m_admin_0_countries_lakes.zip
    unzip ne_10m_admin_0_countries_lakes.zip
    ogr2ogr -f PostgreSQL PG:"dbname=geodb" -nln countries -nlt MULTIPOLYGON ne_10m_admin_0_countries_lakes.shp
    psql -d geodb -c "GRANT ALL ON countries TO PUBLIC"

Create a mapfile naturalearth.map in the directory
mapconfig/maps.example.com:

    MAP
      NAME naturalearth

      WEB
        METADATA
          "wms_title"                  "Natural Earth"
          "wms_onlineresource"         "http://wms.example.com/naturalearth"
          "wms_srs"                    "EPSG:4326 EPSG:3857 EPSG:21781"
          "wms_extent"                 "-180 -90 180 90"
          "wms_feature_info_mime_type" "application/vnd.ogc.gml"
          'ows_enable_request'    '*'
          "wms_encoding"               "UTF-8"
        END
        IMAGEPATH '/tmp/'
        IMAGEURL '/tmp/'
      END

      SIZE 512 512
      MAXSIZE 8192
      RESOLUTION 96
      DEFRESOLUTION 96

      UNITS DD
      PROJECTION
        "init=epsg:4326"
      END

      EXTENT -180 -90 180 90

      IMAGECOLOR 255 255 255

      #SYMBOLSET "../symbols/base.sym"
      #FONTSET   "../fonts/fonts.txt"

      IMAGETYPE png

      OUTPUTFORMAT
        NAME png
        DRIVER "AGG/PNG"
        IMAGEMODE rgb
        FORMATOPTION "INTERLACE=OFF"
      END

      LAYER
        NAME 'countries'
        METADATA
          "wms_title"                       "Countries"
          "wms_srs"                         "EPSG:4326"
          "wms_extent"                      "-180 -90 180 90"
          "wms_include_items"               "name,pop_est"
          "gml_include_items"               "name,pop_est"
          "gml_name_alias"                  "Name"
          "gml_pop_est_alias"               "Population"
        END
        TEMPLATE "blank.html"

        EXTENT -180 -90 180 90
        MINSCALEDENOM 1
        MAXSCALEDENOM 500000000.5

        STATUS ON
        TYPE POLYGON
        CONNECTIONTYPE postgis
        CONNECTION "dbname=geodb port=5432"
        DATA "wkb_geometry FROM countries USING UNIQUE ogc_fid"

        CLASS
          NAME 'All countries'
          STYLE
            WIDTH 0.91 
            OUTLINECOLOR 0 0 0
            COLOR 0 255 0
          END
        END
      END

    END

Check your WMS setup:

    #sudo apt-get install cgi-mapserver
    wget -O map.png "http://localhost/cgi-bin/mapserv?map=$(pwd)/mapconfig/maps.example.com/naturalearth.map&SERVICE=WMS&VERSION=1.3.0&REQUEST=GetMap&BBOX=-90,-180,90,180&CRS=EPSG:4326&WIDTH=706&HEIGHT=354&LAYERS=countries&STYLES=&FORMAT=image/png"

Open `config/initializers/mapfish.rb` to configure your application.

Import the mapfile into a new topic:

    rake mapfile:import_topic MAPFILE=mapconfig/maps.example.com/naturalearth.map SITE=maps.example.com

Start the application server and open your first viewer application in
your web browser:

    rails server
    x-www-browser localhost:3000/

To access the backend:

    x-www-browser localhost:3000/gbadmin

Login as 'admin' with with the generated password. After your first
login you will be redirected back to the root path. Go to
[http://localhost:3000/gbadmin/user/1/edit](http://localhost:3000/gbadmin/user/1/edit)
to change the password.
