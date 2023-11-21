---
theme: css/robot-lung.css
transition: none
width: "1920"
height: "1080"
maxScale: "4"
controls: true
progress: true
slideNumber: true
enableMenu: true
css:
  - css/extra.css
defaultTemplate: "[[tpl-ukaea-slide]]"
---
<!-- slide template="[[tpl-ukaea-title]]" -->

## IMAS UDA Workshop
# JSON Mapping Hands-On
#### _<u>Adam Parker</u>, Jonathan Hollocombe, Stephen Dixon_ <br> November 21, 2023 &#8212; ITER

---
## Overview

- This hands-on will cover mapping signals using JSON mapping files
- You will become comfortable with the types of mapping: 
	- VALUE
	- DIMENSION
	- PLUGIN
	- EXPR
- And how to call the relevant mappings directly and through IMAS
- If time permits additional topics such as debugging, mapping tests, exploring MAST-U mappings

---
## Contents

- Cloning the needed repositories
	- JSON_mapping_plugin
	- IMAS_workshop_mappings
	- UDA_workshop_plugin
- Install mappings to UDA server
- Map homogeneous_time directly and through IMAS
- Experiment with the functionality of the map types
- Break it! + How to fix
- Extra:
	- pytest and validation

---
## Workshop Format

- Each section will have some technical details
- Any hands-on exercises will be marked with __Task \#N__
- We will stop at each task to let people complete them before moving on
- Hands-on material may be surrounded by relevant slides + hints
- Please ask questions as we go

---
## Installing JSON Mapping Plugin

<div class="task">
Task #1: Clone the JSON mapping plugin repo
</div>

```bash
cd $HOME
git clone -b v0.1.2-alpha https://github.com/adam-parker1/JSON-mapping-plugin.git json-plugin
cd json-plugin
```

<div class="task">
Task #2: Install and activate
</div>

```bash
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/home/user/uda/install/lib/pkgconfig
cmake -B build -DCMAKE_INSTALL_PREFIX=$HOME/uda/install
cmake --build build/ -j 2
cmake --install build/
./build/scripts/activate-plugins.sh

uda_cli -h localhost -p 56565 "help::services()"
uda_cli -h localhost -p 56565 "imas_json_map::help()"
```

---
## Install Workshop IMAS Mappings

<div class="task">
Task #3: Clone the repo containing the dummy mapping files
</div>

```bash
cd $HOME/json-plugin
git clone https://github.com/adam-parker1/IMAS_workshop_mappings.git my_mappings
cd my_mappings
```

<div class="task">
Task #4: Explore the repository
</div>

- Take a look inside the directories, familiarise yourself
- E.g. mappings should look like this

```bash
╰─ ls mappings
magnetics        mock           pf_active
```

- There should be a `toplevel.schema.json` in `schemas/`<br>
  (We will touch on this later)
- Feel free to ask any questions
---
## Copy/Install Mappings

<div class="task">
Task #5: Install the mapping files to the server
</div>
 
```bash
cd /home/user/uda/install/etc
mkdir JSON_mappings
mkdir JSON_mappings/draft
```

- Copy the magnetics and pf_active from <br>`$HOME/json-plugin/my-mappings/mappings/*` into `JSON_mappings/draft`
- As well as the toplevel config `mappings.cfg.json`
- `JSON_mappings/draft` should then look as follows:

```bash
.
├── globals.json
├── magnetics
│   ├── globals.json
│   └── mappings.json
├── mappings.cfg.json
└── pf_active
    ├── globals.json
    └── mappings.json
```

---
## Installing pyuda client

<div class="task">
Extra Task if not completed yesterday
</div>

```sh
cd python3 -m venv venv 
source venv/bin/activate 
pip3 install --upgrade pip
pip3 install wheel cython numpy
pip3 install uda/install/python_installer
```


---

## Get homogeneous_time

<div class="task">
Task #6: Retrieve homogeneous_time for magnetics and pf_active
</div>

- Call the plugin directly through pyuda
- Set your host and port (hopefully you know this)

```python
import pyuda
pyuda.Client.server = '<host>'
pyuda.Client.port = <port>
client = pyuda.Client()
result = client.get("IMAS_JSON_MAP::get(path=magnetics/ids_properties/homogeneous_time, mapping=DRAFT)")
...
```
- Then print and inspect what is returned

<div class="task">
Task #7: Retrieve homogeneous_time for magnetics and pf_active **through IMAS**
</div>
<br>

- Now do the same using `partial_get` through the IMAS client
-  you may need to set `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/user/uda/install/lib` 

```python
import imas
entry = imas.DBEntry('imas://localhost:56565/uda?path=/home/user/data/105029/1&backend=hdf5&verbose=1', 'r')
entry.partial_get('magnetics' ,'flux_loop(3)')
```

---

## Break it! (very quick)

<div class="task">
Task #8: Modify your pf_active/mappings.json file so it is not valid
</div>
<br>
<br>

- Try call homogeneous_time again for pf_active
- Inspect the logs within your `install/etc`
	- Access.log
	- Debug.log
	- Error.log
	- JSON_plugin.log

- Hopefully you have something similar to  <br> `Could not load JSON mapping file`

---

## Fix and Validate

<div class="task">
Task #8: Install check-jsonschema
</div>

```bash
pip3 install check-jsonschema
## or use a virtual environment
cd
python3 -m venv venv
./venv/bin/activate
pip3 install check-jsonschema
```

<div class="task">
Task #9: Validate JSON and compare to toplevel schema
</div>

```bash
# cd to mapping directory
cd /home/user/json-plugin/my-mappings
check-jsonschema -v --schemafile schemas/toplevel.schema.json mappings/pf_active/mappings.json
```

Inspect the output and fix your problem (hopefully you know what it is... :)) <br> NOTE you made the changes to the copied server version, rather than the repo version!!

---

## Create Test Mapping Signal 

<div class="task">
Task #10: Add VALUE mapping entry to pf_active
</div>

```json
{
...
	"my/random/signal": {
		"MAP_TYPE": "VALUE",
		"VALUE": 4.6
	},
...
}
```

Call this mystery, random, test signal directly with the JSON plugin

```python
client.get("IMAS_JSON_MAP::get(path=pf_active/my/random/signal, mapping=DRAFT)")
```

Note, we still need to start with pf_active as this if the mapping file containing our mapping <br>(even if it is not an IDS path)

<br> 

**Extra points if you use templating**

---

## 80% of your problems from here on out will be invalid JSON

---

## Map, Map, Map

<div class="task">
Task #11: Quickfire add mapping entries 1
</div>
<br>

- Add VALUE map signal with a float value
- Add VALUE map signal with an array of floats
- **Extra:** Feel free to call both separately to check they work

<div class="task">
Task #12: Quickfire add mapping entries 2
</div> 
<br>

- Use these two signals in an EXPR map type

```json
{
...
	"test/expression_signal": {
		"MAP_TYPE": "EXPR",
		"PARAMETERS": {
			"X": "/test/signal1",
			"Y": "/test/signal2"
		},
		"EXPR": "X*Y + 3.4"
	},
...
}
```

- And call directly through `IMAS_JSON_MAP`

---

## Clone Data Access Template

<div class="task">
Task #13: Clone one more repo.. I promise
</div>
<br>

```bash
cd $HOME
git clone git@github.com:adam-parker1/UDA_workshop_plugin.git
cd UDA_workshop_plugin
```

Install and activate (the usual)

```bash
cmake -B build -DCMAKE_INSTALL_PREFIX=$HOME/uda/install
cmake --build build/ -j 2
cmake --install build/
./build/scripts/activate-plugins.sh

uda_cli -h localhost -p 56565 "help::services()"
uda_cli -h localhost -p 56565 "draft_json::help()"
uda_cli -h localhost -p 56565 "draft_json::hands_on_status()"
```

---

## Use the ::get_test_array() function

<div class="task">
Task #14: Add PLUGIN entry and the FUNCTION field to pf_active
</div>
<br>

```json
{
...
	"test/plugin_function": {
		"MAP_TYPE": "PLUGIN",
		"PLUGIN": "DRAFT_JSON",
		"ARGS": {},
		"FUNCTION": "get_test_array"
	},
...
}
```

Give it a go calling this `test/plugin_function` signal directly again

```bash
# should return...
[0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0]
```


---

## Map, Map, Map 2

<div class="task">
Task #15: Quickfire add mapping entries 1
</div>
<br>

- Attempt the DIMENSION map_type to get the size of `get_test_array()`

```json
{
...
	"test/plugin_function": {
		"MAP_TYPE": "DIMENSION",
		"DIM_PROBE": "test/plugin_function"
	},
...
}
```

<div class="task">
Task #16: Quickfire add mapping entries 2
</div>
<br>

- Add signal entry to get the SLICE of the array to return only one float value

<div class="task">
Task #17: Quickfire add mapping entries 3
</div> 
<br>

- Modify the SLICE entry signal to also OFFSET the data to **return 0**

---

## Final Mapping Test

<div class="task">
Task #18: Neutralise the get_test_array() data
</div>
<br>

- Add a final signal entry to scale get_test_array() by **-1**

```json
	"SCALE": -1.0
```

- Using both the original get_test_array() and the scaled version, <br> 


write an EXPR mapping type entry to `pf_active` such that the returned array is full of zeroes

```bash
# should return...
[0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
```

---

## Extra

- Using pytest to run schema mappings validation tests

<br> 

 **Add more if we get there**