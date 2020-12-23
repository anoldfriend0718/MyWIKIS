# Thermal Model

## Notes online

- One should **use rhoThermo for fluids with incompressible or low compressibility** like water where the density change is mostly due to temperature variation. The **psiThermo model should be used for fully compressible gas**.

```
No guaranty but I'd say that the choice of the model depends on your application. Buoyant solvers, manly changing their density due to temperature changes, use the rhoThermo model. They use a simplified pressure equation where psi is accounted for explicitly:

Code:
   fvc::ddt(rho) + psi*correction(fvm::ddt(p_rgh))
 + fvc::div(phiHbyA)
 - fvm::laplacian(rAUf, p_rgh)
==
   fvOptions(psi, p_rgh, rho.name())
So huge pressure jumps will not be represented correctly. I'd use such a solver for fluids with low compressibility and low pressure differences.
In contrast, some combustion solvers and trans sonic solvers use the psiThermo model. They have another pressure equation where the change in time of pressure is treated implicit by ddt(psi, p):

Code:
   fvm::ddt(psi, p)
 + fvc::div(phiHbyA)
 - fvm::laplacian(rhorAUf, p)
==
   fvOptions(psi, p, rho.name())
Larger pressure jumps are now possible. Such solvers are used when the flow is mainly driven by pressure changes and temperature change is only an effect of large compression and expansion.

So the comment of chriss85 #3 is not entirely wrong. But still no guaranty that all my statements are correct.

Cheers

Fabian

```

- First, I think the term "compressible" has to be defined carefully here. (I don't have experience on supersonic flows or high Ma flows, so what I have in mind may be wrong) Compressible flows, or "fully" compressible flows, suggest that the density should change with respect to pressure (and temperature, mixture componenets, etc.). However, in many heat transfer and combustion problems, the density mainly changes with temperature and mixture components, but not with pressure. It is still a compressible flow in the sense that the density does vary(and sometimes it varies a lot, one or two orders of magnitude difference), but not so in the sense that the density does NOT vary with pressure changes. To deal with the latter case people have developed the so-called low Mach number formulation to solve the whole system of equations.

- thermo.rho() returns a stored rho field that is calculated from pressure and
  temperature fields according to the selected thermophysical model in thermo.correct().

- heRhoThermo variations

```
heRhoThermo  homogeneousMixture         const       hConst       incompressiblePerfectGas  specie  sensibleEnthalpy

heRhoThermo  homogeneousMixture         const       hConst       perfectGas                specie  sensibleEnthalpy
heRhoThermo  homogeneousMixture         sutherland  janaf        incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  homogeneousMixture         sutherland  janaf        perfectGas                specie  sensibleEnthalpy
heRhoThermo  inhomogeneousMixture       const       hConst       incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  inhomogeneousMixture       const       hConst       perfectGas                specie  sensibleEnthalpy
heRhoThermo  inhomogeneousMixture       sutherland  janaf        incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  inhomogeneousMixture       sutherland  janaf        perfectGas                specie  sensibleEnthalpy
heRhoThermo  multiComponentMixture      const       eConst       adiabaticPerfectFluid     specie  sensibleInternalEnergy
heRhoThermo  multiComponentMixture      const       eConst       incompressiblePerfectGas  specie  sensibleInternalEnergy
heRhoThermo  multiComponentMixture      const       eConst       perfectFluid              specie  sensibleInternalEnergy
heRhoThermo  multiComponentMixture      const       eConst       perfectGas                specie  sensibleInternalEnergy
heRhoThermo  multiComponentMixture      const       eConst       rhoConst                  specie  sensibleInternalEnergy
heRhoThermo  multiComponentMixture      const       hConst       adiabaticPerfectFluid     specie  sensibleEnthalpy
heRhoThermo  multiComponentMixture      const       hConst       incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  multiComponentMixture      const       hConst       perfectFluid              specie  sensibleEnthalpy
heRhoThermo  multiComponentMixture      const       hConst       perfectGas                specie  sensibleEnthalpy
heRhoThermo  multiComponentMixture      const       hConst       rhoConst                  specie  sensibleEnthalpy
heRhoThermo  multiComponentMixture      polynomial  hPolynomial  icoPolynomial             specie  sensibleEnthalpy
heRhoThermo  multiComponentMixture      polynomial  hPolynomial  icoPolynomial             specie  sensibleInternalEnergy
heRhoThermo  multiComponentMixture      sutherland  janaf        incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  multiComponentMixture      sutherland  janaf        incompressiblePerfectGas  specie  sensibleInternalEnergy
heRhoThermo  multiComponentMixture      sutherland  janaf        perfectGas                specie  sensibleEnthalpy
heRhoThermo  multiComponentMixture      sutherland  janaf        perfectGas                specie  sensibleInternalEnergy
heRhoThermo  pureMixture                WLF         eConst       rhoConst                  specie  sensibleInternalEnergy
**heRhoThermo  pureMixture                const       eConst       Boussinesq                specie  sensibleInternalEnergy**
heRhoThermo  pureMixture                const       eConst       adiabaticPerfectFluid     specie  sensibleInternalEnergy
heRhoThermo  pureMixture                const       eConst       perfectFluid              specie  sensibleInternalEnergy
heRhoThermo  pureMixture                const       eConst       rhoConst                  specie  sensibleInternalEnergy
heRhoThermo  pureMixture                const       hConst       Boussinesq                specie  sensibleEnthalpy
heRhoThermo  pureMixture                const       hConst       Boussinesq                specie  sensibleInternalEnergy
heRhoThermo  pureMixture                const       hConst       adiabaticPerfectFluid     specie  sensibleEnthalpy
heRhoThermo  pureMixture                const       hConst       adiabaticPerfectFluid     specie  sensibleInternalEnergy
heRhoThermo  pureMixture                const       hConst       incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  pureMixture                const       hConst       incompressiblePerfectGas  specie  sensibleInternalEnergy
heRhoThermo  pureMixture                const       hConst       perfectFluid              specie  sensibleEnthalpy
heRhoThermo  pureMixture                const       hConst       perfectFluid              specie  sensibleInternalEnergy
heRhoThermo  pureMixture                const       hConst       perfectGas                specie  sensibleEnthalpy
heRhoThermo  pureMixture                const       hConst       perfectGas                specie  sensibleInternalEnergy
heRhoThermo  pureMixture                const       hConst       rhoConst                  specie  sensibleEnthalpy
heRhoThermo  pureMixture                const       hConst       rhoConst                  specie  sensibleInternalEnergy
heRhoThermo  pureMixture                polynomial  hPolynomial  PengRobinsonGas           specie  sensibleEnthalpy
heRhoThermo  pureMixture                polynomial  hPolynomial  icoPolynomial             specie  sensibleEnthalpy
heRhoThermo  pureMixture                polynomial  hPolynomial  icoPolynomial             specie  sensibleInternalEnergy
heRhoThermo  pureMixture                polynomial  janaf        PengRobinsonGas           specie  sensibleEnthalpy
heRhoThermo  pureMixture                sutherland  hConst       Boussinesq                specie  sensibleEnthalpy
heRhoThermo  pureMixture                sutherland  hConst       Boussinesq                specie  sensibleInternalEnergy
heRhoThermo  pureMixture                sutherland  hConst       PengRobinsonGas           specie  sensibleEnthalpy
heRhoThermo  pureMixture                sutherland  hConst       incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  pureMixture                sutherland  hConst       incompressiblePerfectGas  specie  sensibleInternalEnergy
heRhoThermo  pureMixture                sutherland  hConst       perfectGas                specie  sensibleEnthalpy
heRhoThermo  pureMixture                sutherland  hConst       perfectGas                specie  sensibleInternalEnergy
heRhoThermo  pureMixture                sutherland  janaf        Boussinesq                specie  sensibleEnthalpy
heRhoThermo  pureMixture                sutherland  janaf        Boussinesq                specie  sensibleInternalEnergy
heRhoThermo  pureMixture                sutherland  janaf        incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  pureMixture                sutherland  janaf        incompressiblePerfectGas  specie  sensibleInternalEnergy
heRhoThermo  pureMixture                sutherland  janaf        perfectGas                specie  sensibleEnthalpy
heRhoThermo  pureMixture                sutherland  janaf        perfectGas                specie  sensibleInternalEnergy
heRhoThermo  reactingMixture            const       eConst       adiabaticPerfectFluid     specie  sensibleInternalEnergy
heRhoThermo  reactingMixture            const       eConst       incompressiblePerfectGas  specie  sensibleInternalEnergy
heRhoThermo  reactingMixture            const       eConst       perfectFluid              specie  sensibleInternalEnergy
heRhoThermo  reactingMixture            const       eConst       perfectGas                specie  sensibleInternalEnergy
heRhoThermo  reactingMixture            const       eConst       rhoConst                  specie  sensibleInternalEnergy
heRhoThermo  reactingMixture            const       hConst       adiabaticPerfectFluid     specie  sensibleEnthalpy
heRhoThermo  reactingMixture            const       hConst       incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  reactingMixture            const       hConst       perfectFluid              specie  sensibleEnthalpy
heRhoThermo  reactingMixture            const       hConst       perfectGas                specie  sensibleEnthalpy
heRhoThermo  reactingMixture            const       hConst       rhoConst                  specie  sensibleEnthalpy
heRhoThermo  reactingMixture            polynomial  hPolynomial  icoPolynomial             specie  sensibleEnthalpy
heRhoThermo  reactingMixture            polynomial  hPolynomial  icoPolynomial             specie  sensibleInternalEnergy
heRhoThermo  reactingMixture            sutherland  janaf        incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  reactingMixture            sutherland  janaf        incompressiblePerfectGas  specie  sensibleInternalEnergy
heRhoThermo  reactingMixture            sutherland  janaf        perfectGas                specie  sensibleEnthalpy
**heRhoThermo  reactingMixture            sutherland  janaf        perfectGas                specie  sensibleInternalEnergy**
**heRhoThermo  singleStepReactingMixture  sutherland  janaf        perfectGas                specie  sensibleEnthalpy**
heRhoThermo  singleStepReactingMixture  sutherland  janaf        perfectGas                specie  sensibleInternalEnergy
heRhoThermo  veryInhomogeneousMixture   const       hConst       incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  veryInhomogeneousMixture   const       hConst       perfectGas                specie  sensibleEnthalpy
heRhoThermo  veryInhomogeneousMixture   sutherland  janaf        incompressiblePerfectGas  specie  sensibleEnthalpy
heRhoThermo  veryInhomogeneousMixture   sutherland  janaf        perfectGas                specie  sensibleEnthalpy
```

- hePsiThermo variations

```
type            mixture                    transport   thermo       equationOfState           specie  energy


hePsiThermo     homogeneousMixture         const       hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     homogeneousMixture         sutherland  hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     homogeneousMixture         sutherland  janaf        perfectGas                specie  sensibleEnthalpy
hePsiThermo     inhomogeneousMixture       const       hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     inhomogeneousMixture       sutherland  hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     inhomogeneousMixture       sutherland  janaf        perfectGas                specie  sensibleEnthalpy
hePsiThermo     multiComponentMixture      const       eConst       perfectGas                specie  sensibleInternalEnergy
hePsiThermo     multiComponentMixture      const       hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     multiComponentMixture      sutherland  janaf        perfectGas                specie  sensibleEnthalpy
hePsiThermo     multiComponentMixture      sutherland  janaf        perfectGas                specie  sensibleInternalEnergy
hePsiThermo     pureMixture                const       eConst       perfectGas                specie  sensibleInternalEnergy
hePsiThermo     pureMixture                const       hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     pureMixture                const       hConst       perfectGas                specie  sensibleInternalEnergy
hePsiThermo     pureMixture                polynomial  hPolynomial  PengRobinsonGas           specie  sensibleEnthalpy
hePsiThermo     pureMixture                polynomial  janaf        PengRobinsonGas           specie  sensibleEnthalpy
hePsiThermo     pureMixture                sutherland  eConst       perfectGas                specie  sensibleInternalEnergy
hePsiThermo     pureMixture                sutherland  hConst       PengRobinsonGas           specie  sensibleEnthalpy
hePsiThermo     pureMixture                sutherland  hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     pureMixture                sutherland  hConst       perfectGas                specie  sensibleInternalEnergy
hePsiThermo     pureMixture                sutherland  janaf        PengRobinsonGas           specie  sensibleEnthalpy
hePsiThermo     pureMixture                sutherland  janaf        PengRobinsonGas           specie  sensibleInternalEnergy
hePsiThermo     pureMixture                sutherland  janaf        perfectGas                specie  sensibleEnthalpy
hePsiThermo     pureMixture                sutherland  janaf        perfectGas                specie  sensibleInternalEnergy
hePsiThermo     reactingMixture            const       eConst       perfectGas                specie  sensibleInternalEnergy
hePsiThermo     reactingMixture            const       hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     reactingMixture            sutherland  janaf        perfectGas                specie  sensibleEnthalpy
hePsiThermo     reactingMixture            sutherland  janaf        perfectGas                specie  sensibleInternalEnergy
hePsiThermo     singleStepReactingMixture  sutherland  janaf        perfectGas                specie  sensibleEnthalpy
hePsiThermo     singleStepReactingMixture  sutherland  janaf        perfectGas                specie  sensibleInternalEnergy
hePsiThermo     veryInhomogeneousMixture   const       hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     veryInhomogeneousMixture   sutherland  hConst       perfectGas                specie  sensibleEnthalpy
hePsiThermo     veryInhomogeneousMixture   sutherland  janaf        perfectGas                specie  sensibleEnthalpy
```

- thermofile example

```OpenFOAM
thermoType
{
    type            hePsiThermo;
    mixture         pureMixture;
    transport       polynomial;
    thermo          hPolynomial;
    equationOfState PengRobinsonGas;
    specie          specie;
    energy          sensibleEnthalpy;
}

mixture
{

     specie//AirValues
    {
        molWeight   28.9;
    }
    thermodynamics
    {
    	Hf              0;
        Sf              0;
        CpCoeffs<8>     ( 1029.48769276813  -0.259754651949017  0.000720524392649807 -4.32731178093884e-7 8.18443559767102e-11 0 0 0); // [J/kg/K]
    }

    transport
    {
        muCoeffs<8>     ( 6.33240897464201e-6 4.34314235629553e-8 -7.67992288001185e-12 0 0 0 0 0 );   //- Dynamic viscosity [kg/m/s]
        kappaCoeffs<8>  ( 0.00422477982917543 7.58163166906076e-5 -1.29122249700985e-8 0 0 0 0 0 ); //- Thermal conductivity [W/m/K]
    }

    equationOfState
	{
  		Tc  132.5; //- Critical Temperature [K]
  		Vc  8.45251377759746E-2; //- Critical volume [m^3/kmol]
  		Pc  3.7860E6;//- Critical Pressure [Pa]
  		omega  0.036;//- Acentric factor [-] http://www.coolprop.org/fluid_properties/fluids/Air.html blended from oxygen and nitrogen
	}

}
```

- For mixtures with variable composition, required by thermophysical models with reactions, the reactingMixture option is used. Species and reactions are listed in a chemistry file, specified by the foamChemistryFile keyword. The reactingMixture model then requires the thermophysical models coefficients to be specified for each specie within sub-dictionaries named after each specie, e.g. O2, N2.

```thermophysicalProperties
thermoType
{
    type            hePsiThermo;
    mixture         reactingMixture;
    transport       sutherland;
    thermo          janaf;
    energy          sensibleEnthalpy;
    equationOfState perfectGas;
    specie          specie;
}

inertSpecie N2;

chemistryReader foamChemistryReader;

foamChemistryFile "$FOAM_CASE/constant/reactions";

foamChemistryThermoFile "$FOAM_CASE/constant/thermo.compressibleGas";

```

```reactions
species
(
    O2
    H2O
    CH4
    CO2
    N2
);

reactions
{
    methaneReaction
    {
        type     irreversibleArrheniusReaction;
        reaction "CH4 + 2O2 = CO2 + 2H2O";
        A        5.2e16;
        beta     0;
        Ta       14906;
    }
}
```

```thermo.compressibleGas

O2
{
    specie
    {
        molWeight       31.9988;
    }
    thermodynamics
    {
        Tlow            200;
        Thigh           5000;
        Tcommon         1000;
        highCpCoeffs    ( 3.69758 0.00061352 -1.25884e-07 1.77528e-11 -1.13644e-15 -1233.93 3.18917 );
        lowCpCoeffs     ( 3.21294 0.00112749 -5.75615e-07 1.31388e-09 -8.76855e-13 -1005.25 6.03474 );
    }
    transport
    {
        As              1.67212e-06;
        Ts              170.672;
    }
}



```
