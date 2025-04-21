VBSP (Source SDK 2013 Edition)
==============================

This reference is based on code in the Source SDK 2013 repo as of commit `0565403b153dfcde602f6f58d8f4d13483696a13` (Tuesday 1st April 2025).

## VBSP Compile Process Overview

This is for a standard, full compile: no `-onlyents` or simimlar options. The most important steps are in bold.

1. Initialise required systems (eg. FS, materials, ...)
2. Load config (eg. command line options, surface properties)
3. **Load map source file and parse into pre-BSP items (entities, containing brushes, containing faces, etc.)**
4. Post-process items and prep for BSP tree construction
5. **Construct BSP tree: world brushes, plus any brush-based entities**
6. Add arbitrary binary content, if provided, to embedded ZIP/PAK section of file
7. Write BSP file
8. Shut all systems down and exit

Step 3 can be considered the **CSG** (Constructive Solid Geometry) step, which used to be its own executable for GoldSrc/Quake. Step 5 can be considered the **BSP** (Binary Space Partition) step.

### CSG Step

CSG converts the map from its source file (MAP or VMF) into the required in-memory structures. For Source, the basic hierarchy is:

* Entities: at least one (the world), but almost certainly more. For each entity:
	* Keyvalues, representing the entity's properties.
	* Zero or more brushes. For each brush:
		* Attributes, eg. contents, bounds, etc.
		* Sides that form the brush's volume. For each side:
			* Attributes, eg. contents, visibility, etc.
			* Winding describing the vertices.
			* Displacement info, if applicable.

### BSP Step (World)

This step is run for entity 0 (the world).

BSP divides the geometry of the map up according to the planes that are used to form the brushes. The BSP process is split into "blocks" of 1024 units on X and Y, corresponding to the 1024-unit gridlines in Hammer. These blocks do not partition the space on Z, only on X and Y.

The BSP process is:

* For each block:
	* Iterate over all available brushes and clip them to the bounds of the block. This gives a list of brushes that are contained within the block.
	* If there were no brushes in the block, make a BSP tree leaf node marked as `SOLID` (ie. it's part of the surrounding map void).
	* Fix araeaportals in water.
	* Carve up intersecting brushes if required.
	* **Build a BSP tree** for the brushes in this block.
	* Store a pointer to the tree for this block.
* Once all blocks are processed, construct a root BSP tree that encompasses all the blocks.

The step for **building a BSP tree** (`BuildTree_r()`) is where the meat of the algorithm is. The algorithm proceeds recursively.

1. Find the best split plane for the current group of brushes.
If there is no best plane, return a leaf node containing the group of brushes.
2. Otherwise, split the group of brushes into two new groups, according to the chosen split plane.
3. Split the node volume into two based on the split plane.
4. Call the function recursively for each child of the split node.

After the core BSP algorithm has been run, some subsequent steps are performed:
* Create all "portals" (ie. planes which connect BSP leaves) for the BSP tree.
* Perform flood-fill checks for each entity in the map, to see if there are any leaks. Set any nodes that were not reached as solid.
* Mark visible brush sides. These are sides that were actually used in the BSP process.

This entire process is run twice. The second time, only brush faces which were marked as visible will be used.

After the BSP process has been run twice, the following post-processing steps are performed:

* Flood-fill to find leaves bounded by areaportals. This finds the unique areas available in the map.
* Remove remaining areaportal brushes.
* Make visible faces from portals.
* Assign areas to occluders.
* Compute areas which comprise the 3D skybox.
* Merge detail brushes with main BSP tree (may need investigating and documenting).
* Fix T-junctions (may need investigating and documenting).
* Prune solid nodes: for any nodes where both childs of the node are solid, turn that node into a solid leaf instead.

### BSP Step (Submodels)

This step is run for brush-based entities. Is is similar to the world BSP process, and uses some of the same functions.

* Create a BSP brush list for the entity.
* Chop up intersecting brushes into convex pieces, if required.
* Create a BSP tree from the brushes.
* Create portals for the BSP tree.
* Mark visible brush sides.
* Make faces.
* Fix T-junctions.
