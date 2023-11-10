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

# Introduction
## IMAS UDA workshop, ITER
#### Jonathan Hollocombe, November 2023

---
## Contents

- Motivation
- Background
- Schedule

---
## Motivation

- This workshop is aiming to give you an insight into what UDA is, how it works and how it can be used in IMAS to map experimental data
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
- Previous efforts have used existing work (EXP2ITM) and relied on somewhat shaky infrastructure stacks to enable this
- With release of AL5 and introduction of IMAS URIs UDA can now be used to read IMAS data files remotely
- Building on the improved UDA backend and plugins for AL5 the JSON mapping plugin is designed to provide a better way to write and manage the mappings for multiple machines

---
## Schedule (Day 1)

| Name | Type | Day | Description |
| -----|-------| ----| ------------|
| UDA Overview | Presentation | 1 | Detailing how UDA works to provide context for it's use in IMAS |
| UDA Installation | Hands-On | 1 | Covering installing and configuring a UDA server, plugin installation, and basic UDA usage |
| IMAS UDA | Presentation |  1 | Detailing how UDA is used in IMAS remote data access and experimental data mapping |
| IMAS UDA | Hands-On | 1 | Using IMAS to get data using UDA from IMAS data files and mapped data |


---
## Schedule (Day 2)

| Name | Type | Day | Description |
| -----|-------| ----| ------------|
| Mappings | Presentation | 2 | ... |
| Mappings | Hands-On | 2 | ... |
| MASTU Mapping | Presentation | 2 | ... |
| Data Access Plugin | Hands-On | 2 | ... |

---
## Workshop Prerequisites

What we assume from attendees:<!-- element style="text-align: left; width: 90%" -->

- Interest in using UDA to map experimental data into IMAS
- Some basic C/C++ & Python knowledge
- Familiarity with Linux systems and command line usage

---
## Workshop Information

- Please ask questions during the presentations &#8212; any that can't be answered quickly can be discussed later or noted for expansion upon in training materials
- All hands-on work is designed to be done on the provided ITER training VMs
	- The material should work on other machines but may require the installation of some prerequisites and we probably can't support this during the workshop
- Adam & I will be available for on-site questions and will try and answer on-line questions when we can. Stephen will be provided on-line support to provide additional support.
- Please use «live document» for all questions and discussions

---
## Logistics & Administration

«TODO»

- Details of workshop dinner, timings, buses, site tour, etc.
- Any health & safety info.