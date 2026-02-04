# Vaadin 23+ UIDL analyzer

If your Vaadin communication grows significantly, this script will tell you
what is the main meat of that communication.

## UIDL anatomy

Run [vaadin-boot-example-gradle](https://github.com/mvysny/vaadin-boot-example-gradle) and open the "Network" tab in your browser dev tools.
Find requests named `/?v-r=uidl&v-uiId=*` and save those as
`dump0.uidl`. The JSON file will roughly look as follows:
```JSON
[
  {
    "clientId": 1,
    "LAZY": [...],
    "constants": {...},
    "changes": [
      {
        "node": 5,
        "type": "attach"
      },
      {
        "node": 5,
        "type": "put",
        "key": "tag",
        "feat": 0,
        "value": "vaadin-button"
      },
      {
        "node": 5,
        "type": "put",
        "key": "theme",
        "feat": 3,
        "value": "primary"
      }
    ],
    "execute": [
      [
        false,
        {
          "@v-node": 6
        },
        "return (async function() { this.invalid = $0}).apply($1)"
      ],
      [
        false,
        {
          "@v-node": 3
        },
        "return (async function() { this.serverConnected($0)}).apply($1)"
      ]
    ],
    "timings": [
      970,
      1
    ],
    "syncId": 1
  }
]
```
Most important parts are `execute` and `changes`:
* `changes` update component properties, add new components and remove existing ones
* `execute` lists all JavaScript snippets executed via `UI.getCurrent().getPage().executeJs()`

You can see that a component with node ID (internal ID used by UIDL for every Component) has been attached; it is a `<vaadin-button>` with the `primary` theme. Some nodes also executed certain async calls (v-node 6 set its `invalid` property to `false`; that is a TextField but I've ommitted it from the snippet above for brevity).

## Analyzing UIDL

Run the `./uidl-analyzer`. It is a simple Ruby script which reads the UIDL and dumps some statistics:
```
* Parsing UIDL
JSON sizes in bytes: 4k; changes 2k, execute 0k
* New Components: 3
{"vaadin-button" => 1, "vaadin-text-field" => 1, "vaadin-vertical-layout" => 1}
* Most common component
5<vaadin-button>
* Executions: 3
* Component with most executions
3<>,executions=1
```
You start seeing network/CPU issues if:
* The number of new components is too high: 500+ . This usually happens in large Grids when overusing ComponentRenderers
* The number of executions is too high

