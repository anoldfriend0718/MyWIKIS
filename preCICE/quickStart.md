1. Features

Fully parallel peer-to-peer concept: Coupled solvers directly communicate with each other without requiring a central instance. All coupling operations are executed directly on the solvers’ compute resources. This enables massively parallel simulations without the coupling being the bottleneck of the overall simulation.

Pure library approach: In contrast to a framework approach, the solvers call preCICE instead of being called by the framework. This makes coupling minimally-invasive and thus easy to set up and to maintain. The preCICE API operates on a generic level, allowing highest flexibility and the implementation of new adapters in as little as approximately 30 lines of code.

Sophisticated and robust quasi-Newton coupling algorithms: They enable the partitioned realization of strongly-coupled problems, such as those observed in hemodynamic applications. In some communities, quasi-Newton coupling is also known as Anderson acceleration.

Multi coupling: preCICE allows the robust coupling of an arbitrary number of solvers to one overall simulation.

Introduction video :
https://www.youtube.com/channel/UCxZdSQdmDrheEqxq8g48t6A
https://www.youtube.com/watch?v=INGsFlCW3B8

2. [training slides](https://github.com/MakisH/ofw15-slides)
3. Tutorial for CHT-> Flow over a heated plate:

   https://github.com/precice/openfoam-adapter/wiki/

   https://github.com/precice/openfoam-adapter/tree/master/tutorials/CHT/flow-over-plate/buoyantPimpleFoam-laplacianFoam

4. volume coupling :
   https://www.youtube.com/watch?v=TrZD9SnG2Ts
5. [preCICE homepage](https://www.precice.org/)
6. [preCICE repo](https://github.com/precice)
7. [preCICE WIKI](https://github.com/precice/precice/wiki)
7. [openfoam adaptor](https://github.com/precice/openfoam-adapter)

8. [generator-propagation example](https://github.com/MakisH/ofw15-slides/tree/master/generator-propagator/solution)

Generator

```python
import numpy as np
import time
import precice


n = 20
dn = 1 / n

# generate mesh
y = np.linspace(0, 1, n + 1)

# preCICE setup
participant_name = "Generator"
config_file_name = "precice-config.xml"
solver_process_index = 0
solver_process_size = 1
interface = precice.Interface(participant_name, config_file_name, solver_process_index, solver_process_size)

mesh_name = "Generator-Mesh"
mesh_id = interface.get_mesh_id(mesh_name)

data_name = "Data"
data_id = interface.get_data_id(data_name, mesh_id)

vertices = [[1, y0] for y0 in y[:-1]]

vertex_ids = interface.set_mesh_vertices(mesh_id, vertices)

precice_dt = interface.initialize()

dt = 0.01
t = 0

while interface.is_coupling_ongoing():

  print("Generating data")
  dt = np.minimum(dt, precice_dt)
  time.sleep(0.2)
  u = 1 - 2 * np.random.rand(n)

  interface.write_block_scalar_data(data_id, vertex_ids, u)

  precice_dt = interface.advance(dt)

  t = t + dt

interface.finalize()
```

Propagator

```python

import numpy as np
import matplotlib.pyplot as plt
import precice


n = 20
dn = 1 / n

# generate mesh
x = np.linspace(0, 1, n+1)
y = np.linspace(0, 1, n+1)

# initial data, associated to cell centers
u = np.zeros([n, n])

# preCICE setup
participant_name = "Propagator"
config_file_name = "precice-config.xml"
solver_process_index = 0
solver_process_size = 1
interface = precice.Interface(participant_name, config_file_name, solver_process_index, solver_process_size)

mesh_name = "Propagator-Mesh"
mesh_id = interface.get_mesh_id(mesh_name)

data_name = "Data"
data_id = interface.get_data_id(data_name, mesh_id)

vertices = [[1, y0] for y0 in y[:-1]]

vertex_ids = interface.set_mesh_vertices(mesh_id, vertices)

precice_dt = interface.initialize()

# plot initial data
fig = plt.figure()
ax = fig.add_subplot(111)
X, Y = np.meshgrid(x, y)
c = ax.pcolor(X, Y, u, vmin=-1, vmax=1, edgecolors='k', linewidths=1, cmap='RdBu')
fig.colorbar(c, ax=ax)
plt.axvspan(0.99, 1.0, color='grey', alpha=0.8)
plt.xlabel("x (Interface at x=1)")
plt.ylabel("y")
plt.ion()
plt.show()

t = 0

while interface.is_coupling_ongoing():

  dt = 0.0025

  print("Propagating data")
  dt = np.minimum(dt, precice_dt)

  u[:,-1] = interface.read_block_scalar_data(data_id, vertex_ids)

  # some (arbitrary) convection-diffusion rule
  # top row
  u[0, :-1] = 0.3 * u[0, :-1] + 0.6 * u[0, 1:] + 0.1 * u[1, :-1]
  # inner domain
  u[1:-1, :-1] = 0.2 * u[1:-1, :-1] + 0.6 * u[1:-1, 1:] + 0.1 * u[2:, :-1] + 0.1 * u[:-2, :-1]
  # bottom row
  u[-1, :-1] = 0.3 * u[-1, :-1] + 0.6 * u[-1, 1:] + 0.1 * u[-1, :-1]

  # plot current result
  plt.pause(0.2)
  ax.pcolor(X, Y, u, vmin=-1, vmax=1, edgecolors='k', linewidths=1, cmap='RdBu')

  precice_dt = interface.advance(dt)

  # advance
  t = t + dt

interface.finalize()
```

precice-config.xml

```xml
<?xml version="1.0"?>

<precice-configuration>

  <log>
    <sink filter= "%Severity% > debug" format="---[precice] %ColorizedSeverity% %Message%" enabled="true" />
  </log>

  <solver-interface dimensions="2">

  <data:scalar name="Data"/>

  <mesh name="Generator-Mesh">
    <use-data name="Data"/>
  </mesh>

  <mesh name="Propagator-Mesh">
    <use-data name="Data" />
  </mesh>

  <participant name="Generator">
    <use-mesh name="Generator-Mesh" provide="yes"/>
    <write-data name="Data" mesh="Generator-Mesh"/>
  </participant>

  <participant name="Propagator">
    <use-mesh name="Generator-Mesh" from="Generator"/>
    <use-mesh name="Propagator-Mesh" provide="yes"/>
    <mapping:nearest-neighbor direction="read"  from="Generator-Mesh" to="Propagator-Mesh" constraint="consistent"/>
    <read-data name="Data" mesh="Propagator-Mesh" />
  </participant>

  <m2n:sockets from="Generator" to="Propagator"/>

  <coupling-scheme:serial-explicit>
    <participants first="Generator" second="Propagator"/>
    <time-window-size value="0.01"/>
    <max-time value="0.1"/>
    <exchange data="Data" mesh="Generator-Mesh" from="Generator" to="Propagator"/>
  </coupling-scheme:serial-explicit>

  </solver-interface>

</precice-configuration>
```


