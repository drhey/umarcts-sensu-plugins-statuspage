## Sensu-Plugins-statuspage

[![Build Status](https://travis-ci.org/sensu-plugins/sensu-plugins-statuspage.svg?branch=master)](https://travis-ci.org/sensu-plugins/sensu-plugins-statuspage)
[![Gem Version](https://badge.fury.io/rb/sensu-plugins-statuspage.svg)](http://badge.fury.io/rb/sensu-plugins-statuspage)
[![Code Climate](https://codeclimate.com/github/sensu-plugins/sensu-plugins-statuspage/badges/gpa.svg)](https://codeclimate.com/github/sensu-plugins/sensu-plugins-statuspage)
[![Test Coverage](https://codeclimate.com/github/sensu-plugins/sensu-plugins-statuspage/badges/coverage.svg)](https://codeclimate.com/github/sensu-plugins/sensu-plugins-statuspage)
[![Dependency Status](https://gemnasium.com/sensu-plugins/sensu-plugins-statuspage.svg)](https://gemnasium.com/sensu-plugins/sensu-plugins-statuspage)

## Functionality

This is a fork of the original plugin. Rather than having individual machines report into StatusPage, we wanted Sensu check aggregates to be the main generator of information into StatusPage. This took only a small reworking of the original code. Instead of creating StatusPage incidents for client name and check name, the handler will now look for a custom check attribute in the event data and title the StatusPage incident name after that (for ease of understanding of those looking at the StatusPage interface).

**handler-statuspage**

Creates an issue on StatusPage.io and (optionally) updates a component status.

**metrics-statuspageio**

Sends graphite-style metrics to statuspage.io, for displaying public metrics.  Note, this forks and is not meant for high-throughput.  Rather, it is meant for high-value, low-throughput metrics for display on status page.

## Files
 * bin/handler-statuspage
 * bin/metrics-statuspageio

## Usage

**handler-statuspage**
```
{
  "statuspage": {
    "api_key": "YOURAPIKEY",
    "page_id": "YOURPAGEID"
  }
}
```

For use of a basic proxy, use "proxy_address" and "proxy_port":
```
{
  "statuspage": {
    "api_key": "YOURAPIKEY",
    "page_id": "YOURPAGEID",
    "proxy_port": "YOURPROXY",
    "proxy_address": "YOURPROXYADDRESS"
  }
}
```

**metrics-statuspageio**
```
{
    "metrics-statuspageio": {
        "api_key": "my_api_key",
        "page_id": "my_page_id",
        "metrics": {
            "some.metric.identifier": "my_metric_id",
            "another.metric.identifier": "another_metric_id"
        }
    }
}
```
## Installation

[Installation and Setup](http://sensu-plugins.io/docs/installation_instructions.html)

## Notes

The Redphone functionality offered by statuspage.rb and helpers.rb have now been
incorporated into this gem. So, you no longer have to clone the Redphone repo,
build, and install from scratch.

To update a component add a `"component_id": "IDHERE"` attribute to the corresponding check definition

Example:
```
{
  "checks": {
    "check_sshd": {
      "handlers": ["statuspage"],
      "component_id": "IDHERE",
      "command": "/etc/sensu/plugins/check-procs.rb -p sshd -C 1 ",
      "interval": 60,
      "subscribers": [ "default" ]
    }
  }
}
```

To choose your own component or incident statuses instead of the defaults add the `statuspage_<component/incident>_<status>` in the check definition.

Example:
```
{
  "checks": {
    "check_sshd": {
      "handlers": ["statuspage"],
      "component_id": "IDHERE",
      "statuspage_component_warning": "degraded_performance",
      "statuspage_component_critical": "partial_outage",
      "statuspage_incident_warning": "ignore",
      "statuspage_incident_critical": "identified",
      "command": "/etc/sensu/plugins/check-procs.rb -p sshd -C 1 ",
      "interval": 60,
      "subscribers": [ "default" ]
    }
  }
}
```

To choose the name of your incident that will be reported to StatusPage, simply add a custom attribute like this in your check definition:
```
"custom": {
  "incident_name": "Compute Nodes"
}      
```

The handler is specifically looking for the above to be defined. If it's not, then incident_key won't be defined in the handler and this will cause errors and improper functionality
