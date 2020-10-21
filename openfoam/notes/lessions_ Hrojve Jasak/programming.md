# programming in openFOAM

## slices

![](img/programming_2020-09-01-23-55-39.png)

### OpenFOAM Work Spaace

![](img/programming_2020-09-03-10-54-50.png)

- Run directory(\$FOAM_RUN): ready-to0run cases and results, test loop etc. May contain case-specific setup tools, solvers and utilities

![](img/programming_2020-09-03-11-42-57.png)

### build

![](img/programming_2020-09-03-11-06-25.png)

![wmake files](img/programming_2020-09-03-11-27-10.png)

![wmake cmd](img/programming_2020-09-03-11-27-57.png)

![wmake file example for EXE](img/programming_2020-09-03-11-45-28.png)

![wmake option example for EXE](img/programming_2020-09-03-11-51-06.png)

![wmake file example for LIB](img/programming_2020-09-03-12-07-13.png)

![wamke option example for LIB](img/programming_2020-09-03-13-50-19.png)

- wmake support multiple platform
- only recompile the changes
- make exe: wmake
- make lib: wmake libso

### Creating your applications

![](img/programming_2020-09-03-11-17-13.png)

![](img/programming_2020-09-03-11-17-52.png)

- runtime selection table (动态绑定，多态 by interface0
- post-process by function object in controldict
- edit the make file (such as the target dir)

### Programming guidances

![](img/programming_2020-09-03-11-33-11.png)

- implement new code with class, not function

![](img/programming_2020-09-03-11-37-50.png)

### Debugging

![](img/programming_2020-09-03-11-41-42.png)
