---
theme: css/robot-lung.css
transition: none
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
<!-- slide template="[[tpl-ukaea-title]]" -->

# IMAS UDA presentation
## IMAS UDA workshop, ITER
#### Jonathan Hollocombe, November 2023

---
## Contents

- Background
- Low-level overviews
- UDA backend
- UDA IMAS plugin
- JSON mapping plugin

---
## Background

- UDA's plugin functionality, ability to directly call C/C++ libraries, and return the heterogenous data makes it a good fit for providing remote data and mapped data access from IMAS
- The UDA backend slots in alongside existing backends (MDS+, HDF5, etc.)
- Move to URIs in AL5 allowed for updating of UDA backend to support remote data access.
	- Remote URI:<br> `imas://<server>:<port>/uda?path=<path>&backend=<backend>`
- New `IMAS` plugin created for AL5 to handle requests coming from IMAS UDA-backend

---
## IMAS Overview

![[IMAS overview.svg|1000]]

- The high-level language bindings call the low-level library
- The calls to the low-level library are constructed from the structure of the IDS as defined in the data dictionary — either code-generated or dynamically
- The low-level maintains context for the call using `Context` objects and passes the calls through to the specified backend to do the interaction with the data file

---
## Low-level Overview

![[IMAS low-level.svg|1000]]

---
## Low-level Backend Calls

![[IMAS data flow.svg|1000]]

---

## UDA Backend

![[IMAS remote data.svg|1000]]

---
## UDA Backend

- The UDA backend supports all of the IMAS-backend functions (`openPulse`, `beginAction`, etc.)
- Each backend function generates a UDA request which is sent to the server
- For example: 
	- `openPulse(DataEntryContext* ctx, int mode)` <br> ⇒ <br> `IMAS::open(uri=<URI>, mode=<MODE>)`
	- `URI` is extracted from the `DataEntryContext` but converted from remote URI to local one, i.e. <br> `imas://server:port/uda?path=foo&backend=hdf5` ⇒ `imas:hdf5?path=foo`
	- `MODE` is a string representation of the IMAS open mode integer, i.e. `OPEN_PULSE` ⇒ `"open"`

---
## Data pre-fetching

- Handling each `readData` function separately leads to a huge number of requests being generated and sent to the server
	- Large TCP round-trip overhead amongst other inefficiencies
- To avoid this we can pre-fetch the required data in the `beginAction` or `beginArraystructAction` functions
- This requires the use of the data dictionary XML to generate the calls that will be called later from the high level access layer

---
## Data pre-fetching

![[IMAS data prefetching.svg|1000]]

---
## XML processing

- In the `beginAction` we know that the high-level is asking for an IDS
- We can use this information and the data dictionary XML to generate all the requests needed to fill this IDS
- We first have to request the IDS `ids_properties/homogeneous_flag` — this needs to be done before any other calls
- We then walk the XML generating requests for each element — including attributes extracted from the XML such as data type and rank

---
### XML processing — generated requests

```C++
ss << plugin_  
   << "::get("  
   << "uri='" << uri << "'"  
   << ", mode='" << imas::uda::convert_imas_to_uda<imas::uda::OpenMode>(open_mode_) << "'"  
   << ", dataObject='" << ids << "'"  
   << ", access='" << imas::uda::convert_imas_to_uda<imas::uda::AccessMode>(op_ctx->getAccessmode()) << "'"  
   << ", range='" << imas::uda::convert_imas_to_uda<imas::uda::RangeMode>(op_ctx->getRangemode()) << "'"  
   << ", time=" << op_ctx->getTime()  
   << ", interp='" << imas::uda::convert_imas_to_uda<imas::uda::InterpMode>(op_ctx->getInterpmode()) << "'"  
   << ", path='" << request << "'"  
   << ", datatype='" << imas::uda::convert_imas_to_uda<imas::uda::DataType>(attr.data_type) << "'"  
   << ", rank=" << attr.rank  
   << ", is_homogeneous=" << is_homogeneous  
   << ", dynamic_flags=" << imas::uda::get_dynamic_flags(attributes, request);
```

- Each request is generated with enough information that it can be called independently from previous request (`openPulse` etc.)
- In this way it is shielded from server crashes, load-balancing etc.


---
### XML processing — wild-card requests

- For the dynamically sized structures in the IDS we can't know how big the structure is on disk at this point
- We handle this by generating a wild-card request which will be expanded on the server
- I.e. `magnetics/flux_loop[:]/flux/data` will expand to <br> `magnetics/flux_loop[0]/flux/data` <br> `magnetics/flux_loop[1]/flux/data` <br> `...` <br> `magnetics/flux_loop[N]/flux/data`
- This also works for multiple levels of structures such as `magnetics/flux_loop[:]/position[:]/r`, expanding to read all elements
- This means that whilst $N$ requests are sent to the server $M$ elements are returned where $M \gg N$


---
### XML processing — wild-card requests (cont.)

- Each dynamically sized structure also has a size request which is simply that structure without the wild-card, i.e. `magnetics/flux_loop` will request the number of flux loops
- The size data along with all the returned elements can be used to populate the correct number of structures in the returned IDS
---
### XML processing — example

`get('magnetics')` ⇒
```text
magnetics/ids_properties/comment
magnetics/ids_properties/homogeneous_time
magnetics/ids_properties/source
magnetics/ids_properties/...
magnetics/flux_loop
magnetics/flux_loop[:]/position
magnetics/flux_loop[:]/position[:]/r
magnetics/flux_loop[:]/position[:]/z
magnetics/flux_loop[:]/flux/data
magnetics/flux_loop[:]/flux/time
...
```

---
## Data Fetching

- Once all the requests have generated they are then passed to UDA as a batch request
- There is a balance between number of TCP round-trips and amount of data that needs to be processed per batch of requests so a tuning parameter can be used on the URI to set number of request per batch
	- I.e. `imas://server/uda?...&batch_size=X`
	- The current default is 20 — this may change after performance studies
	- 20 requests are likely to result in 100s of elements being returned due to the wild-card request expansion

---
### Data Fetching (cont.)

- The batch request results in $X$ data blocks being returned from UDA where $X$ is the `batch_size`
- Each data block contains a Cap'n'Proto serialised list of data elements
- For non wild-card requests there will be one data element per request but for wild-card requests there will be one data element per array-struct, and sub-array-struct, and so on
- Only found data is returned, IMAS `null` values such as -999999999 for integers are not returned to avoid sending back many 'empty' nodes

---

### Data Fetching — Returned Data

![[IMAS UDA returned data.svg|1000]]

---
## Data Caching

```C++
using VariantVector = boost::variant<std::vector<int>, std::vector<double>, std::vector<char>, std::vector<double _Complex>>;  
  
struct CacheData {  
    std::vector<int> shape;  
    VariantVector values;  
};
```

- The returned data is unpacked and stored in a RAM cache using the above cache structure

---
## Data Loading

- Once we have cached data in the `beginAction` or `beginArraystructAction` then we no longer need to generate the requests in the `readData` calls
- Instead we simply return the data as stored in the cache if found
- Data not found in the cache will be defaulted to the IMAS `null` value (e.g. -999999999 for integers, etc.)
- The cache value for the read is then cleared at this point to avoid duplicating the data in memory
	- We have to copy the data on return as the returned IDS data object might outlive the UDA-backend, and hence the cache

---
## Partial Gets

- For partial gets we need to avoid retrieving the entire IDS in the cache step when only a small part of it will be requested by the `readData` calls
- This was achieved by passing through the partial get path in the `OperationContext` object
- If this path is set then we only perform the UDA request generation using the given node in the data dictionary
	- I.e. for `partial_get('flux_loop(3)')` we only need to request <br> `flux_loop[3]/flux/data` <br> `flux_loop[3]/flux/time` <br> `flux_loop[3]/position` <br> `flux_loop[3]/position[:]/r` <br> `...`

---
## Get Slice

- Handling of get slice doesn't require any changes as we already pass the range mode (`global` or `slice`) as well as the interpolation time and interpolation mode to the UDA IMAS plugin
- These options are then passed to the backend reading the data which handles the slice read operation

---
## Occurrences

- Similarly for occurrences we don't need much special handling as the occurrence is encoded in the IDS name passed to the UDA IMAS plugin, i.e. `magnetics/2` for the second occurrence of the `magnetics` IDS
- The only change required is to strip the occurrence number from the IDS before we look up the node in the XML
- The IDS name with occurrence attached is passed to the UDA IMAS plugin to enable reading of the correct occurrence

---
## UDA IMAS plugin

- The second half of the IMAS remote data & mapping functionality is the UDA IMAS plugin that is required to respond to the requests being generated by the IMAS UDA-backend
- 

- This section covers the functionality of the UDA IMAS plugin

---
## Function overview

- Outside of the standard plugin functions normally provided (`help`, `version`, `builddate`, etc.) the following functions are defined on the IMAS plugin:
	- `open(uri, mode)`
	- `close(uri, mode)`
	- `get(uri, access, range, time, interp, path, datatype, rank, is_homogeneous, dynamic_flags)`

---
### Function overview — open

- The open function handles two cases:
	1. If the URI contains `machine` then all further calls are forwarded onto the mapping plugin responsible for the given machine (this case will be discussed in later session)
	2. Otherwise uses the IMAS low-level library to find and open the IMAS pulse file(s) and caches the resultant `Context` handle for later use (i.e. in later `get` calls)

---
### Function overview — close

- Similarly the close function  handles the two cases:
	1. If the URI is for a mapped experiment nothing needs to be done
	2. If the URI is for remote data access then the IMAS low-level is used to close the pulse file(s)

---
### Function overview — get

- The get function is where the magic happens, it performs the following steps:

1. If the given URI is not already open then call the open function to do this — this is so the IMAS plugin works even if the functions are called out of sequence (i.e. due to server crash or load balancing)
2. The provided IMAS path is then split into tokens using `/` as the separator, i.e. <br> `magnetics/flux_loop[:]/flux/data` <br> ⇒ <br> `[magnetics, flux_loop[:], flux, data]`

---
### Function overview — get (remote data access)

3. We then loop over the tokens:
	- If the token is an arraystruct we use the IMAS low-level to create the `ArraystructContext`, also finding the size of the array, and for each element $0..(size-1)$ we loop the remaining tokens
	- If the token is not an arraystruct we build the path of the element and use the IMAS low-level to read the data using the current `Context` objects
4. The found array sizes and data are serialised using Cap'n'Proto and returned to the client on via the data block 

---
### Function overview — get example

```text
[magnetics, flux_loop[:], flux, data]

magnetics        => al_begin_global_action (or slice)
flux_loop[:]     => al_begin_arraystruct_action => size
flux_loop[0] ArraystructContext
	flux/data    => al_read_data
flux_loop[1] ArraystructContext
	flux/data    => al_read_data
...
flux_loop[size-1] ArraystructContext
	flux/data    => al_read_data
```

```text
{ 'magnetics/flux_loop': size },
{ 'magnetics/flux_loop[1]/flux/data': [...] }
{ 'magnetics/flux_loop[2]/flux/data': [...] }
...
{ 'magnetics/flux_loop[size-1]/flux/data': [...] }
```

---
### Function overview — get (mapped data)

3. As for remote data access we loop over the tokens:
	- For both arraystruct and non-arraystruct tokens we generate a plugin request <br> `PLUGIN::get(machine=, path=, rank=, shape=, datatype=, <args>)`
	- `machine` is the machine as provided on the URI
	- `path` is the current IDS path, i.e. `magnetics/flux_loop[3]/flux/data`
	- `rank` is the expected data rank
	- `datatype` is the expected data type
	- `<args>` are any other args passed on the URI, i.e. `imas://server/uda?machine=MASTU&shot=30420&run=0` would make `<args>` = `"shot=30420, run=0"`
1. All the data returned by the mapping plugin are serialised using Cap'n'Proto and returned to the client on via the data block

---
### Mapped data — plugin selection

- When the IMAS plugin processing a mapped data request it must select a plugin to forward the request to
- The selection is done via a configuration file loaded when the IMAS plugin first initialises
- The location of this configuration files is specified by the environmental variable `UDA_IMAS_MACHINE_MAP` via the server configuration

---
### Mapped data — plugin configuration file

- This file is a white space separated configuration with 5 columns: <br> `MACHINE IDS_NAME HOST PORT PLUGIN_NAME`
	- `MACHINE` is the machine looked for in the URI
	- `IDS_NAME` restricts this configuration entry to a single IDS, or `*` to apply to all IDSs
	- `HOST` is the server host to forward the request to, or `-` to call a plugin on the current server
	- `PORT` is the server port to forward the request to, or `-` to call a plugin on the current server
	- `PLUGIN_NAME` is the name of the plugin to call


---
### Mapped data — plugin configuration example

```text
DRAFT   magnetics -          -     IMAS_JSON_MAP  
MASTU   *         -          -     IMAS_JSON_MAP  
JET     *         uda.jet.uk 60000 JET_MAP
```

1. Uses the `IMAS_JSON_MAP` plugin on the local server when `machine=DRAFT` for the `magnetics` IDS only (other IDSs will throw an error)
2. Uses the `IMAS_JSON_MAP` plugin on the local server for all IDSs when `machine=MASTU`
3. Uses the `JET_MAP` plugin on the `uda.jet.uk:60000` server for all IDSs when `machine=JET`

> For most machines we hope that the `IMAS_JSON_MAP` plugin will be capable of handling your mapping requirements

---
### Mapped data — JSON mapping plugin

- The requests from the IMAS plugin to the selected mapping plugin need to be handled for the specified `path`, `rank` & `datatype` using whatever extra `<args>` are passed using the URI.
- Details of how this can be achieved will be show in the presentation on the JSON mapping plugin and mapping plugin development hands-on.