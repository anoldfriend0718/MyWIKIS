1. reference OpenFoam reference codes
https://github.com/precice/openfoam-adapter/pull/97/files


2. [comments from MakisH](https://precice.discourse.group/t/can-precice-be-used-for-volume-coupling/27)

   Yes, but it will be computationally expensive. preCICE is mainly designed to couple simulations that share a common surface boundary. In this case, all the coupled volume nodes should be specified in the coupling mesh.

preCICE only knows about points and their connectivity, so it doesn’t make much of a difference if the points are on a surface or on a volume. However, in the case of volume coupling, nearest-projection mapping would not be available. You may still use the nearest-neighbor or RBF mapping, however.
