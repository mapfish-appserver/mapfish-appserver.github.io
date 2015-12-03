---
layout: architect
title: Printing
---

Printing
========

Mapfish Appserver v1.1.0 or later uses Mapfish Print v3. Previous versions were using Mapfish Print v2.
Print requests to the Mapfish Appserver are still using the v2 API.

See also the [Mapfish Print v3 documentation](http://mapfish.github.io/mapfish-print-doc)


Mapfish Print v3 installation
-----------------------------

### Web application

Mapfish Print can be run as a web application e.g. on Tomcat.

Copy the Mapfish Print WAR to the webapps directory

    cp print-servlet-3.2.0.war /usr/local/tomcat/webapps/mapfish_print.war

Add a symlink from your Mapfish Appserver application's print configuration directory to the Mapfish Print print apps directory

    ln -s $MAPFISH_APP_DIR/print/ /usr/local/tomcat/webapps/mapfish_print/print-apps/myapp

The Mapfish Print application will then run on [http://localhost:8080/mapfish_print/index.html](http://localhost:8080/mapfish_print/index.html)

Add the print URL to your Mapfish Appserver configuration (e.g. in `config/environments/development.rb`)

    PRINT_URL = 'http://localhost:8080/mapfish_print/print/myapp'

### Standalone

For development, Mapfish Print can also be run standalone.

Add the popen4 gem to your Mapfish Appserver `Gemfile`

    gem 'popen4'

Activate standalone printing and add the path to the Mapfish Print JARs to your Mapfish Appserver configuration

    PRINT_URL = nil
    PRINT_STANDALONE_JARS = '$PATH_TO/mapfish-print/core/lib/*'


Configuration
-------------

The print configuration, JasperReports templates and any images are located in the `print/` directory.

### Workflow

- create new JasperReports template and place the `*.jrxml` in `print/`
- add new template to `print/config.yaml`
  - set `reportTemplate` to the new `*.jrxml`
  - add any external report parameters to the template attributes
  - add any Mapfish Print elements to the template attributes and processors

### Custom reports

By default, all templates in `print/config.yaml` will get listed in the print capabilities.
To remove a specific template from the default print layouts, the custom attribute `gb_custom_template` can be added, e.g.

    templates:
      #============================================================================
      Custom report: !template
      #============================================================================
        reportTemplate: custom_report.jrxml
        attributes:
          # mark as custom template (hide in print/info.json)
          gb_custom_template: !boolean {default: true}
        processors:
        - !reportBuilder
            directory: "."

Print requests for the custom report will then still be accepted but the report will no longer show up in the print capabilities.


Customization
-------------

The `PrintController` can be subclassed to add report specific permission checks, set print parameters or customize the returned filename

```ruby
class MyAppPrintController < PrintController

  # check permission for printing, e.g. check params to limit reports to user roles
  def check_permissions
    true
  end

  # add custom print params according to report type, e.g. add map and zoom to feature in custom reports
  def set_custom_print_params(report, print_params)
    case report
    when "TopicName"
      # add map and layer
      layer_url = rewrite_wms_uri("#{wms_host}/TopicName", false)
      print_params['attributes']['map'] = {
        :dpi => print_params[:dpi],
        :bbox => [669242, 223923, 716907, 283315],
        :projection => 'EPSG:21781',
        :layers => [
          {
            :type => 'WMS',
            :baseURL => layer_url,
            :layers => ['Layer1', 'Layer2'],
            :imageFormat => 'image/png; mode=8bit',
            :styles => [''],
            :customParams => {
              :TRANSPARENT => true,
              :map_resolution => print_params['dpi']
            }
          }
        ]
      }
    end
  end

  # handler for sending custom report file, e.g. to customize filename depending on params
  def send_custom_report(report, print_params, path, type, filename)
    send_file path, :type => type, :disposition => 'attachment', :filename => filename
  end

end
```

Add routes to custom print controller to `config/routes.rb`

```ruby
  match 'print/info.:format' => "my_app_print#info"
  match 'print/create' => "my_app_print#create", :via => :post
  match 'print/:id' => "my_app_print#show"
```
