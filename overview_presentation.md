---
theme: css/robot-lung.css
transition: convex
width: "1024"
height: "800"
maxScale: "4"
controls: true
progress: true
slideNumber: true
enableMenu: true
css:
  - css/extra.css
defaultTemplate: "[[tpl-ukaea-slide]]"
---

# Overview Presentation 

- UDA
	- Design
	- Architecture
	- Request processing
	- Batch requests
	- Plugins
- IMAS UDA
	- Low-level overview
	- UDA backend
	- XML processing
	- Data pre-fetching
	- Data loading
- UDA plugin
	- Function overview
	- Remote data access
	- Mapped data access
- JSON mapping plugin
	- Function overview
	- JSON schema
	- Map types

---

<!-- slide template="[[tpl-ukaea-title]]" -->

# UDA Overview Presentation
## IMAS UDA workshop, ITER
### Jonathan Hollocombe, November 2023

---
# UDA

- History & Philosophy
- Design
- Architecture
- Request Processing
- Batch Requests
- Plugins 

---
## UDA - History & Philosophy

- Created for MAST to abstract away change from IDA3 (custom binary format) to NetCDF4
- Also to allow interaction between 32bit and 64bit machines
- Client-Server architecture to separate users from location and format of data files
- Developed a plugin architecture on the server to allow more formats to be added without having to update the server
- Coupling of network data transport with extensible server plugins makes it useful for interfacing with IMAS for remote data access

---
## UDA - Design

![[images/UDA.svg|1000]]

---



## UDA - Design (Server)

![[UDA Server.svg|700]]

---



## UDA - Design (Client)

![[UDA Client.svg|400]]

---



## UDA - Server Architecture

- Plugins loaded based on Plugins.conf
- Requests are received as Signal & Source
- Multiple requests can be received in one batch
- Requests are handled by calling one or more plugins
- Data is serialised and returned to client

---



## UDA - udaPlugins.conf

`NAME, TYPE, FUNC, LIB_NAME, CONFIG, DESC, EXAMPLE`

- `NAME` - name plugin is known to UDA client
- `TYPE` - type of plugin: **`function`**, `file`, `server`
- `FUNC` - the entry function that will be called by the UDA server
- `LIB_NAME` - the name of the shared library to load
- `CONFIG` - nearly always "`*, 1, 1, 1`"
	- `EXTENSION` - only used by `file` plugins
	- `INTERFACE`  - plugin interface supported, currently must be `1`
	- `CACHE` - whether the returned data can be cached in the server
	- `PUBLIC` - whether the plugin is public, i.e. can be called by clients
- `DESC` - a description of the plugin
- `EXAMPLE` - an example of calling the plugin

---



## UDA - udaPlugins.conf (cont.)

```text
BYTES, function, bytesPlugin, libbytes_plugin.dylib, *, 1, 1, 1, data reader to access files as a block of bytes without interpretation, BYTES::read()  
HELP, function, helpPlugin, libhelp_plugin.dylib, *, 1, 1, 1, Service Discovery: list the details on all registered services, HELP::services()  
TEMPLATEPLUGIN, function, templatePlugin, libtemplate_plugin.dylib, *, 1, 1, 1, Standardised Plugin Template, TEMPLATEPLUGIN::functionName()  
TESTPLUGIN, function, testplugin, libtestplugin_plugin.dylib, *, 1, 1, 1, Generate Test Data, TESTPLUGIN::test1()  
UDA, function, UDAPlugin, libuda_plugin.dylib, *, 1, 1, 1, forward on requests to the UDA client, UDA::get(host=idam0, port=56565, signal=ip, source=12345)  
VIEWPORT, function, viewport, libviewport_plugin.dylib, *, 1, 1, 1, Reduce data to viewport pixel size, viewport::get()
```

 > **Note\:** udaPlugins.conf may be replaced by auto-plugin discovery in a future UDA version


---



## UDA - Request Processing

- Client sends request to server:
	- `signal` - what to read
	- `source` - where to read it from
- A metadata database can be used to perform name aliasing
	- e.g. for MAST-U we map
		- signal name `ip` -> `AMC_PLASMA_CURRENT`
		- source `30420` -> `$MAST_DATA/30420/LATEST/amc_30420.nc`
	- Metadata lookup is not covered in this workshop but more details can be provided on request
- For direct plugin requests only the signal argument is needed
	- Example:
		- signal = "HELP::help()"
		- source = ""

---


## UDA - Request Processing (cont.)

- A plugin request looks like:
	- `PLUGIN::FUNCTION(ARGS)`
	- `PLUGIN` - the name of the plugin as defined in the `udaPlugins.conf`
	- `FUNCTION`
	- `ARGS` - 0 or more comma separated arguments to be passed to the function, e.g.
		- `PLUGIN::FUNC(foo=1)` - passing integer arg
		- `PLUGIN::FUNC(foo=1.23)` - parsing float arg
		- `PLUGIN::FUNC(foo='abc')` - parsing string arg (quotes are optional)
		- `PLUGIN::FUNC(foo=1;2;3)` - parsing list arg

---



## UDA - Batch Requests

- Multiple requests can be sent to the UDA server at

---



## UDA - Plugins

- What is a UDA plugin
- How is called by the server
- What it is expected to do
- Examples

---

## UDA - Serialisation

---

# IMAS UDA

- Section details how UDA is used in IMAS
---

## Low-level overview

- Covers the how the low-level works
---

## UDA backend

- How the UDA backend is called
---

## XML processing

- How the data dictionary is used
---

## Data pre-fetching

- How we pre-fetch the data
---

## Data loading

- How the data is passed back to IMAS
---

# UDA IMAS plugin

- This section covers the functionality of the UDA IMAS plugin
---

## Function overview

- Covering the plugin functions
---

## Remote data access

- Getting data from IMAS files
---

## Mapped data access

- Forwarding requests to mapping plugin
---

# JSON mapping plugin

- Briefly mention JSON plugin - leading on to later presentation