Customisable BSP Compiler
=========================

A BSP compiler (and other stages such as VIS and RAD) with modular stages that can be aggregated via shared libraries, so that it can be used to support multiple games.

Aspects could include:

* **Command-line parameters**
	* Parameters for a specific run, eg. `-onlyents`, top-level BSP tree block size, etc.
* **Program config**
	* Dynamically enable and disable functionality, such as whether the user is allowed to run certain sub-stages, and what command-line parameters are supported.
	* Provide per-game parameters, eg. max supported brushes.
	* Specify which entities and entity properties map to specific compiler features, eg. the classname for a light, the property keys that control light parameters, texture paths that refer to hint surfaces, etc.
* **Map source parsers**
	* For reading .map files, .vmf files, etc. and producing geometry collections from them.
* **Algorithm hooks**
	* Eg. choosing the best split plane for a group of brushes.
* **Custom stages**
	* Performing extra processing on geometry before a subsequent stage.
	* Performing additional, arbitrary computation steps, the results of which may or may not form part of the final map file, eg. collecting stats or finding other resources on disk.
* **Map exporters**
	* For writing the compiled data to a game-ready map format.
	* Related: **Lump Generators** for specifying which lumps an output map file should have, and taking compiled data and storing it in these lumps in a known format.
