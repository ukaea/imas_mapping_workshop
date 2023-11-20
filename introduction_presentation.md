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
# Introduction
#### _<u>Jonathan Hollocombe</u>, Adam Parker, Stephen Dixon_ <br> November 20, 2023 &#8212; ITER

---
## Contents

- Motivation
- Background
- Schedule

---
## Motivation

- This workshop is aiming to give you an insight into what UDA is, how it works and how it can be used in IMAS to map experimental data
- Experimental data in IMAS format is needed to validate ITER software and can help creating multi-machine databases for AI/ML
- There will be hands-on session to cover how to set up a UDA server, develop JSON mappings and create a mapping plugin for your device
- At the end of this workshop you should have the knowledge of the basic building blocks needed to develop mappings for any machine
- Future training and documentation can build on this knowledge for machine specific cases

---
## Background (UDA)

- UDA &#8212; **U**niversal **D**ata **A**ccess (formerly known as IDAM) is a client-server technology originally created for the MAST experiment
- Designed to separate data access and format from transferring that data across the network to a user
- Dynamically loaded plugin architecture allows for data access plugins to be written without changing UDA core code
- Consists C/C++ client library which talks to a C/C++ server running behind xinetd
- Several higher level language wrappers available, the most popular of which is pyuda &#8212; the Python wrapper

---
## Background (IMAS-UDA)

- UDA has been used for several years to provide mapped data access in IMAS
- Previous efforts have built upon existing work (EXP2ITM) and relied on somewhat shaky infrastructure stacks to enable this
- With release of AL5 and introduction of IMAS URIs UDA can now be used to read IMAS data files remotely
- Building on the improved UDA backend and plugins for AL5 the JSON mapping plugin is designed to provide a better way to write and manage the mappings for multiple machines

---
## Schedule (Day 1)

| Name | Type | Description |
| -----|-------| ------------|
| UDA Overview | Presentation | Detailing how UDA works to provide context for it's use in IMAS |
| IMAS UDA | Presentation | Detailing how UDA is used in IMAS remote data access and experimental data mapping |
| UDA Installation | Presentation | Covering installing and configuring a UDA server, plugin installation, and basic UDA usage |


---
## Schedule (Day 2)

| Name | Type | Description |
| -----|-------| ------------|
| Mappings | Presentation | ... |
| Mappings | Hands-On | ... |
| MASTU Mapping | Presentation | ... |
| Data Access Plugin | Hands-On | ... |

---
## Workshop Prerequisites

What we assume from attendees:<!-- element style="text-align: left; width: 50%" -->

- Basic knowledge on IMAS and particularly its data model
- Interest in using UDA to map experimental data into IMAS
- Some basic C/C++ & Python knowledge
- Familiarity with Linux systems and command line usage

---
## Workshop Information

- Please ask questions during the presentations &#8212; any that can't be answered quickly can be discussed later or noted for expansion upon in future training materials
- All hands-on work is designed to be done on the provided ITER training VMs
	- The material should work on other machines but may require the installation of some prerequisites and we probably can't support this during the workshop
- Adam & I will be available for on-site questions and will try and answer on-line questions when we can. Stephen will be provided on-line support to provide additional support.
- Please use «live document» for all questions and discussions

---
## Logistics & Administration

- WiFi -> choose "Guest wireless" and then "Login with your ITER account"
- How-to connect to SDCC: https://confluence.iter.org/display/IMP/ITER+Computing+Cluster
- Regular ITER bus lines to Aix (A, B, C) are departing at 17:45 from the parking lot in entrance C
- Work site visit Tuesday afternoon at 15:45
- Workshop dinner at the restaurant L'Orangerie in Hotel Aquabella (Aix-en-Provence) at 19:30