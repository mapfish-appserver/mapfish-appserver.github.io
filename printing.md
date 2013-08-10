---
layout: architect
title: Printing
---

Printing
========

<!--
Mapfish Print
-------------

config/print.yml

Bezeichnung KARTENNAME: Topic-Name

Disclaimer für Footer

Der Standardtext für den Disclaimer befindet sich in der Datei app/views/topics/_print_disclaimer.txt

Soll ein anderer Text für ein Topic gesetzt werden, so kann eine Textdatei app/views/topics/custom/_KARTENNAME_print_disclaimer.txt hinzugefügt werden.

Wenn eine spezifische Textdatei für ein Topic existiert, wird deren Inhalt für den Disclaimer verwendet. Andernfalls wird der Standardtext angezeigt.


JasperReports
-------------

Mit dem printreportsadd Event können Reports zum PrintPanel hinzugefügt werden.

Konfiguration:

    {
      name: "<Reportname für Layout>",
      id: "<ReportId>",
      map: {
        width: <Kartenbreite in [mm]>,
        height: <Kartenhöhe in [mm]>
      }
    }

ReportID entspricht dem Pfad im JasperReports: http://testmaps.kt.ktzh.ch/jasperserver/rest_v2/reports/reports/gb/{ReportID}.pdf

Umrechnung von JasperReports Einheiten in mm:
    A4: 210mm x 297mm = 595 x 842 -> {JasperReports Units} * 210/595 = {mm}

Reports für bestimmte Topics/Layers können in der Legende eingebunden werden
z.B. in app/views/topics/custom/_basiskartezh_legend.html.erb:

    <div onClick='
      GbZh.base.ViewerState.fireEvent("printreportsadd",
      [
        {
            name: "JasperReport Demo",
            id: "demo",
            map: {
              width: 150,
              height: 100
            }
        },
        {
            name: "JasperReport Gemeinden",
            id: "gemeinden",
            map: {
              width: 200,
              height: 280
            }
        }
      ]);
    '>REPORTS</div>

-->