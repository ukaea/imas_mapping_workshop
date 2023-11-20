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
# IMAS-UDA Hands-On
#### _<u>Jonathan Hollocombe</u>, Adam Parker, Stephen Dixon_ <br> November 20, 2023 &#8212; ITER

---
## Overview

- This hands-on will cover how to install & configure UDA
- It will then cover how to use UDA as part of IMAS
- It will be split into 2 sessions depending on how much material we get through in the time
- If time permits additional topics can be covered such as SSL, debugging, etc.
	- If not the material will be available for offline exercises

---
## Contents

- Installing and configuring a UDA server
- Using UDA CLI & pyuda
- Building and installing plugins
- Using UDA in IMAS
- Configuring UDA SSL
- Debugging UDA using gdb
- Installing UDA production server

---
## Workshop format

- Each section will have some technical details
- Any hands-on exercises will be marked with __Task \#N__
- We will stop at each task to let people complete them before moving on
- Please ask questions as we go

---
## Connection details

- Please connect to the VM from the SDCC login nodes:
 
```bash
ssh user@io-ls-udatrain##
```

- Password is also `user`
- Replace the `##` above with the VM number you have been assigned, i.e. if you have VM#03 you would log on as follows:

```bash
ssh user@io-ls-udatrain03
```

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
dnf group install -y 'Development Tools' && \  
dnf install -y \  
    git \  
    wget \  
    boost-devel \  
    openssl-devel \  
    cmake \  
    libxml2-devel \  
    libtirpc-devel \  
    python3-devel \  
    python3-pip
```

- These dependencies have already been installed on the VMs

---
## Clone UDA

<div class="task">
Task #1: Clone UDA
</div>

```bash
git clone https://github.com/ukaea/UDA.git -b release/2.7.4 uda
cd uda
```

- This performs a read-only clone using https connection
- Following slides assume we are in the `uda` directory

---
## CMake configuration

<div class="task">
Task #2: Configure CMake
</div>

```bash
cmake -B build \  
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
- `CMAKE_INSTALL_PREFIX` specifies where to install UDA (defaults to `/usr/local`)

---
## Build

<div class="task">
Task #3: Build UDA
</div>

```bash
cmake --build build/ -j 2
```

- Building via `cmake` avoids worrying about what generator has been used, i.e. running `make -C build/` vs `ninja -C build/`
- The `-j` flags specifies the number of jobs to use for a parallel build

---
## Install

<div class="task">
Task #4: Install UDA
</div>

```bash
cmake --install build/
```

- This will install into directory specified by `CMAKE_INSTALL_PREFIX` in the configure step.
- If `CMAKE_INSTALL_PREFIX` is not specified it will try and install in the standard system path, i.e. `/usr/local` on Linux.

---
## Installed files

- You should now have in the install directory:

```bash
$ tree -L 1 install
install
├── bin                  # Contains the server and CLI binaries
├── etc                  # Contains all the server and plugin configuration files
├── include              # Contains the UDA headers
├── lib                  # Contains the UDA libraries
├── modulefiles          # Contains the module files
└── python_installer     # Files to install pyuda
```

---
## Server configuration directory

- Inside the `install/etc` directory you will find the following:

```bash
$ tree -L 1 install/etc/
install/etc/
├── machine.d           # Configuration files for specific machines
├── plugins             # Contains the configuration files to specify while plugins to load
├── plugins.d           # Contains any configuration files that the plugins require
├── rc.uda              # Script used to launch a development xinetd
├── README.md           # Details on use of rc.uda
├── udagenstruct.conf   # Internal configuration file used by the server
├── udaserver.cfg       # Runtime configuration for the UDA server
├── udaserver.sh        # Server launch script
└── xinetd.conf         # Xinetd options for running the server
```


---
## Configure the server

<div class="task">
Task #5: Navigate to the server configuration directory
</div>

```bash
cd install/etc
```

- Inside the `machine.d` directory you'll see the current machine configuration files:

```bash
$ tree machine.d/
machine.d/
├── hpc.l.cfg
├── iter.org.cfg
├── itm.rzg.mpg.de.cfg
├── jet.uk.cfg
├── mast.l.cfg
└── uda-external-jet.cfg
```

- The configuration file to load from `machine.d` is determined by `dnsdomainname` — but this can overridden by setting `UDAHOSTNAME` environmental variable in `udaserver.cfg`

---
### Configure the server (cont.)

- Create a new configuration file to match the name of the VM:

<div class="task">
Task #6: Create machine configuration file
</div>

```bash
touch machine.d/udatrain.cfg
```

- The name of this file must match the result of `dnsdomainname` (or `hostname -f` if `dsndomainname` doesn't exist)
- This can be overridden by setting `export UDAHOSTNAME=...` in `udaserver.cfg`, in which case the file should be `$UDAHOSTNAME.cfg`

<div class="task">
Task #7: Update server configuration file
</div>

```bash
vim udaserver.cfg
```
- Add `export UDAHOSTNAME=udatrain` to the top of this

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
## Checking the server

<div class="task">
Task #8: Test UDA server script
</div>

```bash
./udaserver.sh
```

- Will hang if successful &#8212; xinetd communicates with process on stdin/stdout so server is waiting for input
- `CTRL + C` to kill the server
- You should now see some debug files in the `etc` directory:

```bash
$ ls -1 *.log
DebugServer.log   # Debug logging for the server
Error.log         # Error logging for the server
Access.log        # Access logging for the server
server.log        # Any outputs written by the configuration scripts
startup.log       # Anything written by the server before XDR streams are created
```

- Last 2 are there to protect against anything breaking the server by writing to stdout
---
### DebugServer.log

Inside the `DebugServer.log` you should see:

```text
$ cat DebugServer.log
2023:11:16T11:53:29.6583Z, getServerEnvironment.cpp:12 >> Server Environment Variable values
2023:11:16T11:53:29.6775Z, getServerEnvironment.cpp:13 >> Log Location    : /home/user/uda/install/etc/
2023:11:16T11:53:29.6790Z, getServerEnvironment.cpp:14 >> Log Write Mode  : a
2023:11:16T11:53:29.6795Z, getServerEnvironment.cpp:15 >> Log Level       : 1
2023:11:16T11:53:29.6799Z, getServerEnvironment.cpp:16 >> External User?  : 0
2023:11:16T11:53:29.6804Z, getServerEnvironment.cpp:17 >> UDA Proxy Host  :
2023:11:16T11:53:29.6808Z, getServerEnvironment.cpp:18 >> UDA This Host   :
2023:11:16T11:53:29.6812Z, getServerEnvironment.cpp:19 >> Private File Path Target    :
2023:11:16T11:53:29.6816Z, getServerEnvironment.cpp:20 >> Private File Path Substitute:
2023:11:16T11:53:29.6821Z, udaServer.cpp:1125 >> New Server Instance
2023:11:16T11:53:29.6934Z, udaServer.cpp:1149 >> XDR Streams Created
2023:11:16T11:53:29.108707Z, udaServer.cpp:1191 >> List of Plugins available
2023:11:16T11:53:29.108745Z, udaServer.cpp:1193 >> [0] 2 GENERIC
2023:11:16T11:53:29.108752Z, udaServer.cpp:1193 >> [1] 2 SERVERSIDE
2023:11:16T11:53:29.108782Z, udaServer.cpp:1193 >> [2] 2 SSIDE
2023:11:16T11:53:29.108788Z, udaServer.cpp:1193 >> [3] 2 SS
2023:11:16T11:53:29.108793Z, udaServer.cpp:1193 >> [4] 1000 BYTES
2023:11:16T11:53:29.108797Z, udaServer.cpp:1193 >> [5] 1001 NEWHDF5
2023:11:16T11:53:29.108802Z, udaServer.cpp:1193 >> [6] 1002 HELP
2023:11:16T11:53:29.108806Z, udaServer.cpp:1193 >> [7] 1003 TEMPLATEPLUGIN
2023:11:16T11:53:29.108810Z, udaServer.cpp:1193 >> [8] 1004 TESTPLUGIN
2023:11:16T11:53:29.108814Z, udaServer.cpp:1193 >> [9] 1005 UDA
2023:11:16T11:53:29.108818Z, udaServer.cpp:1193 >> [10] 1006 VIEWPORT
2023:11:16T11:53:29.108828Z, udaServer.cpp:1035 >> Waiting for Initial Client Block
```
- Each line starts with a timestamp and the source location of the logging
- You can see the list of the plugins that have been located and successfully loaded

---
## Running a test server

<div class="task">
Task #9: Start server
</div>

```bash
./rc.uda start
```

- The `rc.uda` script launches a `xinetd` service in user space. In production you would use the system `xinetd` but for testing and development it is easier to do it this way.
- To see the status of the service run:

<div class="task">
Task #10: Check server status
</div>

```bash
./rc.uda status
```
---
### Running a test server (cont.)

- In the `etc/` directory you will also now see:

```text
mylog.io-ls-udatrain01.iter.org        - xinetd service log file
xinetd.io-ls-udatrain01.iter.org.pid   - file containing the pid of the local service
xinetd.logfile                         - xinetd logfile
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
Task #11: Check server socket connection
</div>

```bash
nc -zv localhost 56565
```

- This tests that the socket is open and receiving connections

<div class="task">
Task #12: Use UDA CLI to test server
</div>

```bash
export PATH=$PATH:$HOME/uda/install/bin
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
## UDA CLI

```bash
$ uda_cli --help
Usage: uda_cli [options] request
Allowed options:
  --help                         produce help message
  -h [ --host ] arg (=localhost) server host name
  -p [ --port ] arg (=56565)     server port
  --request arg                  request
  --source arg                   source
  --ping                         ping the server
```

- UDA CLI is a small helper program that can make requests to the server and print the resulting data to the console, e.g.:

```bash
$ uda_cli -h localhost -p 56565 "testplugin::capnp()"
request: testplugin::capnp()
name: root
  name: double_array
  data: [0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, ... (30 elements)]
  name: i32_array
  data: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, ... (100 elements)]
  name: i64_scalar
  data: 999
```

---
## Running test suite

<div class="task">
Task #13: Run UDA tests
</div>

```bash
cd build/test/plugins
export UDA_HOST=localhost
export UDA_PORT=56565
./plugin_test_testplugin
```

- Expected output:

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
Task #14: Clone plugins repo
</div>

```bash
git clone -c http.extraheader="Authorization: Bearer $PLUGIN_TOKEN" https://git.iter.org/scm/imas/uda-plugins.git -b release/1.4.0
cd uda-plugins
```

---
## Installing plugins &#8212; configure

<div class="task">
Task #15: Configure plugins
</div>

```bash
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/home/user/uda/install/lib/pkgconfig
cmake -Bbuild -DCMAKE_BUILD_TYPE=Debug -DBUILD_PLUGINS=imas -DCMAKE_INSTALL_PREFIX=$HOME/uda/install
```

- The `PKG_CONFIG_PATH` environmental variable let `pkg-config` know where to find UDA & IMAS
- The `BUILD_PLUGINS` CMake option specifies which plugins in the repo we want to build &#8212; for remote and mapped data for AL5 we only need the `IMAS` plugin
- `CMAKE_INSTALL_PREFIX` sets where to install the plugins &#8212;  usually set to the UDA server installation directory


<div class="task">
Task #16: Build and install plugins
</div>

```bash
cmake --build build/ -j 2
cmake --install build/
```

---
## Installing plugins &#8212; activate

- The generated `build/scripts/activate-plugins.sh` script calls the `install_plugin` helper script installed with UDA for each plugin being built
- The `install_plugin` script takes the plugin configuration line for the plugin, found in the `udaPlugins_<PLUGIN_NAME>.conf` and copies it into the servers `udaPlugins.conf`

```bash
#!/bin/bash  
  
UDA_HOME=/home/test/uda/install
PLUGINS_HOME=/home/test/uda/install
  
for PLUGIN in IMAS
do  
    $UDA_HOME/bin/install_plugin -u $PLUGINS_HOME install $PLUGIN  
done
```


<div class="task">
Task #17: Activate plugins
</div>

```bash
./build/scripts/activate-plugins.sh
```

---
## Updating server configuration

- In the `install/etc/machine.d` directory edit the `udatrain.cfg` configuration that was created earlier.

<div class="task">
Task #18: Update configuration
</div>

```bash
cd $HOME/uda/install/etc
vim machine.d/udatrain.cfg
```

- Add to the configuration file:

```bash
export UDA_IMAS_MAPPINGS_FILE=/home/user/uda-plugins/source/imas/mappings.txt
```

- This defines the `UDA_IMAS_MAPPINGS_FILE` variable required by the IMAS plugin to locate the mappings configuration

---
## Testing plugins

<div class="task">
Task #19: Test plugin with UDA CLI
</div>

```bash
uda_cli -h localhost -p 56565 "IMAS::help()"
```

- Runs the help function on the `IMAS` plugin &#8212; demonstrates that the plugin is installed and functioning

```text
request: imas::help()
# IMAS plugin

The IMAS plugin is responsible for responding to requests from the IMAS UDA backend.
The low level IMAS calls are mapped to plugin functions which are then used
to read the data using either a local version of the IMAS backend or another UDA plugin
to map non-IMAS data into the IMAS structure.

## Functions

The IMAS plugin responds to the following plugin requests:

...
```

---
## Calling IMAS plugin

<div class="task">
Task #20: Opening IMAS data using plugin directly
</div>

```bash
uda_cli -h localhost -p 56565 "imas::open(uri='imas:hdf5?path=/home/user/data/122319/1', mode='open')"
```
- This should return `1` which is the indication that the file was opened successfully
- This is the type of plugin call that is generated by the IMAS UDA backend, it is not designed to be called manually but can be useful for debugging

---
## Using UDA with (python) IMAS

<div class="task">
Task #21: Opening remote data with IMAS
</div>

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/uda/install/lib
python3
```

In `python3` run the following command:

```python
>>> import imas
>>> entry = imas.DBEntry('imas://localhost:56565/uda?path=/home/user/data/105029/1&backend=hdf5&verbose=1', 'r')
>>> entry.open()
```

<div class="task">
Task #22: Getting data using IMAS
</div>

In the same `python3` session:

```python
>>> mag = entry.get('magnetics')
>>> flux_loop = entry.partial_get('magnetics' ,'flux_loop(3)')
>>> eq_slice = entry.get_slice('equilibrium', time_requested=0.1, interpolation_method=imas.imasdef.CLOSEST_INTERP)
```

---
## Installing JSON mapping plugin

<div class="task">
Task #23: Clone the repo
</div>

```bash
cd $HOME
git clone ... json-plugin
cd json-plugin
```

<div class="task">
Task #24: Install and activate
</div>

```bash
cmake -B build -DCMAKE_INSTALL_PREFIX=$HOME/uda/install
cmake --build build/ -j 2
cmake --install build/
./build/scripts/activate-plugins.sh
```

---
## Debugging UDA

<div class="task">
Task #25: Breaking in UDA client
</div>

```bash
gdb --args uda_cli -h localhost -p 56565 "help::help()"
break udaClient.cpp:743
run
```

---
## Debugging UDA &#8212; server

<div class="task">
Task #26: Find pid of UDA server
</div>

```bash
ps -fe | uda_server
```

<div class="task">
Task #27: Hook gdb onto running server
</div>

```bash
gdb -p <PID>
```

<div class="task">
Task #28: Breaking in plugin
</div>

```bash
break help_plugin.cpp:66
cont
```

---
## Debugging UDA &#8212; client

<div class="task">
Task #29: Continue client
</div>

```
cont
```

---
## Installing a production server

<div class="task">
Task #30: Copy xinetd configuration
</div>

```bash
cp xinetd.conf /etc/xinetd.d/uda
service xinetd restart
```

- The service name (`uda` above) can be anything you like and there might be more than 1 if you have multiple services running (i.e. `uda-internal` and `uda-external`)

---
## Installing pyuda

<div class="task">
Task #31: Install pyuda
</div>

```bash
cd
python3 -m venv venv
source venv/bin/activate
pip3 install --upgrade pip
pip3 install uda/install/python_installer
```

- If you've installed UDA in another directory then you will need to change the last line to `pip3 install <INSTALL_DIR>/python_installer`

---

## Setting up SSL authenticated server

<div class="task">
Task #32: Copying server certificates
</div>

```bash
mkdir $HOME/.uda/
cp <keys>/* $HOME/.uda/
```

<div class="task">
Task #33: Configuring UDA server for SSL
</div>

```bash
export UDA_SERVER_SSL_AUTHENTICATE=1
export UDA_SERVER_SSL_CERT=...
export UDA_SERVER_SSL_KEY=...
export UDA_SERVER_CA_SSL_CERT=...
export UDA_SERVER_CA_SSL_CRL=...
```

---
## SSL authenticated client

<div class="task">
Task #34: Configuring UDA client for SSL
</div>

```bash
export UDA_CLIENT_SSL_AUTHENTICATE=1
export UDA_CLIENT_SSL_CERT=...
export UDA_CLIENT_SSL_KEY=...
export UDA_CLIENT_CA_SSL_CERT=...
```

<div class="task">
Task #35: Checking SSL connection
</div>

```bash
uda_cli -h localhost -p 56565 "help::help()"
```
