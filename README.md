MoRiBS-PIMC
===========

Molecular Rotor in Bosonic Solvents, a Path-Integral Monte Carlo program

  MoRiBS-PIMC: Molecular Rotators in Bosonic Solvents with Path-Integral Monte Carlo

-----------------------------------------------------------------------------------

	* Disclaimer: We disclaim any and all warranties concerning the enclosed program. *

The program MoRiBS-PIMC is mainly written by Tao (Toby) Zeng, Nicholas Blinov, Gregoire Guillon, Hui Li, and Pierre-Nicholas Roy at University of Alberta and University of Waterloo, Canada. If you encounter any problem with respect to compiling or running the program, please contact T. Zeng by email (tzeng@ualberta.ca or t2zeng@uwaterloo.ca).

					************************
					**    COMPILATION     **
					************************

Users should not change the directory structure after they unzip the distributed file. We label the main directory, where the source codes (*.cc, *.h, and *.f) are, as $MAIN. Please note that we have provided example configuration files like makefile with the distribution and one may just adjust the files according to the architecture of his/her computer. There is no need to create new configuration files. The compilation procedure of MoRiBS-PIMC contains the following steps:

	1: `cd $MAIN/spring` and open make.CHOICES. One should specify the platform of his/her computer by uncommenting the correct line, i.e., PLAT = LINUX;
	2: `cd $MAIN/spring/SRC` and open make.${PLAT}. With the above choice, it should be make.LINUX. In make.${PLAT}, one should specify the fortran and C/C++ compilers that are installed in his/her computer;
	3: at $MAIN/spring/SRC, `make clean` and `make`. The generated libraries are in $MAIN/spring/lib. Steps 1-3 are to compile the sprng libraries that are needed by the main code;
	4: at $MAIN, open makefile, and specify options, CFLAGS, LDFLAGS, CC and FC. Those are the optimization options, C/C++ compilation flags, link flags, C/C++ compiler, and Fortran compiler respectively;
	5: at $MAIN, `make clean` and `make`. If there is no error message, the generated executable is called pimc.

					************************
					**   OUTPUT SUMMARY   **
					************************

Are are two relevant directories for a MoRiBS-PIMC run, the working directory and the output directory. The former is the directory where the program executable is and the latter is specified in qmc.input. In below we simply call the two directories $WORK and $OUTPUT. And we also use $FILNAM to specify the FILENAMEPREFIX that is specified in qmc.input.

************
   energy
************
All energy outputs are stored in the following two files:

	$OUTPUT/$FILNAM_sum.eng and $OUTPUT/$FILNAM.eng.

The first contains the energy components averaged on the fly while the latter contains the block-averaged quantities for every block. In $OUTPUT/$FILNAM_sum.eng, column
	1: counting indices of punching out the energy components;
	2: translational kinetic energy;
	3: potential energy;
	4: the sum of translational and potential energies;
	5: rotational kinetic energy;
	6: heat capacity term A (CvA) for debugging;
	7: total energy (translational + potential + rotational);
	8: total heat capacity;
	9: heat capacity term B (CvB) for debugging;
	10: heat capacity term C (CvC) for debugging.

All energy quantities have the unit of Kelvin (K) and the heat capacity has the unit of Kb, which is the Boltzmann constant. The CvX quantiites are not of interest for general users. Interested users can look at the SaveSumEnergy routine in mc_main.cc to figure out what they are. CvB should be equal to the pure translational heat capacity when the translational motion is not coupled to any potential. CvC should be equal to the pure rotational heat capacity when the rotational motion is not coupled to any potential.

In $OUTPUT/$FILNAM.eng, column
	1: block index;
	2: block averaged translational energy;
	3: block averaged potential energy;
	4: block averaged translational + potential energies;
	5: block averaged rotational energy;
	6: heat capacity term for debugging;
	7: block averaged total energy (translational + potential + rotational);
	8: heat capacity related term (CvE) for calculating averaged Cv;
	9 and 10: heat capacity terms for debugging.

All energy quantities have the unit of K while CvE has the unit of K^2. The three debugging terms are not of general interest and not explained here. Users can calculate the averaged quantities and variances by treating each block averaged value as an observed result. Thus, the number of block has the meaning of the number of observation. The Averaged Cv should be calculated as

	            [ 1.5*N*P*T - < E > ]^2 + < CvE >
	< Cv > = -----------------------------------
                                 T^2

N is the total number of particles, including point-like particles and rotors, P the number of *translational* beads, and T the temperature in Kelvin. The resultant < Cv > has a unit of Kb, and its variance is obtained through error propagation.

*******************
   distribution
*******************

Whenever there are point-like particles, the $OUTPUT/$FILNAM_sum.gra contains pair radial distribution between point-like particles. The first column has radius and the second the rho(r). It follows the convention that integrating r from 0 to infinity rho(r)*dr gives 1.

If there is a linear rotor, the $OUTPUT/$FILNAM_sum.g2d file contains the 2-D distribution of the point-like particles around the rotor. The 2 dimensions are radius and polar-angle. The file has three columns, where column

	1: polar angles in degree;
	2: radius in Angstrom;
	3: density rho(r, theta).

The density follows the following normalization:

	Integrating from 0 to pi for theta and from 0 to infinity for r rho(r,theta)dr dtheta = 1.

For a non-linear rotor, there are three files:

	$OUTPUT/$FILNAM_sum.gri;
	$OUTPUT/$FILNAM_sum.grt;
	$OUTPUT/$FILNAM_sum.grc

which contain the radial (r), polar-angular (theta), and azimuthal-angular (chi) distributions of the point-like particles in the molecu-fixed frame (MFF) of the rotor. The MFF is the same as the one used in the definition of Eular angles. The densities follow the following normalizations:

	Integrating r from 0 to infinity rho(r) dr = number of point like particles;
	Integrating theta from 0 to pi rho(theta) dtheta = 1;
	Integrating chi from 0 to 2*pi rho(chi) dchi =1.

In addition, at the beginning of each block, the program punches out a $OUTPUT/$FILNAM###.xyz file to record the path configuration of the system. The file has N*P+2 lines, where N is the number of all particles (point-like + rotor) and P is the number of translational beads. Line one contains the permutational table for the configuration snapshot and line two is a comment line. The rest lines loop over all particles, point-like first and rotor second, and for each particle loop over all beads and punch out the corresponding configurations. Column 1 has the particle name, columns 2, 4, and 6 the x, y, and z coordinates in the space-fixed frame respectively, and columns 3, 5, 7 the phi, cos(theta), and chi angular coordinates. For linear rotors, phi and theta are the azimuthal and polar angles. For a top, phi, theta, and chi are the Eular angles defined in Figure 3.1, page 78 of Zare (Angular Momentum, 1988, John Wiley & Sons, Inc). When the number of rotational beads (Q) is a fraction of P, only the first Q lines of each particle block contains the meaningful angular coordinates.

*******************
   superfluidity
*******************

When bosons are involved, the program created a $OUTPUT/$FILNAM.sffs3d file to record the block-averaged classical moment of inertia of the bosons in the space-fixed frame (SFF) and the quantum deductions from area estimator. Readers should consult PRL, 1989, 63, 1601-1604 for more details of the area estimator. This estimator gives

	I_{ij}^{eff} = I_{ij}^{cl} - 4m^2 < A_i A_j > / ( beta*hbar^2 ),

where the subtrahend is the quantum deduction. In $OUTPUT/$FILNAM.sffs3d, column

	1: block indices;
	2 - 10 : the xx, xy, xz, yx, yy, yz, zx, zy, and zz elements of the block-averaged classical moment of inertia;
	11 - 16 : the xx, xy, xz, yy, yz, and zz components of the quantum deduction tensor.

All quantities above have the unit of u*Angs^2. If the simulation is ergodic, the off-diagonal elements of the I^{cl} and the quantum deduction tensor should be averaged to zeros and making the SFF effective moment of inertia to be diagonal.

If a top rotor is involved, then a $OUTPUT/$FILNAM.mffs3d file will be created. This file has the same format as the $OUTPUT/$FILNAM.sffs3d file but with the properties calculated along the molecule-fixed frame (MFF) axes. One can calculate the MFF effective moment of inertia using data in this file. Note that off-diagonal matrix element may occur since the MFF is usually anisotropic.

If a linear rotor is involved, a $OUTPUT/$FILNAM.sup file will be created. It has the following content:
	Column 1: block indices;
	Column 2: block-averaged superfluid fraction perpendicular to the rotor axis;
	Column 3: block-averaged superfluid fraction parallel to the rotor axis;
        Column 4: block-averaged classical moment of inertia perpendiculal to the rotor axis;
        Column 5: block-averaged classical moment of inertia parallel to the rotor axis;
        Column 6 and 7: debugging data.

Note that averaged fraction of column 2 multiplied by the averaged moment of inertia of column 4 gives the effective moment of inertia I_b of the effective symmetric top rotor and can be used to calculate the rotational constant B. Similarly, the non-degenerate rotational constant (A or C depending on prolate- or oblate-like) of the effective symmetric top rotor can be obtained from averaged quantities of columns 3 and 5.

When one rotor is involved in the simulation, no matter it is a linear or a top, the program creates a $OUTPUT/$FILNAM_sum.rcf file, which records the averaged imaginary time orientational correlation function for the z-axis of the MFF. This function can be used to extract effective rotational constant of the effective rotor. Please refer to JCP, 2004, 120, 5916-5931 for details.

Another superfluidity-relevant file is the $WORK/permutation.tab file. It records the permutation table of all the bosons from time to time and it can be used to calculate the probability of a boson being in a permutation cycle with a certain length.
