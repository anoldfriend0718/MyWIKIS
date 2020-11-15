# Finite volume options

## explanations

- https://openfoam.org/release/2-2-0/fv-options/
- https://www.openfoam.com/documentation/guides/latest/doc/guide-fvoptions.html

## Run-time Selectable Physics

A new framework has been introduced to allow users to select any physics that can be represented as sources or constraints on the governing equations, e.g. porous media, MRF and body forces. This new fvOptions framework enhances and supercedes the previous run-time selectable sources in version 2.1.

## Find Examples

- cmd: `foamInfo fvOptions`
- Heat Exchanger – example of interRegionExplicitPorositySource, interRegionHeatTransferModel and MRFSource
  \$FOAM_TUTORIALS/heatTransfer/chtMultiRegionSimpleFoam/heatExchanger
- Filter – example of semiImplicitSource and explicitPorositySource
  \$FOAM_TUTORIALS/lagrangian/reactingParcelFoam/filter
- Angled Duct – example of explicitPorositySource
  \$FOAM_TUTORIALS/compressible/rhoPimpleFoam/ras/angledDuct
- 2D Mixer Vessel – example of MRFSource
  \$FOAM_TUTORIALS/incompressible/simpleFoam/mixerVessel2D
- Coal Chemistry – example of fixedTemperatureConstraint
  \$FOAM_TUTORIALS/lagrangian/coalChemistryFoam/simplifiedSiwek

- Usage example
- scalarExplicitSetValue: https://www.cfd-online.com/Forums/openfoam-programming-development/121464-building-solver-fixedtemperatureconstraint-using-fvoptions.html

```OpenFOAM
source1
{
    type            scalarExplicitSetValue;
    active          true;
    selectionMode   cellZone;
    cellZone         fluid-porous;

    scalarExplicitSetValueCoeffs
    {
        volumeMode      absolute;
        injectionRate
        {
            T              323;
        }
    }
}
```

- meanVelocityForce or patchMeanVelocityForce: https://caefn.com/openfoam/fvoptions-meanvelocityforce#mjx-eqn-eq%3AmagUbarAve

```OpenFOAM
/*--------------------------------*- C++ -*----------------------------------*\
| =========                 |                                                 |
| \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox           |
|  \\    /   O peration     | Version:  dev                                   |
|   \\  /    A nd           | Web:      www.OpenFOAM.org                      |
|    \\/     M anipulation  |                                                 |
\*---------------------------------------------------------------------------*/
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    location    "constant";
    object      fvOptions;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

momentumSource
{
    type            meanVelocityForce;
    active          yes;

    meanVelocityForceCoeffs
    {
        selectionMode   all;

        fields          (U);
        Ubar            (0.1335 0 0);
        relaxation      1.0;
    }
}
```

- explicitPorositySource:
  https://bugs.openfoam.org/view.php?id=1904

  https://openfoam.org/release/2-2-0/fv-options/

```OpenFOAM

porosity1
{
    type          explicitPorositySource;
    active        yes;
    selectionMode cellZone;
    cellZone      porosity;

    explicitPorositySourceCoeffs
    {
        type DarcyForchheimer;

        DarcyForchheimerCoeffs
        {
              d    d [0 -2 0 0 0 0 0] (5e7 -1000 -1000);
              f    f [0 -1 0 0 0 0 0] (0 0 0);

              coordinateSystem
              {
                  e1    (0.70710678 0.70710678 0);
                  e2    (0 0 1);
              }
        }
    }
}

```
