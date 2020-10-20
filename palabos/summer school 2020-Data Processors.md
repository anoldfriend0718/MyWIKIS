![](images/summer%20school%202020-Data%20Processors_2020-10-18-22-42-25.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-22-45-33.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-22-46-27.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-22-50-19.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-22-55-08.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-22-56-30.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-22-57-56.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-01-26.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-03-35.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-08-01.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-12-17.png)

static variables: population and additional data, such as external force, stored in the cell

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-13-25.png)
If we need to recreate the dynamics object, we should use the data structure type

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-20-46.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-23-15.png)

Even though this operation is local, we cannot use the dyanmics object for this implementation, since it involves the coupling between two block-lattices

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-32-18.png)

From the presentor explained, this functional is only applied to the bulk

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-44-06.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-45-05.png)
For the applyProcessingFunctional interface, we need to give the full domain which the processing functional want to apply to. The palabos is then responsible to dive the full domain into the sub domains. Of course, Palabos also automatically implement the parallelism.

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-51-40.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-18-23-53-04.png)
The internal data processor can be automatically implemented after a collide-stream cycles. Additionally, the internal data processor introduce a concept of group, specified by the argument, level, which carry out the MPI communication based on the group, not each internal data processor. It can significantly reduce the expensive MPI communication

![](images/summer%20school%202020-Data%20Processors_2020-10-19-00-04-31.png)

BoxScalarSumFunction3D helps implement the MPI reduction inside

![](images/summer%20school%202020-Data%20Processors_2020-10-19-00-06-19.png)

![](images/summer%20school%202020-Data%20Processors_2020-10-19-00-13-23.png)

