## SomeExamplesFromDongYue

```
https://www.cfd-china.com/topic/3499/openfoam%E5%B0%8F%E4%BB%A3%E7%A0%81/2
```

## CodeStream to customize the boundary condition

```cpp
    codeInclude
    #{
        #include "fvCFD.H"
    #};

    codeOptions
    #{
        -I$(LIB_SRC)/finiteVolume/lnInclude \
        -I$(LIB_SRC)/meshTools/lnInclude
    #};

    //libs needed to visualize BC in paraview
    codeLibs
    #{
        -lmeshTools \
        -lfiniteVolume
    #};

    code
    #{
        const IOdictionary& d = static_cast<const IOdictionary&>(dict.parent().parent());

        const fvMesh& mesh = refCast<const fvMesh>(d.db());
        const label id = mesh.boundary().findPatchID("velocity-inlet-5");
        const fvPatch& patch = mesh.boundary()[id];

        //vectorField U(mesh.boundary()[id].size(), vector(0, 0, 0));
        vectorField U(patch.size(), vector(0, 0, 0));

        const scalar pi = constant::mathematical::pi;
        const scalar U_0   = 2.;	//max vel
        const scalar p_ctr = 8.;	//patch center
        const scalar p_r   = 8.;	//patch radius

        forAll(U, i)
        {
            const scalar y = patch.Cf()[i][1];
            //U[i] = vector(U_0*sin(pi*y/r), 0., 0.);
            U[i] = vector(U_0*(1-(pow(y - p_ctr,2))/(p_r*p_r)), 0., 0.);
        }

        writeEntry(os, "", U);
    #};
};
```

## CodedFixedValue to hanle time-dependence boundary condition

```cpp
    velocity-inlet-5
    {
   	    type            codedFixedValue;
    	value           uniform (0 0 0);
    	redirectType    parabolicProfile;

        code
            #{
                scalar U_0 = 2, p_ctr = 8, p_r = 8;
                const fvPatch& boundaryPatch = patch();
                const vectorField& Cf = boundaryPatch.Cf();
                vectorField& field = *this;

                scalar t = this->db().time().value();

                forAll(Cf, faceI)
                {
                    field[faceI] = vector(sin(t)*U_0*(1-(pow(Cf[faceI].y()-p_ctr,2))/(p_r*p_r)),0,0);
                }

            #};

        codeOptions
        #{

                -I$(LIB_SRC)/finiteVolume/lnInclude \
                -I$(LIB_SRC)/meshTools/lnInclude

        #};

        codeInclude
        #{
                #include "fvCFD.H"
                #include <cmath>
                #include <iostream>
        #};
    }

```

## CodeStream to init the field

```cpp

internalField   #codeStream
{
    codeInclude
    #{
        #include "fvCFD.H"
    #};

    codeOptions
    #{
        -I$(LIB_SRC)/finiteVolume/lnInclude \
        -I$(LIB_SRC)/meshTools/lnInclude
    #};

    //libs needed to visualize BC in paraview
    codeLibs
    #{
        -lmeshTools \
		-lfiniteVolume
    #};

    code
    #{
        const IOdictionary& d = static_cast<const IOdictionary&>(dict);
        const fvMesh& mesh = refCast<const fvMesh>(d.db());
        vectorField U(mesh.nCells(), vector(0, 0, 0));

        forAll(U, i)
        {
            const scalar x 	= mesh.C()[i][0];
            const scalar a_d 	= 10.;			//incidence angle in deg
            const scalar a_r 	= a_d*3.14159/180.;	//incidence angle in rad
            const scalar U0 	= 1.;			//reference velocity

            U[i] = vector(1., 0., 0.);

            if (x >= 0)
            {
                U[i] = vector(U0*cos(a_r), U0*sin(a_r), 0.);
            }
        }

        writeEntry(os, "", U);
    #};
};
```

## AccessFieldObjectFromTime

```cpp
    const word fieldName {"T"};
    const fvMesh& mesh=time_.lookupObject<fvMesh>(polyMesh::defaultRegion);
    bool isFoundObject=mesh.foundObject<volScalarField>(fieldName_);
    if(isFoundObject)
    {
        const volScalarField& field=mesh.lookupObject<volScalarField>(fieldName_);
        Info<<"Temperature at cell 274: "<<field[274]<<endl;
    }
    else
    {
        FatalErrorInFunction<<"cannot found field "<<fieldName_<<exit(FatalError);
    }

```

## ReadPropertiesFromIODictionary

```cpp
    Info<<"Reading sampling properties"<<endl;
    Foam::IOdictionary logProperties
    (
        Foam::IOobject
        (
            "samplingProperties",
            runTime.system(),
            mesh,
            Foam::IOobject::MUST_READ_IF_MODIFIED,
            Foam::IOobject::NO_WRITE
        )
    );

    const word fieldName {logProperties.lookup("field")};
    const label cellIndex=Foam::readLabel(logProperties.lookup("cellIndex"));

```

## Update the field, velocity or scalarField

```cpp
    Info<< "    Reading U" << endl;
    volVectorField U
    (
        IOobject
        (
            "U",
            runTime.timeName(),
            mesh,
            IOobject::MUST_READ,
            IOobject::NO_WRITE
        ),
        mesh
    );



    // Do cells
    const volVectorField& centres = mesh.C();

    const point origin(1, 1, 0.05);
    const vector axis(0, 0, -1);

    Info<<"change the velocity primitive field"<<endl;
    U.primitiveFieldRef()=axis^(centres.primitiveField()-origin);

    forAll(U.boundaryField(),patchI)
    {
        fvPatchField<Vector<scalar>>& patchField=U.boundaryFieldRef()[patchI];
        if(patchField.type()!="empty")
        {
            Info<<"change the velocity patch field at patch index: "<<patchI<<" , with patch type"<<patchField.type()<<endl;
            patchField=(axis ^ (centres.boundaryField()[patchI] - origin));
        }
    }

    U.write();

    volScalarField source
    (
        IOobject
        (
            "source",
            runTime.timeName(),
            mesh,
            IOobject::MUST_READ,
            IOobject::NO_WRITE
        ),
        mesh
    );

    Info<<"change the source primitive field"<<endl;
    source.primitiveFieldRef()=centres.primitiveField().component(vector::X);

    forAll(source.boundaryField(),patchI)
    {
        fvPatchField<scalar>& patchField=source.boundaryFieldRef()[patchI];
        if(patchField.type()!="empty")
        {
            Info<<"change the source patch field at patch index: "<<patchI<<" , with patch type"<<patchField.type()<<endl;
            patchField==centres.boundaryField()[patchI].component(vector::X);
        }
    }
    source.write();


```
