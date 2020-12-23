# Thermal Mode Code

## Run Time Table

## makeReactionThermos

以 rhoReactionThermo 为例

```cpp
makeReactionThermos
(
    rhoThermo,
    rhoReactionThermo,
    heRhoThermo,
    homogeneousMixture,
    constTransport,
    sensibleEnthalpy,
    hConstThermo,
    perfectGas,
    specie
);
```

makeReactionThermos 宏展开后的代码

```cpp
#define makeReactionThermo(BaseReactionThermo,CThermo,Mixture,Transport,Type,Thermo,EqnOfState,Specie) \
                                                                               \
    typedefThermoPhysics(Transport,Type,Thermo,EqnOfState,Specie);             \
                                                                               \
    makeThermoPhysicsReactionThermo                                            \
    (                                                                          \
        BaseReactionThermo,                                                    \
        CThermo,                                                               \
        Mixture,                                                               \
        Transport##Type##Thermo##EqnOfState##Specie                            \
    )

```

其中参数列表的对应关系是

| 形参               | 实参               |
| :----------------- | :----------------- |
| BaseThermo         | rhoThermo          |
| BaseReactionThermo | rhoReactionThermo  |
| CThermo            | heRhoThermo        |
| Mixture            | homogeneousMixture |
| Transport          | constTransport     |
| Type               | sensibleEnthalpy   |
| Thermo             | hConstThermo       |
| EqnOfState         | perfectGas         |
| Specie             | specie             |

typedefThermoPhysics 宏继续展开

```cpp
#define typedefThermoPhysics(Transport,Type,Thermo,EqnOfState,Specie)          \
                                                                               \
    typedef                                                                    \
        Transport                                                              \
        <                                                                      \
            species::thermo                                                    \
            <                                                                  \
                Thermo                                                         \
                <                                                              \
                    EqnOfState                                                 \
                    <                                                          \
                        Specie                                                 \
                    >                                                          \
                >,                                                             \
                Type                                                           \
            >                                                                  \
        >                                                                      \
        Transport##Type##Thermo##EqnOfState##Specie
```

在 typedefThermoPhysics 中定义了一个 typedef,将

```cpp
constTransport<species::thermo<hConstThermo<perfectGas<specie>,sensibleEnthalpy>>>
```

定义为别名 constTransportsensibleEnthalpyhConstThermoperfectGasperfectGasspecie

makeReactionThermo 继续调用 makeThermoPhysicsReactionThermos 宏，将 constTransportsensibleEnthalpyhConstThermoperfectGasperfectGasspecie 传给 makeThermoPhysicsReactionThermos 的虚参 ThermoPhys，
makeThermoPhysicsReactionThermos 的代码展开

```cpp
#define makeThermoPhysicsReactionThermos(BaseThermo,BaseReactionThermo,CThermo,Mixture,ThermoPhys) \
                                                                               \
    defineThermoPhysicsReactionThermo                                          \
    (                                                                          \
        BaseReactionThermo,                                                    \
        CThermo,                                                               \
        Mixture,                                                               \
        ThermoPhys                                                             \
    );                                                                         \
                                                                               \
    addThermoPhysicsThermo(basicThermo, CThermo##Mixture##ThermoPhys);         \
    addThermoPhysicsThermo(fluidThermo, CThermo##Mixture##ThermoPhys);         \
    addThermoPhysicsThermo(BaseThermo, CThermo##Mixture##ThermoPhys);          \
    addThermoPhysicsThermo(BaseReactionThermo, CThermo##Mixture##ThermoPhys)
```

defineThermoPhysicsReactionThermo 的形参和实参的对应关系

```cpp
    defineThermoPhysicsReactionThermo
    (
        BaseReactionThermo,//rhoReactionThermo
        CThermo,//heRhoThermo
        Mixture,//homogeneousMixture
        ThermoPhys//constTransport
    );
```

后续的解析参见https://openfoam.top/thermodynamicLIB/#makethermophysicsreactionthermos

## basicThermo

basicThermo 继承 IOdictionary，定义的数据成员为

```cpp
    // Protected data

        //- Phase-name
        const word& phaseName_;


        // Fields

            //- Pressure [Pa]
            volScalarField& p_;

            //- Temperature [K]
            volScalarField T_;

            //- Laminar thermal diffusuvity [kg/m/s]
            volScalarField alpha_;

        //- Should the dpdt term be included in the enthalpy equation
        Switch dpdt_;


```

其中，p*为引用,关于 p*引用的讨论见 UserManual，部分内容为

> In Listing 201, we see that the solver’s reference to the pressure field is provided by the thermophysical model,
> see Line 20 of that listing. As the basicThermo model holds only a reference to the pressure field, in contrast,
> the basicThermo model holds the temperature field as a field; i.e. the basicThermo “owns” the temperature
> field but only refers to the pressure field. Thus, we are left with the puzzle who created the pressure field, since
> the thermophysical model is able to provide the pressure field immediately after its construction, we turn our
> attention to the base classes of the thermophysical models.
> Taking a look at the constructor of the class basicThermo, it appears as if the pressure field needs to be
> already existing, see Line 19 in Listing 203. The method lookupOrConstruct() strongly reminds us of the
> various lookup\*() methods provided by OpenFOAM’s object registry, see Section 57.7. However, this is not
> the case with this class: the method lookupOrConstruct() is provided by the class basicThermo itself, see
> Listing 204. This is actually a pretty elegant way to handle the pressure field.
> This approach allows the designers of OpenFOAM not to care about the pressure field in regards to the
> thermophysical models. E.g. in multiphase solvers, there are two or more thermophysical models constructed,
> one for each phase, and the constructor of the first thermophysical model creates the pressure field, whereas all
> subsequent constructor-calls result in a look-up from the object registry.

除了构造函数，RTS 所需要的宏 TypeName，declareRunTimeSelectionTable 以及工厂的 Selector New 外，在 basicThermo 中还定义了的类方法（有实现）为

```cpp
        //- Check that the thermodynamics package is consistent
        //  with energy forms supported by the application
        void validate
        (
            const string& app,
            const word&
        ) const;

        //- Pressure [Pa]
        //  Non-const access allowed for transport equations
        virtual volScalarField& p();


        //- Temperature [K]
        virtual const volScalarField& T() const;

        //- Thermal diffusivity for enthalpy of mixture [kg/m/s]
        virtual const volScalarField& alpha() const;
```

其中 p()，T()，alpha()直接返回所存储的数据成员

在 basicThermo 中定义了以下的虚方法

```cpp
        //- Update properties
        virtual void correct() = 0;

        //- Return the name of the thermo physics
        virtual word thermoName() const = 0;

        //- Return true if the equation of state is incompressible
        //  i.e. rho != f(p)
        virtual bool incompressible() const = 0;

        //- Return true if the equation of state is isochoric
        //  i.e. rho = const
        virtual bool isochoric() const = 0;

        //- Should the dpdt term be included in the enthalpy equation
        Switch dpdt() const
        {
            return dpdt_;
        }

        //- Density [kg/m^3]
        virtual tmp<volScalarField> rho() const = 0;

        //- Enthalpy/Internal energy [J/kg]
        virtual const volScalarField& he() const = 0;

        //- Chemical enthalpy [J/kg]
        virtual tmp<volScalarField> hc() const = 0;

        //- Heat capacity at constant pressure [J/kg/K]
        virtual tmp<volScalarField> Cp() const = 0;

        //- Heat capacity at constant volume [J/kg/K]
        virtual tmp<volScalarField> Cv() const = 0;

        //- Gamma = Cp/Cv []
        virtual tmp<volScalarField> gamma() const = 0;

        //- Heat capacity at constant pressure/volume [J/kg/K]
        virtual tmp<volScalarField> Cpv() const = 0;


        //- Heat capacity ratio []
        virtual tmp<volScalarField> CpByCpv() const = 0;

        //- Molecular weight [kg/kmol]
        virtual tmp<volScalarField> W() const = 0;

        //- Thermal diffusivity for temperature of mixture [W/m/K]
        virtual tmp<volScalarField> kappa() const = 0;


        //- Thermal diffusivity for energy of mixture [kg/m/s]
        virtual tmp<volScalarField> alphahe() const = 0;

        //- Effective thermal turbulent diffusivity for temperature
        //  of mixture [W/m/K]
        virtual tmp<volScalarField> kappaEff
        (
            const volScalarField&
        ) const = 0;

        //- Effective thermal turbulent diffusivity of mixture [kg/m/s]
        virtual tmp<volScalarField> alphaEff
        (
            const volScalarField& alphat
        ) const = 0;

```

在 basicThermo 中还定义了一些 generic 的静态方法，来 support run time selection

```cpp
    //- Generic lookup for thermodynamics package thermoTypeName
        template<class Thermo, class Table>
        static typename Table::iterator lookupThermo
        (
            const dictionary& thermoTypeDict,
            Table* tablePtr,
            const int nCmpt,
            const char* cmptNames[],
            const word& thermoTypeName
        );

        //- Generic lookup for each of the related thermodynamics packages
        template<class Thermo, class Table>
        static typename Table::iterator lookupThermo
        (
            const dictionary& thermoDict,
            Table* tablePtr
        );

        //- Generic New for each of the related thermodynamics packages
        template<class Thermo>
        static autoPtr<Thermo> New
        (
            const fvMesh&,
            const word& phaseName=word::null
        );

        //- Generic New for each of the related thermodynamics packages
        template<class Thermo>
        static autoPtr<Thermo> New
        (
            const fvMesh&,
            const dictionary&,
            const word& phaseName=word::null
        );

        //- Specialisation of the Generic New for basicThermo
        static autoPtr<basicThermo> New
        (
            const fvMesh&,
            const word& phaseName=word::null
        );
```

并在 basicThermoTemplates 中给出了 generic method 的实现

## fluidThermo

fluidThermo 继承 basicThermo,增加了流体相关的热物性和方法

```cpp
            //- Add the given density correction to the density field.
            //  Used to update the density field following pressure solution
            virtual void correctRho(const volScalarField& deltaRho) = 0;

            //- Compressibility [s^2/m^2]
            virtual const volScalarField& psi() const = 0;

            //- Dynamic viscosity of mixture [kg/m/s]
            virtual tmp<volScalarField> mu() const = 0;

            //- Kinematic viscosity of mixture [m^2/s]
            virtual tmp<volScalarField> nu() const;
```

在实现上，它给出了动力粘性系数的计算

```cpp
Foam::tmp<Foam::volScalarField> Foam::fluidThermo::nu() const
{
    return mu()/rho();
}
```

## rhoThermo

在 rhoThermo 的类内数据中定义了 protected 类型的 fields

```cpp

        //- Density field [kg/m^3]
        //  Named 'rhoThermo' to avoid (potential) conflict with solver density
        volScalarField rho_;

        //- Compressibility [s^2/m^2]
        volScalarField psi_;

        //- Dynamic viscosity [kg/m/s]
        volScalarField mu_;
```

相应地定义了 public 的方法，供 client 调用

```cpp
            //- Return non-const access to the local density field [kg/m^3]
            virtual volScalarField& rho();
            //- Compressibility [s^2/m^2]
            virtual const volScalarField& psi() const;
            //- Dynamic viscosity of mixture [kg/m/s]
            virtual tmp<volScalarField> mu() const;

```

其中 rho()的实现是直接返回类内存储的 filed rho\_,如果需要更新密度，需要调用 correctRho 方法

```cpp

            //- Add the given density correction to the density field.
            //  Used to update the density field following pressure solution
            virtual void correctRho(const volScalarField& deltaRho);
```

correctRho 的实现是

```cpp
void Foam::rhoThermo::correctRho(const Foam::volScalarField& deltaRho)
{
    rho_ += deltaRho;
}

```

## basicMultiComponentMixture

basicMultiComponentMixture 定义了 protected 的数据成员

```cpp

        //- Table of specie names
        speciesTable species_;

        //- List of specie active flags
        List<bool> active_;

        //- Species mass fractions
        PtrList<volScalarField> Y_;
```

定义了以下的方法

```cpp

        //- Return the table of species
        inline const speciesTable& species() const;

        //- Does the mixture include this specie?
        inline bool contains(const word& specieName) const;

        //- Return true for active species
        inline bool active(label speciei) const;

        //- Return the bool list of active species
        inline const List<bool>& active() const;

        //- Set speciei active
        inline void setActive(label speciei);

        //- Set speciei inactive
        inline void setInactive(label speciei);

        //- Return the mass-fraction fields
        inline PtrList<volScalarField>& Y();

        //- Return the mass-fraction field for a specie given by index
        inline volScalarField& Y(const label i);

        //- Return the mass-fraction field for a specie given by name
        inline volScalarField& Y(const word& specieName);


```

## basicSpecieMixture

basicSpecieMixture 继承 basicMultiComponentMixture
定义了许多的虚函数

```cpp

    // Per specie properties

            //- Molecular weight of the given specie [kg/kmol]
            virtual scalar Wi(const label speciei) const = 0;


        // Per specie thermo properties

            //- Heat capacity at constant pressure [J/kg/K]
            virtual scalar Cp
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Heat capacity at constant volume [J/kg/K]
            virtual scalar Cv
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Absolute enthalpy [J/kg]
            virtual scalar Ha
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Sensible enthalpy [J/kg]
            virtual scalar Hs
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Chemical enthalpy [J/kg]
            virtual scalar Hc(const label speciei) const = 0;

            //- Entropy [J/kg/K]
            virtual scalar S
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Sensible internal energy [J/kg]
            virtual scalar Es
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Gibbs free energy [J/kg]
            virtual scalar G
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Helmholtz free energy [J/kg]
            virtual scalar A
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;


        // Per specie transport properties

            //- Dynamic viscosity [kg/m/s]
            virtual scalar mu
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Thermal conductivity [W/m/K]
            virtual scalar kappa
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Thermal diffusivity of enthalpy [kg/m/s]
            virtual scalar alphah
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

            //- Density [kg/m^3]
            virtual scalar rho
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const = 0;

```

## multiComponentMixture

multiComponentMixture 是 generic class， with "template<class ThermoType>".
在 multiComponentMixture 中定义的私有的成员数据有

```cpp
        //- Species data
        PtrList<ThermoType> speciesData_;

        //- Temporary storage for the cell/face mixture thermo data
        mutable ThermoType mixture_;

        //- Temporary storage for the volume weighted
        //  cell/face mixture thermo data
        mutable ThermoType mixtureVol_;
```

其中 mixture*是基于质量分数 Y 平均的混合物的热物性，mixtureVol*是基于体积平均的热物性。

cellMixture 成员方法对 Internal Field 进行质量分数平均计算 mixture\_

```cpp
template<class ThermoType>
const ThermoType& Foam::multiComponentMixture<ThermoType>::cellMixture
(
    const label celli
) const
{
    mixture_ = Y_[0][celli]*speciesData_[0];

    for (label n=1; n<Y_.size(); n++)
    {
        mixture_ += Y_[n][celli]*speciesData_[n];
    }

    return mixture_;
}
```

cellVolMixture 成员方法对 Internal Field 进行体积分数平均计算 mixtureVol\_\_

```cpp
template<class ThermoType>
const ThermoType& Foam::multiComponentMixture<ThermoType>::cellVolMixture
(
    const scalar p,
    const scalar T,
    const label celli
) const
{
    scalar rhoInv = 0.0;
    forAll(speciesData_, i)
    {
        rhoInv += Y_[i][celli]/speciesData_[i].rho(p, T);
    }

    mixtureVol_ =
        Y_[0][celli]/speciesData_[0].rho(p, T)/rhoInv*speciesData_[0];

    for (label n=1; n<Y_.size(); n++)
    {
        mixtureVol_ +=
            Y_[n][celli]/speciesData_[n].rho(p, T)/rhoInv*speciesData_[n];
    }

    return mixtureVol_;
}
```

## reactingMixture

reactingMixture 也是 generic class with “template<class ThermoType>”

继承于 speciesTable，autoPtr<chemistryReader<ThermoType>>，c multiComponentMixture<ThermoType>，PtrList<Reaction<ThermoType>>
定义的数据成员有

```cpp


        //- Table of species composition
        speciesCompositionTable speciesComposition_;

```

定义并实现的内联方法有

```cpp
        //- Read dictionary
        void read(const dictionary&);

        label size() const
        {
            return PtrList<Reaction<ThermoType>>::size();
        }

        Reaction<ThermoType>& operator[](const label i)
        {
            return PtrList<Reaction<ThermoType>>::operator[](i);
        }

        const Reaction<ThermoType>& operator[](const label i) const
        {
            return PtrList<Reaction<ThermoType>>::operator[](i);
        }

        //- Table of species composition
        const speciesCompositionTable& specieComposition() const
        {
            return speciesComposition_;
        }
```

## SpecieMixture

SpecieMixture 是 generic class， with template<class MixtureType>，并继承于 MixtureType，并实现了 MixtureType 中的虚方法

```cpp
            //- Molecular weight of the given specie [kg/kmol]
            virtual scalar Wi(const label speciei) const;


    // Per specie thermo properties

            //- Heat capacity at constant pressure [J/kg/K]
            virtual scalar Cp
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Heat capacity at constant volume [J/kg/K]
            virtual scalar Cv
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Absolute enthalpy [J/kg]
            virtual scalar Ha
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Sensible enthalpy [J/kg]
            virtual scalar Hs
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Chemical enthalpy [J/kg]
            virtual scalar Hc(const label speciei) const;

            //- Entropy [J/kg/K]
            virtual scalar S
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Sensible internal energy [J/kg]
            virtual scalar Es
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Gibbs free energy [J/kg]
            virtual scalar G
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Helmholtz free energy [J/kg]
            virtual scalar A
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;


        // Per specie transport properties

            //- Dynamic viscosity [kg/m/s]
            virtual scalar mu
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Thermal conductivity [W/m/K]
            virtual scalar kappa
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Thermal diffusivity of enthalpy [kg/m/s]
            virtual scalar alphah
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

            //- Density [kg/m^3]
            virtual scalar rho
            (
                const label speciei,
                const scalar p,
                const scalar T
            ) const;

```

## heThermo

heThermo 是 generic class with template "template<class BasicThermo, class MixtureType>", 并继承于 class BasicThermo 和 class MixtureType
BasicThermo 可以是 rhoReactionThermo，
MixtureType 可以是 SpecieMixture<reactingMixture<sutherlandTransport<species::thermo<janafThermo<perfectGas<specie>>,sensibleEnthalpy>>>>

heThermo 的构造函数

```cpp
template<class BasicThermo, class MixtureType>
Foam::heThermo<BasicThermo, MixtureType>::heThermo
(
    const fvMesh& mesh,
    const word& phaseName
)
:
    BasicThermo(mesh, phaseName),
    MixtureType(*this, mesh, phaseName),

    he_
    (
        IOobject
        (
            BasicThermo::phasePropertyName
            (
                MixtureType::thermoType::heName()
            ),
            mesh.time().timeName(),
            mesh,
            IOobject::NO_READ,
            IOobject::NO_WRITE
        ),
        mesh,
        dimEnergy/dimMass,
        this->heBoundaryTypes(),
        this->heBoundaryBaseTypes()
    )
{
    init();
}
```

在构造函数中调用了 init 方法，init 的代码是

```code
template<class BasicThermo, class MixtureType>
void Foam::heThermo<BasicThermo, MixtureType>::init()
{
    scalarField& heCells = he_.primitiveFieldRef();
    const scalarField& pCells = this->p_;
    const scalarField& TCells = this->T_;

    forAll(heCells, celli)
    {
        heCells[celli] =
            this->cellMixture(celli).HE(pCells[celli], TCells[celli]);
    }

    volScalarField::Boundary& heBf = he_.boundaryFieldRef();

    forAll(heBf, patchi)
    {
        heBf[patchi] == he
        (
            this->p_.boundaryField()[patchi],
            this->T_.boundaryField()[patchi],
            patchi
        );
    }

    this->heBoundaryCorrection(he_);
}

```

可见，在 init 中 set 了 he 的 filed，并 implement heBoundaryCorrection

```cpp
template<class BasicThermo, class MixtureType>
void Foam::heThermo<BasicThermo, MixtureType>::
heBoundaryCorrection(volScalarField& h)
{
    volScalarField::Boundary& hBf = h.boundaryFieldRef();

    forAll(hBf, patchi)
    {
        if (isA<gradientEnergyFvPatchScalarField>(hBf[patchi]))
        {
            refCast<gradientEnergyFvPatchScalarField>(hBf[patchi]).gradient()
                = hBf[patchi].fvPatchField::snGrad();
        }
        else if (isA<mixedEnergyFvPatchScalarField>(hBf[patchi]))
        {
            refCast<mixedEnergyFvPatchScalarField>(hBf[patchi]).refGrad()
                = hBf[patchi].fvPatchField::snGrad();
        }
    }
}

```

在 heThermo 中实现了 BasicThermo 中定义的虚方法，如 Cp()，Cp(const scalarField& p,const scalarField& T,const label patchi) const，

方法声明

```cpp

            //- Heat capacity at constant pressure for patch [J/kg/K]
            virtual tmp<scalarField> Cp
            (
                const scalarField& p,
                const scalarField& T,
                const label patchi
            ) const;

            //- Heat capacity at constant pressure [J/kg/K]
            virtual tmp<volScalarField> Cp() const;

```

方法实现

```cpp
template<class BasicThermo, class MixtureType>
Foam::tmp<Foam::scalarField> Foam::heThermo<BasicThermo, MixtureType>::Cp
(
    const scalarField& p,
    const scalarField& T,
    const label patchi
) const
{
    tmp<scalarField> tCp(new scalarField(T.size()));
    scalarField& cp = tCp.ref();

    forAll(T, facei)
    {
        cp[facei] =
            this->patchFaceMixture(patchi, facei).Cp(p[facei], T[facei]);
    }

    return tCp;
}


template<class BasicThermo, class MixtureType>
Foam::tmp<Foam::volScalarField>
Foam::heThermo<BasicThermo, MixtureType>::Cp() const
{
    const fvMesh& mesh = this->T_.mesh();

    tmp<volScalarField> tCp
    (
        volScalarField::New
        (
            "Cp",
            mesh,
            dimEnergy/dimMass/dimTemperature
        )
    );

    volScalarField& cp = tCp.ref();

    forAll(this->T_, celli)
    {
        cp[celli] =
            this->cellMixture(celli).Cp(this->p_[celli], this->T_[celli]);
    }

    volScalarField::Boundary& cpBf = cp.boundaryFieldRef();

    forAll(cpBf, patchi)
    {
        const fvPatchScalarField& pp = this->p_.boundaryField()[patchi];
        const fvPatchScalarField& pT = this->T_.boundaryField()[patchi];
        fvPatchScalarField& pCp = cpBf[patchi];

        forAll(pT, facei)
        {
            pCp[facei] =
                this->patchFaceMixture(patchi, facei).Cp(pp[facei], pT[facei]);
        }
    }

    return tCp;
}


```

## heRhoThermo

heRhoThermo 是 generic class，with template<class BasicPsiThermo, class MixtureType>,继承于 public heThermo<BasicPsiThermo, MixtureType>

heRhoThermo 实现了 basicThermo 中最后一个虚方法 correct()

```cpp
template<class BasicPsiThermo, class MixtureType>
void Foam::heRhoThermo<BasicPsiThermo, MixtureType>::correct()
{
    if (debug)
    {
        InfoInFunction << endl;
    }

    calculate();

    if (debug)
    {
        Info<< "    Finished" << endl;
    }
}

```

correct()调用了 calculate(), calculate 中根据此时已知的 he（解能量方程），p（压力方程），调用 THE 方法计算得到混合物的温度，
并基于质量分数平均计算 psi，rho,mu,alphah

```cpp
template<class BasicPsiThermo, class MixtureType>
void Foam::heRhoThermo<BasicPsiThermo, MixtureType>::calculate()
{
    const scalarField& hCells = this->he();
    const scalarField& pCells = this->p_;

    scalarField& TCells = this->T_.primitiveFieldRef();
    scalarField& psiCells = this->psi_.primitiveFieldRef();
    scalarField& rhoCells = this->rho_.primitiveFieldRef();
    scalarField& muCells = this->mu_.primitiveFieldRef();
    scalarField& alphaCells = this->alpha_.primitiveFieldRef();

    forAll(TCells, celli)
    {
        const typename MixtureType::thermoType& mixture_ =
            this->cellMixture(celli);

        TCells[celli] = mixture_.THE
        (
            hCells[celli],
            pCells[celli],
            TCells[celli]
        );

        psiCells[celli] = mixture_.psi(pCells[celli], TCells[celli]);
        rhoCells[celli] = mixture_.rho(pCells[celli], TCells[celli]);

        muCells[celli] = mixture_.mu(pCells[celli], TCells[celli]);
        alphaCells[celli] = mixture_.alphah(pCells[celli], TCells[celli]);
    }

    volScalarField::Boundary& pBf =
        this->p_.boundaryFieldRef();

    volScalarField::Boundary& TBf =
        this->T_.boundaryFieldRef();

    volScalarField::Boundary& psiBf =
        this->psi_.boundaryFieldRef();

    volScalarField::Boundary& rhoBf =
        this->rho_.boundaryFieldRef();

    volScalarField::Boundary& heBf =
        this->he().boundaryFieldRef();

    volScalarField::Boundary& muBf =
        this->mu_.boundaryFieldRef();

    volScalarField::Boundary& alphaBf =
        this->alpha_.boundaryFieldRef();

    forAll(this->T_.boundaryField(), patchi)
    {
        fvPatchScalarField& pp = pBf[patchi];
        fvPatchScalarField& pT = TBf[patchi];
        fvPatchScalarField& ppsi = psiBf[patchi];
        fvPatchScalarField& prho = rhoBf[patchi];
        fvPatchScalarField& phe = heBf[patchi];
        fvPatchScalarField& pmu = muBf[patchi];
        fvPatchScalarField& palpha = alphaBf[patchi];

        if (pT.fixesValue())
        {
            forAll(pT, facei)
            {
                const typename MixtureType::thermoType& mixture_ =
                    this->patchFaceMixture(patchi, facei);

                phe[facei] = mixture_.HE(pp[facei], pT[facei]);

                ppsi[facei] = mixture_.psi(pp[facei], pT[facei]);
                prho[facei] = mixture_.rho(pp[facei], pT[facei]);
                pmu[facei] = mixture_.mu(pp[facei], pT[facei]);
                palpha[facei] = mixture_.alphah(pp[facei], pT[facei]);
            }
        }
        else
        {
            forAll(pT, facei)
            {
                const typename MixtureType::thermoType& mixture_ =
                    this->patchFaceMixture(patchi, facei);

                pT[facei] = mixture_.THE(phe[facei], pp[facei], pT[facei]);

                ppsi[facei] = mixture_.psi(pp[facei], pT[facei]);
                prho[facei] = mixture_.rho(pp[facei], pT[facei]);
                pmu[facei] = mixture_.mu(pp[facei], pT[facei]);
                palpha[facei] = mixture_.alphah(pp[facei], pT[facei]);
            }
        }
    }
}
```

# thermo
thermo是generic class with template<class Thermo, template<class> class Type>,继承于Thermo，Type<thermo<Thermo, Type>>

有意思的是Type（如sensibleEnthalpy） 中的也是generic class,它的typename，也可以是thermo。

我个人还有疑问：这么设计generic class，generic class中的typename难道不是进入死循环吗？