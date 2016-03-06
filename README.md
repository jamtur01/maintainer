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

### Starting maintenance

```json
maintainer --host riemann.example.com
{:host server.example.com, :service maintenance-mode, :state
active, :description Maintenance is active, :metric nil, :tags nil,
:time 1457278453, :ttl Infinity}
```

### Ending maintenance

```json
maintainer --host riemann.example.com --event-state nil
{:host server.example.com, :service maintenance-mode, :state
nil, :description Maintenance is active, :metric nil, :tags nil,
:time 1457278453, :ttl Infinity}
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

