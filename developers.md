---
layout: architect
title: Developers
---

Hacking Mapfish Appserver
-------------------------

gb_mapfish_appserver includes a viewer for developing and testing Mapfish Appserver.

    cd test/dummy

To setup a development environment, copy and adapt the following database configuration files:

    cp config/database.yml.example config/database.yml
    cp config/geodatabase.yml.example config/geodatabase.yml
    cp mapconfig/dapgis.sourcepole.com/connection.inc.example mapconfig/dapgis.sourcepole.com/connection.inc

Install all needed gems:

    bundle install

Setup the application database:

    bundle exec rake db:create db:migrate
    rake mapfish:seed_database SITE=maps.example.com
    rake mapfish:viewer:register name=myviewer

Create a geodatabase following the Quick-start guide.

Import the mapfiles into the application database:

    rake mapfile:import_topic MAPFILE=mapconfig/maps.example.com/naturalearth.map SITE=maps.example.com
    rake mapfile:import_topic MAPFILE=mapconfig/maps.example.com/cascadingwms.map SITE=maps.example.com

Ready to start the server and open your browser:

    rails server
    x-www-browser localhost:3000/
