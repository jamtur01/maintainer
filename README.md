# Maintainer

Sends maintenance events to Riemann.

# Getting started

```bash
gem install maintainer
maintainer --help
```

# Usage

Generates a maintenance event that you can inject into the Riemann index
and use to indicate a host is in maintenance mode.

The state of the event defaults to `active` but you can specify an
alternate state with the `--event-state` flag.

The `:service` field defaults to `maintenance-mode` but can be
overridden with the `--event-service` flag. The host defaults to the
host the command is run on but can be overridden with the `--event-host`
flag.

### Starting maintenance

```
maintainer --host riemann.example.com
{:host server.example.com, :service maintenance-mode, :state
active, :description Maintenance is active, :type maintenance-mode, :metric nil,
:tags nil, :time 1457278453, :ttl Infinity}
```

You will see a new attribute called `:type` with a value of
`maintenance-mode` that you could also use for filtering.

### Ending maintenance

```
maintainer --host riemann.example.com --event-state inactive
{:host server.example.com, :service maintenance-mode, :state
inactive, :description Maintenance is inactive, :type maintenance-mode,
:metric nil, :tags nil, :time 1457278567, :ttl Infinity}
```

### Riemann configuration

We would define a function that checks for maintenance events.

```Clojure
(defn maintenance-mode?
  "Is it currently in maintenance mode?"
  [event]
  (->> '(and (= host (:host event))
             (= service "maintenance-mode"))
       (riemann.index/search (:index @core))
       first
       :state
       (= "active")))
```

And then use that function to control when notifications are generated:

```Clojure
(where (not (maintenance-mode? event))
  (pagerduty))
```

