# Notes on some treatments in OpenFoam

## ddtPhiCorr

### codes for Euler Ddt Scheme

- EulerDdtScheme<Type>::fvcDdtUfCorr in EulerDdtScheme.C

```cpp
    //EulerDdtScheme<Type>::fvcDdtUfCorr (U, Uf), similar to other overloads
    dimensionedScalar rDeltaT = 1.0/mesh().time().deltaT();

    fluxFieldType phiUf0(mesh().Sf() & Uf.oldTime());
    fluxFieldType phiCorr
    (
        phiUf0 - fvc::dotInterpolate(mesh().Sf(), U.oldTime())
    );

    return tmp<fluxFieldType>
    (
        new fluxFieldType
        (
            IOobject
            (
                "ddtCorr(" + U.name() + ',' + Uf.name() + ')',
                mesh().time().timeName(),
                mesh()
            ),
            this->fvcDdtPhiCoeff(U.oldTime(), phiUf0, phiCorr)
           *rDeltaT*phiCorr
        )
    );
```

- fvcDdtPhiCoeff in ddtScheme.C

```cpp
    tmp<surfaceScalarField> tddtCouplingCoeff = scalar(1)
      - min
        (
            mag(phiCorr)
           /(mag(phi) + dimensionedScalar("small", phi.dimensions(), SMALL)),
            scalar(1)
        );

```

### Correction algorithm in IcoFoam

```cpp
    surfaceScalarField phiHbyA
    (
        "phiHbyA",
        fvc::flux(HbyA)
        + fvc::interpolate(rAU)*fvc::ddtCorr(U, phi)
    );
```

- step1: In ddtPhiCorr(...), call appropriate version of fvcDdtPhiCorr
- step2: In function fvcDdtPhiCorr, Values of 1/A, delta T are given. Additionally, the phi difference, which compares the interface velocity and the flux at the last time step, is given by
  $$
  \phi^{d}=\left(\phi^{\text {old }}-\left(\mathbf{U}^{\text {old }} \cdot \mathbf{S}\right)\right)
  $$
- step3: In function fvcDdtPhiCoeff, the correction coefficient is given by

$$
K_{c}=1-\min \left[\frac{\left|\phi^{d}\right|}{\left|\phi^{\text {old }}\right|+\epsilon_{\text {s }}}, 1\right]
$$

- step4: By `this->fvcDdtPhiCoeff(U.oldTime(), phiUf0, phiCorr)*rDeltaT*phiCorr`, the actual phi correction is

  $$
  \phi^{c}=\phi^{d} K_{c} \frac{1}{A} \frac{1}{\Delta t}
  $$

### Why we need ddtPhiCorr

Discretization of incompressible momentum equation:

$$
\frac{\mathbf{U}_{\mathrm{P}}^{*}-\mathbf{U}_{\mathrm{P}}^{t}}{\Delta t} V_{\mathrm{P}}+\sum F_{f}^{n} \frac{\mathrm{U}_{\mathrm{P}}^{*}+\mathrm{U}_{\mathrm{N}}^{*}}{2}=\sum \nu\left|\mathbf{S}_{f}\right| \frac{\mathrm{U}_{\mathrm{N}}^{*}-\mathrm{U}_{\mathrm{P}}^{*}}{|\mathbf{d}|}-\sum \frac{p_{\mathrm{P}}^{t}+p_{\mathrm{N}}^{t}}{2} \mathbf{S}_{f}
$$

Simplify it, we can get

$$
A_{\mathrm{P}} \mathbf{U}_{\mathrm{P}}^{*}+\sum A_{\mathrm{N}} \mathbf{U}_{\mathrm{N}}^{*}=S_{\mathrm{P}}^{t}-\frac{1}{V_{\mathrm{P}}} \sum \frac{p_{\mathrm{P}}^{t}+p_{\mathrm{N}}^{t}}{2} \mathbf{S}_{f}
$$

where

$$
\begin{array}{c}
A_{\mathrm{P}}=\frac{1}{\Delta t}+\frac{1}{V_{\mathrm{P}}} \sum \frac{F_{f}^{t}}{2}+\frac{1}{V_{\mathrm{P}}} \sum \nu \frac{\left|\mathbf{S}_{f}\right|}{|\mathbf{d}|} \\
A_{\mathrm{N}}=\frac{1}{V_{\mathrm{P}}}\left(\frac{F_{f}^{t}}{2}-\nu \frac{\left|\mathbf{S}_{f}\right|}{|\mathbf{d}|}\right) \\
S_{\mathrm{P}}^{t}=\frac{1}{\Delta t} \mathbf{U}_{\mathrm{P}}^{t}
\end{array}
$$

Combine terms, we can get

$$
\begin{array}{c}
\mathbf{U}_{\mathbf{P}}^{* *}=\mathbf{H} \mathbf{b} \mathbf{y} \mathbf{A}_{\mathbf{P}}^{*}-\frac{1}{A_{\mathrm{P}}} \frac{1}{V_{\mathrm{P}}} \sum p_{f}^{*} \mathbf{S}_{f} \\
\mathbf{H} \mathbf{b} \mathbf{y} \mathbf{A}_{\mathrm{P}}^{*}=\frac{1}{A_{\mathrm{P}}}\left(-\sum A_{\mathrm{N}} \mathbf{U}_{\mathrm{N}}^{*}+S_{\mathrm{P}}^{n}\right) \\
\mathbf{U}_{\mathbf{P}, f}^{* *}=\mathbf{H} \mathbf{b} \mathbf{y} \mathbf{A}_{f}^{*}-\frac{1}{A_{\mathrm{P}, f}}\left(\frac{1}{V_{\mathrm{P}}} \sum p_{f}^{*} \mathbf{S}_{f}\right)_{f}
\end{array}
$$

The ddtPhiCorr correction is applied to $\mathbf{H} \mathbf{b} \mathbf{y} \mathbf{A}_{\mathrm{P}}^{*}$, more specially, on the $S_{\mathrm{P}}^{t}$ in $\mathbf{H} \mathbf{b} \mathbf{y} \mathbf{A}_{\mathrm{P}}^{*}$ to correct the flux term at the old time step

$$

\frac{1}{A_{\mathrm{P},f}}(S_{\mathrm{P},f}^{t}\cdot \mathbf{S}+\phi^{d}_{\mathrm{P}} K_{c,{\mathrm{P}}}\frac{1}{\Delta t})=
\frac{1}{A_{\mathrm{P}}}\frac{1}{\Delta t}(\mathbf{U}_{\mathrm{P,f}}^{t}\cdot \mathbf{S}+(\phi^{\mathrm{t }}_{\mathrm{P,f}}-K_{c,{\mathrm{P}}}\left(\mathbf{U}^{\mathrm {t }}_{\mathrm{P,f}} \cdot \mathbf{S}\right))
$$

## constrainPressure for fixedFluxPressure boundary condition

### Code

- constrainPressure method constrainPressure.C

```cpp
   const fvMesh& mesh = p.mesh();

    volScalarField::Boundary& pBf = p.boundaryFieldRef();

    const volVectorField::Boundary& UBf = U.boundaryField();
    const surfaceScalarField::Boundary& phiHbyABf =
        phiHbyA.boundaryField();
    const typename RAUType::Boundary& rhorAUBf =
        rhorAU.boundaryField();
    const surfaceVectorField::Boundary& SfBf =
        mesh.Sf().boundaryField();
    const surfaceScalarField::Boundary& magSfBf =
        mesh.magSf().boundaryField();

    forAll(pBf, patchi)
    {
        if (isA<fixedFluxPressureFvPatchScalarField>(pBf[patchi]))
        {
            refCast<fixedFluxPressureFvPatchScalarField>
            (
                pBf[patchi]
            ).updateCoeffs
            (
                (
                    phiHbyABf[patchi]
                  - rho.boundaryField()[patchi]
                   *MRF.relative(SfBf[patchi] & UBf[patchi], patchi)
                )
               /(magSfBf[patchi]*rhorAUBf[patchi])
            );
        }
    }
```

- updateCoeffs in fixedFluxPressureFvPatchScalarField.C

```cpp
void Foam::fixedFluxPressureFvPatchScalarField::updateCoeffs
(
    const scalarField& snGradp
)
{
    if (updated()) //return  switch "updated_"
    {
        return;
    }

    curTimeIndex_ = this->db().time().timeIndex();

    gradient() = snGradp;
    //set the updated_ as true
    fixedGradientFvPatchScalarField::updateCoeffs();
}
```

### Algorithm

**By constrainPressure, this fixed flux pressure boundary condition adjusts the pressure gradient such that the flux on the boundary is that specified by the velocity boundary condition.**

First of all we need to write down the momentum equation, implemented by buoyantPimpleFoam, discretized using the Rhie-Chow interpolation method.

$$
\begin{array}{l}
\mathbf{p h i}=\mathbf{p h i H b y} \mathbf{A}-\left(\frac{\rho^{*}}{A_{\mathrm{p}}} \nabla p_{r g h}^{* *}\right)_{f} \cdot \mathbf{S}_{f}
\end{array}
$$

where

$$
\text { phiHbyA }=\left(\rho_{f}^{*} \mathbf{H} \mathbf{b y} \mathbf{A}_{f}^{*}-\frac{\rho_{f}^{*}}{A_{\mathrm{P}, f}} \mathbf{g} \mathbf{h} \nabla \rho_{f}^{*}\right) \cdot \mathbf{S}_{f}
$$

Thus the following equation for the pressure surface gradient can be obtained

$$
\begin{array}{l}
\left(\frac{\rho^{*}}{A_{\mathrm{p}}} \nabla p_{r g h}^{* *}\right)_{f} \cdot \mathbf{S}_{f}=\mathbf{p h i H b y} \mathbf{A}-\mathbf{p h i} \\
\left(\frac{\rho^{*}}{A_{\mathrm{p}}} \nabla p_{r g h}^{* *}\right)_{f} \cdot n_{f}\left\|\mathbf{S}_{f}\right\|=\mathbf{p h i H b y} \mathbf{A}-\mathbf{p h i} \\
\left(\nabla p_{r g h}^{* *}\right)_{f}=\frac{(\mathbf{p} \mathbf{h} \mathbf{i} \mathbf{H} \mathbf{b} \mathbf{y}-\mathbf{p} \mathbf{h} \mathbf{i})}{\left\|\mathbf{S}_{f}\right\|\left(\frac{\rho^{*}}{A_{\mathrm{p}}}\right)_{f}}
\end{array}
$$

## AdjustPhi

### code in icoFoam

- adjust phiHbyA in icoFoam

```cpp
    volScalarField rAU(1.0/UEqn.A());
    volVectorField HbyA(constrainHbyA(rAU*UEqn.H(), U, p));
    surfaceScalarField phiHbyA
    (
        "phiHbyA",
        fvc::flux(HbyA)
        + fvc::interpolate(rAU)*fvc::ddtCorr(U, phi)
    );

    adjustPhi(phiHbyA, U, p);
```


- adjustPhi method in adjustPhi.C

```cpp

 if (p.needReference()) // if p need reference,  all Pressure B.C. are Neumann type. Therefore, the adjustPhi only applied if all Pressure B.C. are Neumann type
    {
        scalar massIn = 0.0;
        scalar fixedMassOut = 0.0;
        scalar adjustableMassOut = 0.0;

        surfaceScalarField::Boundary& bphi =
            phi.boundaryFieldRef();

        forAll(bphi, patchi)
        {
            const fvPatchVectorField& Up = U.boundaryField()[patchi];
            const fvsPatchScalarField& phip = bphi[patchi];

            if (!phip.coupled())
            {
                if (Up.fixesValue() && !isA<inletOutletFvPatchVectorField>(Up))
                {
                    forAll(phip, i)
                    {
                        if (phip[i] < 0.0)
                        {
                            massIn -= phip[i];
                        }
                        else
                        {
                            fixedMassOut += phip[i];
                        }
                    }
                }
                else
                {
                    forAll(phip, i)
                    {
                        if (phip[i] < 0.0)
                        {
                            massIn -= phip[i];
                        }
                        else
                        {
                            adjustableMassOut += phip[i];
                        }
                    }
                }
            }
        }

        // Calculate the total flux in the domain, used for normalisation
        scalar totalFlux = vSmall + sum(mag(phi)).value();

        reduce(massIn, sumOp<scalar>());
        reduce(fixedMassOut, sumOp<scalar>());
        reduce(adjustableMassOut, sumOp<scalar>());

        scalar massCorr = 1.0;
        scalar magAdjustableMassOut = mag(adjustableMassOut);

        if
        (
            magAdjustableMassOut > vSmall
         && magAdjustableMassOut/totalFlux > small
        )
        {
            massCorr = (massIn - fixedMassOut)/adjustableMassOut;
        }
        // If the fixedMassOut is larger than massIn, the adjustPhi cannot work
        else if (mag(fixedMassOut - massIn)/totalFlux > 1e-8)
        {
            FatalErrorInFunction
                << "Continuity error cannot be removed by adjusting the"
                   " outflow.\nPlease check the velocity boundary conditions"
                   " and/or run potentialFoam to initialise the outflow." << nl
                << "Total flux              : " << totalFlux << nl
                << "Specified mass inflow   : " << massIn << nl
                << "Specified mass outflow  : " << fixedMassOut << nl
                << "Adjustable mass outflow : " << adjustableMassOut << nl
                << exit(FatalError);
        }

        forAll(bphi, patchi)
        {
            const fvPatchVectorField& Up = U.boundaryField()[patchi];
            fvsPatchScalarField& phip = bphi[patchi];

            if (!phip.coupled())
            {
                //only apply to the inletOutletFvPatchField
                if
                (
                    !Up.fixesValue()
                 || isA<inletOutletFvPatchVectorField>(Up)
                )
                {
                    forAll(phip, i)
                    {
                        if (phip[i] > 0.0)
                        {
                            phip[i] *= massCorr;
                        }
                    }
                }
            }
        }

        return mag(massIn)/totalFlux < small
            && mag(fixedMassOut)/totalFlux < small
            && mag(adjustableMassOut)/totalFlux < small;
    }
    else
    {
        return false;
    }
```

- Algorithm of adjustPhi

If all the pressure B.C. are Neumann type, adjustPhi looks at all the boundary patches of the mesh, adding the mass flux into 3 type: massIn, fixedMassOut and adjustableMassOut. If adjustableMassOut is larger than 0, then distribute adjustableMassOut to the adjustable velocity boundary (derived from inletOutletFvPatchVectorField) to ensure the mass balance.

- Why we need adjustPhi

Based on panelV.Vuorinen's [study](https://www.sciencedirect.com/science/article/pii/S0045793014000334),
the Helmholtz–Hodge (HH) decomposition forms the mathematical basis for coupling velocity and pressure in incompressible flows. In fluid dynamics, the HH-decomposition states that any velocity field may be expressed as a sum two parts: a divergence free and a curl free part [1]. In the HH-decomposition, a pressure p is required such that:

$$
\mathbf{u}=\mathbf{u}^{*}-\nabla p
$$

In Eq. above, $\nabla \cdot \mathbf{u}^{*} \neq 0$, $\nabla \cdot \mathbf{u}=0$ and $\nabla \times \nabla p=0$ since the gradient of any scalar field is curl-free. Taking divergence of Eq. shows that p must be a solution of the Poisson equation

$$
\Delta p=\nabla \cdot \mathbf{u}^{*}
$$

Poisson’s equation with all Neumann boundary conditions must satisfy a [compatibility condition](http://web.stanford.edu/class/cs205b/lectures/lecture16.pdf) for a solution to exist. The problem is given by

$$
\left\{\begin{aligned}
\Delta p=\nabla \cdot \mathbf{u}^{*} & \text { in } \Omega \\
\nabla p \cdot \mathbf{n}=g & \text { on } \partial \Omega
\end{aligned}\right.
$$

where n is the unit normal to the boundary. From the equation we have the relations

$$
\int_{\Omega} \nabla \cdot \mathbf{u}^{*} d V=\int_{\Omega} \Delta p d V=\int_{\Omega} \nabla \cdot \nabla p d V=\int_{\partial \Omega} \nabla p \cdot \mathbf{n} d S=\int_{\partial \Omega} g d S
$$

where the third equality follows from the divergence theorem. The compatibility condition is

$$
\int_{\Omega} \nabla \cdot \mathbf{u}^{*} d V=\int_{\partial \Omega} g d S
$$

The right hand side $g=0$, Therefore, the compatibility condition is

$$
\int_{\Omega} \nabla \cdot \mathbf{u}^{\star} d V=\int_{\partial \Omega} \mathbf{u}^{\star} \cdot \mathbf{n} d S=0
$$

where the first equality follows from the divergence theorem. This condition needs to be satisfied
when specifying the boundary condition on $\mathbf{u}^{\star}$ in order to guarantee the existence of a solution.

Taking icoFoam as example,

$$
\begin{array}{c}
\mathbf{U}_{\mathbf{P}}^{* *}=\mathbf{H} \mathbf{b} \mathbf{y} \mathbf{A}_{\mathbf{P}}^{*}-\frac{1}{A_{\mathrm{P}}} \frac{1}{V_{\mathrm{P}}} \sum p_{f}^{*} \mathbf{S}_{f}
\end{array}
$$

Therefore, we get

$$
\mathbf{u}^{\star}=\text {HbyA}
$$

$$
\int_{\Omega} \nabla \cdot \mathbf{u}^{\star} d V=\int_{\partial \Omega} \text {HbyA} \cdot \mathbf{n} d S=0
$$

AdjustPhi make $
\int_{\partial \Omega} \text {HbyA} \cdot \mathbf{n} d S=0
$ when all the Pressure B.C. are Neumann
