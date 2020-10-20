![](images/software%20architecture_2020-10-13-23-20-03.png)

![](images/software%20architecture_2020-10-13-23-24-13.png)
![](images/software%20architecture_2020-10-13-23-33-56.png)
![](images/software%20architecture_2020-10-13-23-37-10.png)
![](images/software%20architecture_2020-10-13-23-40-06.png)
![](images/software%20architecture_2020-10-13-23-40-55.png)
![](images/software%20architecture_2020-10-13-23-41-09.png)
![](images/software%20architecture_2020-10-13-23-41-38.png)
![](images/software%20architecture_2020-10-13-23-43-29.png)
![](images/software%20architecture_2020-10-13-23-45-02.png)
![](images/software%20architecture_2020-10-13-23-46-01.png)
![](images/software%20architecture_2020-10-13-23-47-40.png)
Notes: the block-lattice not only store the static lattice population, the memory size of which depend on the lattice descriptor, but also some extra scalar variable, such as the external force in each cell, leading to more complicated data structure than the tensor field. 
![](images/software%20architecture_2020-10-13-23-49-12.png)

Notes: the block (cell) object is static, which is only allocated once, while the dynamic object, which is used to implement the collision algorithm, can be changed from time to time based on the specific requirement.

![](images/software%20architecture_2020-10-13-23-57-40.png)
Notes: Internally, the Palabos actually create patches with small pieces of blocks, which cover useful the fluid sub-domains and selectively de-allocate unnecessary parts,like the black domain in this slice, so that the memory is only allocated on the certain multi-blocks.  
The Multi-Block is like a matrix, but not a simple matrix. It allows for memory saving through the sparse memory implementations, and parallel program executions based on a data-parallel model

![](images/software%20architecture_2020-10-14-00-05-08.png)
Notes: The multiblocks create the subdomains to cover the computational domain

![](images/software%20architecture_2020-10-14-00-07-09.png)
Notes:  Except the traditional CFD, the realization of the mesh refinement donot come from the fundamental theory, but from the software level, which create several levels with different resolution mesh and then glue them together with proper numerical methods to copy data from one mesh to another mesh. This kind of method is referred to as the Multi-grid. Palabos simply generate seperate multi-blocks with sparse memory implementation, each of which holds the data at a certain level of grid refinement. Palabos gathers the several multi-blocks together into a single data structure, referred to as multi-grid, which is accessed from the user-end. 

![](images/software%20architecture_2020-10-14-00-08-49.png)
![](images/software%20architecture_2020-10-14-00-22-51.png)
![](images/software%20architecture_2020-10-14-00-25-14.png)
![](images/software%20architecture_2020-10-14-00-27-01.png)
![](images/software%20architecture_2020-10-14-00-28-55.png)

![](images/software%20architecture_2020-10-14-09-08-19.png)
![](images/software%20architecture_2020-10-14-09-13-40.png)
Notes: we can use the defineDynamics with one argument as the function object,which define the boolean operator to determine the dynamics object of which cell should be redefined, to update the dynamics object in the specified domain

![](images/software%20architecture_2020-10-14-09-15-53.png)
The programming interface of Palabos includes the discrete velocity descriptor, the dynamics object for the local collision, the data processor for other non-local algorithms, such BC, or interaction, or coupling, et al.. The data processor also can be parallelled among the full domain

![](images/software%20architecture_2020-10-14-09-18-48.png)
![](images/software%20architecture_2020-10-14-09-32-37.png)
![](images/software%20architecture_2020-10-14-09-38-11.png)
![](images/software%20architecture_2020-10-14-09-44-30.png)
Notes: the principle of unit conversions is the match of governing dimensionless numbers

![](images/software%20architecture_2020-10-14-09-48-44.png)
Notes: We should pay attention to the computation order when making the unit conversion. The velocity in the lattice unit, that is the Mach number, should be much lower than 1 to ensure the flow in the incompressible regime, therefore, we need to use the fine grid or the small time step to guarantee the Mach number constraint.
