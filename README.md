## umarcts-sensu-plugins-statuspage

## Functionality

This is a fork of the original plugin from https://github.com/sensu-plugins/sensu-plugins-statuspage. Rather than having individual machines report into StatusPage, we wanted Sensu check aggregates to be the main generator of information into StatusPage. This took only a small reworking of the original code. 

Instead of creating StatusPage incidents from information like `@event['client']['name'] + '/' + @event['check']['name']`, the handler will now look for a custom check attribute (`@event['check']['custom']['incident_name']`) in the event data and title the StatusPage incident name after that (for ease of understanding of those looking at the StatusPage interface, i.e. end-users).

Also The Redphone functionality required by this plugin has now been baked in. One no longer has to clone the Redphone repo, build, and install from scratch. Also, proxy address and port have been added to this handler as well as the dependent Redphone bits.

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

To choose the name of your incident that will be reported to StatusPage, simply add a custom attribute (e.g. `"custom": { "incident_name": "What you want the incident to be called" },`) like this in your check definition:

Example:
```
{
  "checks": {
    "check_keepalive_aggregate": {
      "command": "check-aggregate.rb -c AGGREGATENAME -C 66 -W 33 -a http://127.0.0.1:4567 -u USER -p PASSWORD",
      "handle": true,
      "interval": 30,
      "occurrences": 5,
      "handlers": [ "statuspage" ],
      "component_id": "IDHERE",
      "custom": {
        "incident_name": "Compute Nodes"
      },
      "refresh": 3600,
      "standalone": true
    }
  }
}   
```

The handler is specifically looking for the above to be defined. If it's not, then incident_key won't be defined in the handler and this will cause errors and improper functionality
