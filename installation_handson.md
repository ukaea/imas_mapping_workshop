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

# Installation Hands-On
## IMAS UDA workshop, ITER
#### Jonathan Hollocombe, November 2023

---
## Contents

- Install dependencies
- Git clone
- CMake config
- Build
- Install
- Installing pyuda
- Configure server
	- machine.d
- Test connection
	- Telnet (or nc -zv)
	- uda_cli
- Running tests
- Installing plugins
- Test plugin calls
- SSL authentication
- Debugging a server

---
## UDA installation workshop

This workshop will cover

- Building and installation of UDA
- Configuring a UDA server
- Testing UDA server connection
- Installing plugins
- SSL authentication
- Debugging (time depending)

---
## Workshop format

- Each section will have some technical details
- Any hands-on exercises will be marked with __Task__
- We will stop at each task to let people complete them
- Please ask questions as we go

---
## UDA dependencies

- Required dependencies:
	- `git`
	- `boost`
	- `ssl`
	- `cmake`
	- C++ compiler with C++17 support
	- `pkg-config`
	- `libxml2`
	- `libspdlog`
- Optional dependencies:
	- `python` - to build pyuda
	- `capnproto` - for Cap'n'Proto support
	- `ninja-build` - to use Ninja generator

---

## Install dependencies

```bash
RUN apt-get update && apt-get install -y \  
    git \  
    libboost-dev \  
    libboost-program-options-dev \  
    libssl-dev \  
    cmake \  
    build-essential \  
    pkg-config \  
    libxml2-dev \  
    libspdlog-dev \  
    ninja-build \  
    capnproto \  
    libcapnp-dev \  
    python3-dev \  
    python3-pip \  
    python3-venv
```

---
## Clone UDA

<div class="task">
Task
</div>

```bash
git clone https://github.com/ukaea/UDA.git -b 2.7.3 uda
cd uda
```

- This performs a read-only clone using https connection
- Following slides assume we are in the `uda` directory

---
## CMake configuration

<div class="task">
Task
</div>

```bash
cmake -G Ninja -B build \  
-DBUILD_SHARED_LIBS=ON \  
-DCMAKE_BUILD_TYPE=Debug \  
-DSSLAUTHENTICATION=ON \  
-DCLIENT_ONLY=OFF \  
-DENABLE_CAPNP=ON \
-DCMAKE_INSTALL_PREFIX=install
```

- `BUILD_SHARED_LIBS` is required when not using `CLIENT_ONLY`
- `CMAKE_BUILD_TYPE` can be `Debug` or `Release`
- `SSLAUTHENTICATION` builds UDA with SSL support
- `CLIENT_ONLY` specifies whether to only build client libraries
- `ENABLE_CAPNP` builds UDA with Cap'n'Proto support

---
## Build

<div class="task">
Task
</div>

```bash
cmake --build build/
```

- Building via `cmake` avoids worrying about what generator has been used, i.e. running `make -C build/` vs `ninja -C build/`

---
## Install

<div class="task">
Task
</div>

```bash
cmake --install build/
```

- This will install into directory specified by `CMAKE_INSTALL_PREFIX` in the configure step.
- If `CMAKE_INSTALL_PREFIX` is not specified it will try and install in the standard system path, i.e. `/usr/local` on Linux.

---

- You should now have in the install directory:

```text
install/
	bin/                   Contains the server and CLI binaries
	etc/                   Contains all the server and plugin configuration files
	include/               Contains the UDA headers
	lib/                   Contains the UDA libraries
	modulefiles/           Contains the module files
	python_installer/      Files to install pyuda
```

---
## Server configuration directory

- Inside the `install/etc` directory you will find the following:

```text
machine.d/            Configuration files for specific machines
plugins/              Contains the configuration files to specify while plugins to load
plugins.d/            Contains any configuration files that the plugins require
rc.uda                Script used to launch a development xinetd service
README.md             Details on use of rc.uda
udagenstruct.conf     Internal configuration file used by the server
udaserver.cfg         Runtime configuration for the UDA server
udaserver.sh          Server launch script
xinetd.conf           Xinetd options for running the server
```

- Configuration file from `machine.d` is determined by `dnsdomainname` — This can overridden by setting `UDAHOSTNAME` environmental variable in `udaserver.cfg`

---
## Checking the server

<div class="task">
Task
</div>

```bash
cd install/etc

./udaserver.sh
```

- Will hang if successful &#8212; xinetd communicates with process on stdin/stdout so server is waiting for input
- `CTRL + C` to kill the server
- You should now see some debug files in the `etc` directory:

```text
etc/
  DebugServer.log   Debug logging for the server
  Error.log         Error logging for the server
  Access.log        Access logging for the server
  server.log        Any outputs written by the configuration scripts
  startup.log       Anything written by the server before XDR streams are created
```

- Last 2 are there to protect against anything breaking the server by writing to stdout
---
### DebugServer.log

Inside the DebugServer.log you should see:

```text
2023:11:10T09:29:53.446219Z, getServerEnvironment.cpp:12 >> Server Environment Variable values  
2023:11:10T09:29:53.446978Z, getServerEnvironment.cpp:13 >> Log Location    : /Users/jhollocombe/CLionProjects/UDA/etc/  
2023:11:10T09:29:53.446989Z, getServerEnvironment.cpp:14 >> Log Write Mode  : a  
2023:11:10T09:29:53.446996Z, getServerEnvironment.cpp:15 >> Log Level       : 1  
2023:11:10T09:29:53.447003Z, getServerEnvironment.cpp:16 >> External User?  : 0  
2023:11:10T09:29:53.447009Z, getServerEnvironment.cpp:17 >> UDA Proxy Host  :   
2023:11:10T09:29:53.447015Z, getServerEnvironment.cpp:18 >> UDA This Host   :   
2023:11:10T09:29:53.447021Z, getServerEnvironment.cpp:19 >> Private File Path Target    :   
2023:11:10T09:29:53.447026Z, getServerEnvironment.cpp:20 >> Private File Path Substitute:   
2023:11:10T09:29:53.447032Z, udaServer.cpp:1125 >> New Server Instance  
2023:11:10T09:29:53.447080Z, udaServer.cpp:1149 >> XDR Streams Created  
2023:11:10T09:29:53.572025Z, udaServer.cpp:1191 >> List of Plugins available  
2023:11:10T09:29:53.572073Z, udaServer.cpp:1193 >> [0] 2 GENERIC  
2023:11:10T09:29:53.572084Z, udaServer.cpp:1193 >> [1] 2 SERVERSIDE  
2023:11:10T09:29:53.572092Z, udaServer.cpp:1193 >> [2] 2 SSIDE  
2023:11:10T09:29:53.572100Z, udaServer.cpp:1193 >> [3] 2 SS  
2023:11:10T09:29:53.572108Z, udaServer.cpp:1193 >> [4] 1000 BYTES  
2023:11:10T09:29:53.572116Z, udaServer.cpp:1193 >> [5] 1001 HELP  
2023:11:10T09:29:53.572124Z, udaServer.cpp:1193 >> [6] 1002 TEMPLATEPLUGIN  
2023:11:10T09:29:53.572131Z, udaServer.cpp:1193 >> [7] 1003 TESTPLUGIN  
2023:11:10T09:29:53.572139Z, udaServer.cpp:1193 >> [8] 1004 UDA  
2023:11:10T09:29:53.572146Z, udaServer.cpp:1193 >> [9] 1005 VIEWPORT  
2023:11:10T09:29:53.572178Z, udaServer.cpp:1035 >> Waiting for Initial Client Block
```
- Each line starts with a timestamp and the source location of the logging
- You can see the list of the plugins that have been located and successfully loaded

---
## Configure the server

- Inside the `machine.d` directory you'll see the current machine configuration files:

```text
etc/machine.d/
  hpc.l.cfg
  iter.org.cfg
  itm.rzg.mpg.de.cfg
  jet.uk.cfg
  mast.l.cfg
  uda-external-jet.cfg
```

---
### Configure the server (cont.)

- Create a new configuration file to match the name of the VM:

<div class="task">
Task
</div>

```bash
cd etc/machine.d/
touch <TODO>.cfg
```

- The name of this file must match the result of `dnsdomainname` (or `hostname -f` if `dsndomainname` doesn't exist)
- This can be overridden by setting `export UDAHOSTNAME=...` in `udaserver.cfg`, in which case the file should be `$UDAHOSTNAME.cfg`
---
### Machine configuration file

- The machine configuration file can be used to set any environmental variables required by your UDA server instance
- The machine configuration file is read after the `udaserver.cfg` file so can overwrite any options set there
- Some important variables are:
	- `UDA_LOG` &#8212; the directory where the log files should be written
	- `UDA_LOG_MODE` &#8212; mode used to create the log file, i.e. `w` or `a`
	- `UDA_LOG_LEVEL` &#8212; level of server logging (`INFO`, `DEBUG`, `WARN`, `ERROR` or `ACCESS`)
	- `UDA_PLUGIN_CONFIG` &#8212; the location of the `udaPlugins.conf` file which specifies which plugins to load
	- `UDA_PLUGIN_DIR` &#8212; the location of the plugin libraries to load (the server will also check the `LD_LIBRARY_PATH`)
	- `LD_LIBRARY_PATH` &#8212; this can be used to append any additional paths need to load dynamic libraries needed by the plugins, i.e. the MDS+ libraries

---
## Running a test server

<div class="task">
Task
</div>

```bash
./rc.uda start
```

- The `rc.uda` script launches a `xinetd` service in user space. In production you would use the system `xinetd` but for testing and development it is easier to do it this way.
- To see the status of the service run:

<div class="task">
Task
</div>

```bash
./rc.uda status
```
---
### Running a test server (cont.)

- In the `etc/` directory you will also now see:

```text
mylog.<MACHINE>.local        - xinetd service log file
xinetd.<MACHINE>.local.pid   - file containing the pid of the local service
```

---
## Xinetd configuration

```text
service uda  
{  
    disable         = no
    flags           = IPv4 NOLIBWRAP KEEPALIVE NODELAY
    type            = UNLISTED
    protocol        = tcp
    port            = 56565 
    socket_type     = stream
    wait            = no
    user            = jhollocombe
    server          = /Users/jhollocombe/CLionProjects/UDA/etc/udaserver.sh
  
    instances       = 100
    per_source      = 30
  
    v6only          = no
    groups          = yes
    umask           = 002
  
    log_type        = FILE /Users/jhollocombe/CLionProjects/UDA/etc/xinetd.logfile
    log_on_success  += DURATION USERID
    log_on_failure  += USERID
}
```
- `port` specifies the port to run on, UDA runs on 56565 by convention
- `user` is the user under which the server is run
- `server` is the server script to execute
- `instances` and `per_source` are DDOS protection options
- `log_` options specify where and what to log from xinetd


---
## Test the connection

<div class="task">
Task
</div>

```bash
nc -zv localhost 56565
```

- This tests that the socket is open and receiving connections

<div class="task">
Task
</div>

```bash
uda_cli -h localhost -p 56565 "help::help()"
```

- Runs a simple plugin request against the server

```text
request: help::help()

Help    List of HELP plugin functions:

services()      Returns a list of available services with descriptions
ping()          Return the Local Server Time in seconds and microseonds
servertime()    Return the Local Server Time in seconds and microseonds
```

---
## Running test suite

<div class="task">
Task
</div>

```bash
cd build/test/plugins
./plugin_test_testplugin
```

```text
===============================================================================
All tests passed (1108166 assertions in 48 test cases)
```

---
## Installing plugins

- Plugins are built against an installed UDA server
- Steps to install a plugin:
	- Clone the plugin repo
	- Configure, build & install
	- Add the plugin to the servers `udaPlugins.conf`

- For the ITER plugins we use CMake to configure the build and generate an `activate-plugins.sh` script to handle the last step

---
## Installing plugins &#8212; clone

<div class="task">
Task
</div>

```bash
git clone https://git.iter.org/scm/imas/uda-plugins.git
cd uda-plugins
```

---
## Installing plugins &#8212; configure

<div class="task">
Task
</div>

```bash
export PKG_CONFIG_PATH=$HOME/uda/install/lib/pkgconfig
cmake -Bbuild -DCMAKE_BUILD_TYPE=Debug -DBUILD_PLUGINS=imas -DCMAKE_INSTALL_PREFIX=$HOME/uda/install
```

- The `PKG_CONFIG_PATH` environmental variable let `pkg-config` know where to find UDA
- The `BUILD_PLUGINS` CMake option specifies which plugins in the repo we want to build &#8212; for remote and mapped data for AL5 we only need the `IMAS` plugin


<div class="task">
Task
</div>

```bash
cmake --build build/
cmake --install build/
```

---
## Installing plugins &#8212; activate

<div class="task">
Task
</div>

```bash
./build/scripts/activate-plugins.sh
```

```bash
#!/bin/bash  
  
UDA_HOME=/home/test/uda/install
PLUGINS_HOME=/home/test/uda/install
  
for PLUGIN in IMAS
do  
    $UDA_HOME/bin/install_plugin -u $PLUGINS_HOME install $PLUGIN  
done
```

- The generated `activate-plugins.sh` script calls the `install_plugin` helper script installed with UDA for each plugin being built
- The `install_plugin` script takes the plugin configuration line for the plugin, found in the `udaPlugins_<PLUGIN_NAME>.conf` and copies it into the servers `udaPlugins.conf`

---
## Testing plugins

<div class="task">
Task
</div>

```
uda_cli -h locahost -p 56565 "IMAS::help()"
```

- Runs the help function on the `IMAS` plugin &#8212; demonstrates that the plugin is installed and functioning

```text
... output ...
```

---
## SSL authentication

- Building with SSL
- Server keys
- SSL environmental vars

---
### Using SSL client

- Client keys
- Environmental vars
- Calling server

---
## Installing a production server

```bash
cp xinetd.conf /etc/xinetd.d/uda
service xinetd restart
```

- The service name (`uda` above) can be anything you like and there might be more than 1 if you have multiple services running (i.e. `uda-internal` and `uda-external`)

---
## Debugging

- Client-server debugging
- Breakpointing client
- Finding server instance
- Hooking debugger onto server
- Breakpoint plugin
- Running to breakpoint

---
## Installing pyuda

```bash
cd
python3 -m venv venv
source venv/bin/activate
pip3 install --upgrade pip
pip3 install uda/install/python_installer
```

- If you've installed UDA in another directory then you will need to change the last line to `pip3 install <INSTALL_DIR>/python_installer`