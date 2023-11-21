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
# UDA Roadmap
#### _<u>Jonathan Hollocombe</u>, Adam Parker, Stephen Dixon_ <br> November 22, 2023 &#8212; ITER

---
## Quick recap

![[IMAS tech stack.svg|1200]]

---
## UDA roadmap

<grid rotate="90" drop="left"  drag="30 100" class="no-border">
![[chevrons.svg]]
</grid>

- **Release 2.8.0**
	- Windows CI build
	- Unification with ITER CODAC UDA
- **Release 2.8.$x$**
	- Any updates and bugfixes required for outstanding issues (SSL, etc.)
- **Release 3.0**
	- Tidying up UDA header files
	- Making `IDAM_PLUGIN_INTERFACE` structure opaque
		- With more plugin helper functions
	- Rewritten C++ client
		- Should hopefully fix issues around multiple clients open in Python etc.


---
## UDA roadmap (cont.)

<grid drop="left"  drag="20 100" class="no-border" style="opacity: 50%;">
![[UK_traffic_sign_7001.svg]]
</grid>

- **Release TBD**
	- Moving from xinetd to systemd
	- Adding ability to run server outside of xinetd/systemd
	- Additional data access plugins in the core repo (MDS+, etc.)
	- Always more documentation!
- **Additional developments?**
	- Async data requests
	- Move from XDR serialisation to something else

---
## IMAS UDA backend roadmap

<grid rotate="90" drop="left"  drag="30 100" class="no-border">
![[chevrons.svg]]
</grid>

- **Release 5.x**
	- ...

---
## IMAS plugin roadmap

<grid rotate="90" drop="left"  drag="30 100" class="no-border">
![[chevrons.svg]]
</grid>

- **Release 1.4.0**
	- Tagged release for changes and fixes from workshop preparation
- **Release 1.4.$x$**
	- Any changes & bugfixes required for outstanding issues
	- Updates from outcome of performance studies
- **Release TBD**


---
## JSON mapping roadmap

<grid rotate="90" drop="left"  drag="30 100" class="no-border">
![[chevrons.svg]]
</grid>

- ...

---
## Mappings roadmap

<grid rotate="90" drop="left"  drag="30 100" class="no-border">
![[chevrons.svg]]
</grid>

- ...

---