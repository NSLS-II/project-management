## J-PLS requests that require hkl
J-PLS requires HKL to run a custom diffractometer. They run a unique setup, as they do diffraction from liquids. This makes a special case whose geometry is not yet included in the base `hkl` package:
- [ ] Create the geometry file for use with the base `hkl` package (they have the existing geometry code used with 'specs' but we would need to convert this to work with 'hkl'.
- [ ] Create an ophyd object for the diffractometer, which includes a reference to the sample lattice parameters.
    - see this link to the CSX definition as an example. They use the E6C class with a 4 circle difractomer and set the unavailable rotation axes to '0'.
    - setting the sample lattice will be done via (or similar)
    ```
    # import the lattice object
    from hkl.util import Lattice
    # define the lattice lengths are in Angstrom, angles are in degrees
    lattice = Lattice(a=9.069, b=9.069, c=10.390, alpha=90.0, beta=90.0, gamma=120.0)
    # add the sample to the calculation engine
    hkl_object.calc.new_sample('some_sample_name', lattice=lattice)
    ```
### Requirements for diffractometer hkl_object

- [ ] Be able to find the current location in reciprocal-space (hkl) and real-space values this can be done by the 2 commands:
    ```
    hkl_object.position  # reciprocal space position in hkl co-ordinates.
    hkl_object.real_postiion  # real-space position.
    ```
- [ ] Be able to calculate real-space values given the reciprocal-space (hkl) values and vice-versa. This can be done using the built-in hkl methods:
    ```
    hkl_object.calc.wavelength = plancks_constant*speed_of_light/ photon_energy
    hkl_object.calc.forward((h,k,l))  # real-space positions based on inputted h,k,l co-ordinates
    hkl_object.calc.reverse(('mu',  'omega',  'chi',  'phi', 'gamma', 'delta'))  # hkl values from angles
    ```

    - Note 1: We may want to write a custom version of the reverse calculation that is a partialof this for the 5 axis diffractometer. along the lines of the following:

    ```
    def hkl_object(hkl.diffract.E6C):

        hkl_object.calc.reverse(value):  # where value = ('mu',  'omega',  'chi',  'phi', 'gamma')
            value.append(0)  # adds a zero for the 'delta' angle
            return super().calc.reverse(value)
    ```
   - Note 2: A similiar version of the above should probably be implemented for the forward calculation that limits the return value to only the known angles.

- [ ] Be able to move the diffractometer to a given (hkl) position. This can be done via the following:
    ```
    hkl_object.move(h,k,l)  # move to these 'h,k,l' co-ordinates
    ```
    - Note 1: Although they have not specified it, we should look into how to 'scan' in reciprocal space. This has been requested for ISR, and we should look to implement these items in a consistent manner across all beamlines.
    - Note 2: In addition to scanning in 'reciprocal space' a request for 'energy' scanning with 'fixed Q' has also been made (by ISR).
        -This would involve fixed h,k,l and therefore non- fixed real-space locations (as the h,k,l calculation relies on photon energy).
