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
# UDA Overview Presentation
#### _<u>Jonathan Hollocombe</u>, Adam Parker, Stephen Dixon_ <br> November 20, 2023 &#8212; ITER

---
## Contents

- History & Philosophy
- Design
- Architecture
- Request Processing
- Batch Requests
- Plugins 

---
## UDA — History & Philosophy

- Created for MAST to abstract away change from IDA3 (custom binary format) to NetCDF4
- Also to allow interaction between 32bit and 64bit machines
- Client-Server architecture to separate users from location and format of data files
- Developed a plugin architecture on the server to allow more formats to be added without having to update the server
- Coupling of network data transport with extensible server plugins makes it useful for interfacing with IMAS for remote data access

---
## UDA — Design

![[images/UDA.svg|1000]]

---
## UDA — Design (Server)

![[UDA Server.svg|700]]

---
## UDA — Design (Client)

![[UDA Client.svg|400]]

---
## UDA — Server Architecture

- Plugins loaded based on Plugins.conf
- Requests are received as Signal & Source
- Multiple requests can be received in one batch
- Requests are handled by calling one or more plugins
- Data is serialised and returned to client

---
## UDA — udaPlugins.conf

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
## UDA — udaPlugins.conf Example

```text
BYTES, function, bytesPlugin, libbytes_plugin.dylib, *, 1, 1, 1, data reader to access files as a block of bytes without interpretation, BYTES::read()  
HELP, function, helpPlugin, libhelp_plugin.dylib, *, 1, 1, 1, Service Discovery: list the details on all registered services, HELP::services()  
TEMPLATEPLUGIN, function, templatePlugin, libtemplate_plugin.dylib, *, 1, 1, 1, Standardised Plugin Template, TEMPLATEPLUGIN::functionName()  
TESTPLUGIN, function, testplugin, libtestplugin_plugin.dylib, *, 1, 1, 1, Generate Test Data, TESTPLUGIN::test1()  
UDA, function, UDAPlugin, libuda_plugin.dylib, *, 1, 1, 1, forward on requests to the UDA client, UDA::get(host=idam0, port=56565, signal=ip, source=12345)  
VIEWPORT, function, viewport, libviewport_plugin.dylib, *, 1, 1, 1, Reduce data to viewport pixel size, viewport::get()
```

<grid drag="80 20" drop="10 70" align="left" bg="orange" pad="0 20px 0 20px">
**Note\:** udaPlugins.conf may be replaced by auto-plugin discovery in a future UDA version
</grid>

---
## UDA — Request Processing

- Client sends request to server:
	- `signal` - what to read
	- `source` - where to read it from
- A metadata database can be used to perform name aliasing
	- e.g. for MAST-U we map
		- signal name `ip` ⇒ `AMC_PLASMA_CURRENT`
		- source `30420` ⇒ `$MAST_DATA/30420/LATEST/amc_30420.nc`
	- Metadata lookup is not covered in this workshop but more details can be provided on request
- For direct plugin requests only the signal argument is needed
	- Example:
		- signal = "HELP::help()"
		- source = ""

---
## UDA — Request Processing (cont.)

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
## UDA — Batch Requests

- Multiple requests can be sent to the UDA server at one time
- Each request is processed by the server and passed to a plugin
- The resultant data blocks are concatenated and returned to the client in one go
- Batch requests reduce TCP round-trip overhead and can help with plugin call times

---
## UDA — Plugins

- The UDA plugins is where most of the work is done in the server
- The plugins responsibility is to handle the request parsed from the server and return data back
- Plugins can be written in any language but must be compiled into a shared library and present a C style entry function
	- This entry function must match the name provided in the udaPlugins.conf
- More details will be provided in the plugin workshop

---
### UDA — Plugins - Entry Function

- The UDA plugin entry function has the following form:

```C
int entryFunction(IDAM_PLUGIN_INTERFACE* plugin_interface)
```

where:

- `IDAM_PLUGIN_INTERFACE` is the structure passed in from the server which is used to access the request and return the data
- The return value should be `0` for success or non-`0` for an error state

---
### UDA — Plugins — Basic Structure

```C
int entryFunction(IDAM_PLUGIN_INTERFACE* plugin_interface)
{
	static bool init = false;

	if (udaFunctionCalled(plugin_interface, "reset")) {  
	    if (!init) { return 0; }
	    // reset plugin to initial state
	    init = false;
	    return 0;
	}

...
```
- Function name passed to `udaFunctionCalled` is case-insensitive

---
### UDA — Plugins — Basic Structure (cont.)

```C
...

if (!init 
	|| udaFunctionCalled(plugin_interface, "init|initialise")) {
	// perform any plugin initialistion
    init = true;
    if (udaFunctionCalled(plugin_interface, "init|initialise")) {  
        return 0;  
    }  
}

```
- Multiple names can be checked if passed to `udaFunctionCalled` separated by `|` character

---
### UDA — Plugins — Basic Structure (cont.)

```C
...

if (udaFunctionCalled(plugin_interface, "help")
	|| udaDefaultFunctionCalled(plugin_interface)) {  
	...
} else if (udaFunctionCalled(plugin_interface, "version")) {  
	...
} else if ...

} else if (udaFunctionCalled(plugin_interface, "something")) {  
    return do_something(plugin_interface);  
}
```

---
### UDA — Plugins — Default Plugins

- `bytes` - a plugin for returning files back as raw bytes, useful if no other access plugins are available
- `hdf5` - a plugin for reading HDF5 files
- `help` - a plugin for return help information about UDA and available services
- `keyvalue` - a plugin for interacting with LevelDB key-value store
- `template` - an empty plugin which can be copied to create new plugins
- `uda` - a plugin used to call other UDA servers
- `viewport` - a plugin to reshape data to fit in a fixed viewport

---
### UDA — Plugins — Examples

__HELP:__
```text
help::ping() - check server is responding
help::help() - get basic UDA usage
help::services() - list the plugins available on the server
```

__UDA__:
```text
UDA::get(host=..., port=..., signal=..., source=...)
```

__BYTES__:
```text
BYTES::read(path=...)
```
---
## UDA — Serialisation

- Data is returned from the server to the client via the `DATA_BLOCK` structure
	- Available on the plugin interface as<br>
	  `interface->data_block`
- This data block contains space for the data, metadata & dimensions

---
### UDA — Serialisation — Data Block

```C
typedef struct DataBlock {  
    unsigned int rank;  
    int order;  
    int data_type;
  
    int data_n;
    char* data;
  
    char data_units[STRING_LENGTH];  
    char data_label[STRING_LENGTH];  
    char data_desc[STRING_LENGTH];  
  
    DIMS* dims;
} DATA_BLOCK;
```

Note\: Not the full DataBlock, only showing the relevant fields for plugin development.

---
### UDA — Serialisation — Dims Block

```C
typedef struct Dims {  
    int data_type; // Type of data  
    int dim_n; // Array lengths  
  
    char* dim; // Dimension Data Array  
  
    char dim_units[STRING_LENGTH];  
    char dim_label[STRING_LENGTH];  
} DIMS;
```

---
### UDA — Serialisation — Data Types

```C
typedef enum UdaType {  
    UDA_TYPE_UNKNOWN = 0,  
    UDA_TYPE_CHAR = 1,  
    UDA_TYPE_SHORT = 2,  
    UDA_TYPE_INT = 3,  
    UDA_TYPE_UNSIGNED_INT = 4,  
    UDA_TYPE_LONG = 5,  
    UDA_TYPE_FLOAT = 6,  
    UDA_TYPE_DOUBLE = 7,  
    UDA_TYPE_UNSIGNED_CHAR = 8,  
    UDA_TYPE_UNSIGNED_SHORT = 9,  
    UDA_TYPE_UNSIGNED_LONG = 10,  
    UDA_TYPE_LONG64 = 11,  
    UDA_TYPE_UNSIGNED_LONG64 = 12,  
    UDA_TYPE_COMPLEX = 13,  
    UDA_TYPE_DCOMPLEX = 14,  
    UDA_TYPE_UNDEFINED = 15,  
    UDA_TYPE_VLEN = 16,  
    UDA_TYPE_STRING = 17,  
    UDA_TYPE_COMPOUND = 18,  
    UDA_TYPE_OPAQUE = 19,  
    UDA_TYPE_ENUM = 20,  
    UDA_TYPE_VOID  = 21,  
    UDA_TYPE_CAPNP = 22,  
    UDA_TYPE_STRING2 = 99  
} UDA_TYPE;
```

---
### UDA — Serialisation — Cap'n'Proto Serialisation

![[Capnp Tree.svg|800]]

- As an alternative of writing array data to the DataBlock you can instead write the data in a structure tree and then serialise that tree using Cap'n'Proto, returning the serialised buffer

---
### UDA — Serialisation — Cap'n'Proto Example

```C++
auto tree = uda_capnp_new_tree();  
auto root = uda_capnp_get_root(tree);  
uda_capnp_set_node_name(root, "root");  
uda_capnp_add_children(root, 3);  
  
auto child = uda_capnp_get_child(tree, root, 0);  
uda_capnp_set_node_name(child, "double_array");  
  
std::vector<double> vec(30);  
for (int i = 0; i < 30; ++i) {  
    vec[i] = i / 10.0;  
}  
uda_capnp_add_array_f64(child, vec.data(), vec.size());  
  
child = uda_capnp_get_child(tree, root, 1);  
uda_capnp_set_node_name(child, "i32_array");  
  
std::vector<int32_t> ivec(100);  
for (int i = 0; i < 100; ++i) {  
    ivec[i] = i;  
}  
uda_capnp_add_array_i32(child, ivec.data(), ivec.size());  
  
child = uda_capnp_get_child(tree, root, 2);  
uda_capnp_set_node_name(child, "i64_scalar");  
  
uda_capnp_add_i64(child, 999);  
  
auto buffer = uda_capnp_serialise(tree);  
  
DATA_BLOCK* data_block = plugin_interface->data_block;  
initDataBlock(data_block);  
  
data_block->data_n = static_cast<int>(buffer.size);  
data_block->data = buffer.data;  
data_block->dims = nullptr;  
data_block->data_type = UDA_TYPE_CAPNP;
```