---
layout: architect
title: Publishing WFS services
---

Publishing WFS services
=======================

UMN Mapserver
-------------

-   Add WFS support in your mapfile according to [Mapserver documentation](http://mapserver.org/ogc/wfs_server.html)
-   Test WFS locally using CGI with parameter 'map' pointing to your mapfile
-   Set permissions for resources `WFS` and `Topic`
-   URL after deployment: `http://maps.example.com/wfs/MyWFS?SERVICE=WFS&VERSION=1.0.0&REQUEST=GetCapabilities`

Tables without geometries
-------------------------

WFS expects a geometry column in a layer. If you want to publish a WFS without geometry, you can add an artificial geometry on-the fly:

    LAYER
        NAME "mylayer" 
        STATUS on
        METADATA
           "wfs_enable_request"              "*" 
           "wfs_title"                       "My Layer" 
           "wfs_srs"                         "EPSG:21781 EPSG:21780 EPSG:4326 EPSG:31297 EPSG:4258" 
           "gml_include_items"               "all"   
           "gml_geometries"                  "none"  
           "wfs_featureid"                   "geodb_oid"    
        END
        INCLUDE 'connection.inc'
        DATA "wfsgeom FROM (SELECT *,ST_GeomFromText('POINT(680000 260000)', 21781) AS wfsgeom FROM mytable) AS subquery USING UNIQUE geodb_oid USING srid=21781" 
        TYPE POINT
    END
