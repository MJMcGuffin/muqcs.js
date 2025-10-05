# <a href="https://mjmcguffin.github.io/muqcs.js/">muqcs.js</a>

Muqcs (pronounced mucks) is McGuffin's Useless Quantum Circuit Simulator.  It is written in JavaScript, and allows one to simulate circuits programmatically or from a command line.  It has no graphical front end, does not leverage the GPU for computations, and makes almost no use of external libraries (<a href="https://mathjs.org/">mathjs</a> is used in only one subroutine), making it easier for others to understand the core algorithms.  On many personal computers, it can simulate circuits of 20+ qubits, and can compute several statistics.

The code is contained entirely in a single file, and defines a small class for complex numbers, a class for complex matrices (i.e., matrices storing complex numbers), and a few utility classes like Sim (for Simulator).  These classes take up about 2000 lines of code.  The rest of the code consists of a regression test (in the function performRegressionTest()).  Having a relatively small amount of source code means that the code is easier to study and understand by others.

Like <a href="https://algassert.com/quirk">Quirk</a>, Muqcs supports quantum logic gates with arbitrary combinations of control and anti-control qubits.
Unlike other javascript quantum circuit simulators, Muqcs also implements partial trace on arbitrary subsets of qubits to compute
reduced density matrices, from which several statistics can be computed:
the phase, probability, purity, linear entropy, von Neumann entropy (to quantify mixedness), and Bloch sphere coordinates of individual qubits;
the concurrence (to quantify entanglement), correlation, purity, and von Neumann entropy of pairs of qubits;
and the stabilizer Rényi entropy (to quantify magic) of a set of qubits.

On my 2022 laptop, running inside Chrome, Muqcs can simulate circuits on N=20 qubits at a speed of less than 100ms per gate, and can also compute all N(N-1)/2 4×4 2-qubit partial traces and all N 2×2 1-qubit partial traces in under 22 seconds, all without ever computing any explicit $2^N \times 2^N$ matrices.

To run the code, <a href="https://mjmcguffin.github.io/muqcs.js/">load the html file</a> into a browser like Chrome, and then open a console (in Chrome, this is done by selecting 'Developer Tools').  From the console prompt, you can call functions in the code and see output printed to the console.

Muqcs is named in allusion to <a href="https://sourceforge.net/projects/mush/">mush</a>, Moser's Useless SHell, written by Derrick Moser (dtmoser).
<!-- 2025-06-01: The commands \bra, \ket, \braket used to render properly, but now they don't, so I'm defining my own versions here. Unfortunately, these render with extra spaces when used with the symbols -, +, -i, +i, but I don't know how to fix this as none of \! nor \hspace{-3mu} nor \mspace{-3mu} nor \kern{-3mu} work. -->
<!-- Using \left, \right makes the symbols excessively big when the expression inside has a subscript.
$\newcommand{\mybra}[1]{\left\langle #1 \right|}$
$\newcommand{\myket}[1]{\left| #1 \right\rangle}$
$\newcommand{\mybraket}[2]{\left\langle #1 \middle| #2 \right\rangle}$
$\newcommand{\myketbra}[2]{\left| #1 \right\rangle \left\langle #2 \right|}$
-->
<!-- 2025-06-12: now \newcommand seems to have stopped working, so I will revert to using \bra, \ket, \braket
$\newcommand{\mybra}[1]{\langle #1|}$
$\newcommand{\myket}[1]{|#1 \rangle}$
$\newcommand{\mybraket}[2]{\langle #1|#2 \rangle}$
$\newcommand{\myketbra}[2]{|#1 \rangle \langle #2|}$
-->

## Companion Video

[![video overview of Muqcs](/doc/video-thumbnail-2.png)](https://youtu.be/b6OqXkqPBeY)
<br/>
<a href="https://youtu.be/b6OqXkqPBeY">View the video</a>


## Companion Paper

A companion paper at <a href="https://arxiv.org/abs/2506.08142">https://arxiv.org/abs/2506.08142</a> explains background concepts and algorithms used in Muqcs.
Please cite that paper if you would like to cite the Muqcs software in your own academic work.

## Related Software

After Muqcs and the companion paper were released, Thomas Trenty re-implemented and extended parts of it in Mojo <a href="https://github.com/ttrenty/QLabs">https://github.com/ttrenty/QLabs</a> , for a Pythonic syntax and much faster execution speed.

<a href="https://profs.etsmtl.ca/mmcguffin/research/#mcguffin_2025_MuqcsCraft">MuqcsCraft</a> is a web-based graphical simulator and visualizer for quantum circuits, built on top of Muqcs.

## Creating and Manipulating Matrices

To create some matrices and print out their contents, we can do

    let m0 = new CMatrix(2,3);  // CMatrix means 'complex matrix', a matrix with complex entries
    console.log("A 2x3 matrix filled with zeros:\n" + m0.toString());
    let m1 = CMatrix.create([[10,20],[30,40]]);
    console.log("A 2x2 matrix:\n" + m1.toString());
    let c0 = CMatrix.createColVector([5,10,15]);
    console.log("The transpose of a column vector is a row vector:\n" + c0.transpose().toString());

which produces this output:

    A 2x3 matrix filled with zeros:
    [_,_,_]
    [_,_,_]
    A 2x2 matrix:
    [10,20]
    [30,40]
    The transpose of a column vector is a row vector:
    [5,10,15]

Notice that the toString() method returns a string containing newline characters.  In the source code, this is referred to as a 'multiline string'.
Next, we create a matrix containing complex numbers, add two matrices together, and print out the matrices and their sum:

    let m2 = CMatrix.create([[new Complex(0,1),new Complex(2,3)],[new Complex(5,7),new Complex(-1,-3)]]);
    let m3 = CMatrix.sum(m1,m2);
    console.log(StringUtil.concatMultiline(m1.toString()," + ",m2.toString()," = ",m3.toString()));

which produces this output:

    [10,20] + [1i  ,2+3i ] = [10+1i,22+3i]
    [30,40]   [5+7i,-1-3i]   [35+7i,39-3i]

Similarly, there are static methods in the CMatrix class for subtracting matrices (diff(m1,m2)), multiplying matrices (mult(m1,m2) and naryMult([m1,m2,...])), and for computing their tensor product (tensor(m1,m2) and naryTensor([m1,m2,...])).
There are also some predefined vectors and matrices.  For example,

    console.log(Sim.ketOne.toString());

prints the column vector $\ket{1}$:

    [_]
    [1]

and

    console.log(Sim.CX.toString());

prints the 4×4 matrix for the CNOT (also called CX) gate:

    [1,_,_,_]
    [_,_,_,1]
    [_,_,1,_]
    [_,1,_,_]

Notice that zeros are replaced with underscores to make it easier for a human to read sparse matrices - a form of pretty printing (to change this behavior, you can call toString({suppressZeros:false}) or toString({charToReplaceSuppressedZero:'.'})).  You might also notice that the matrix for the CNOT gate looks different from the way it is usually presented in textbooks or other sources.  This is related to the ordering of bits and ordering of tensor products.  Search for the usingTextbookConvention flag in the source code for comments that explain this, and set that flag to true if you prefer the textbook ordering.  We can also call a method on a matrix (or a vector) to change its ordering:

    console.log(Sim.CX.reverseEndianness().toString());

prints the 4×4 matrix for the CNOT gate in its more usual form:

    [1,_,_,_]
    [_,1,_,_]
    [_,_,_,1]
    [_,_,1,_]

## Simulating a Quantum Circuit

To simulate a circuit, one approach is to compute an explicit matrix for each layer (or step or stage) of the circuit.  In a circuit with $n$ qubits, these matrices for each layer will have size $2^n \times 2^n$.  Consider this 3-qubit circuit with 4 layers:

![Example circuit 1](/doc/exampleCircuit-1.png)

The input state vector $\ket{\psi_0}$ is an $8 \times 1$ column vector, computed as the tensor product
$\ket{\psi_0}$ = $\ket{0}$ $\otimes$ $\ket{0}$ $\otimes$ $\ket{0}$ = $\ket{000}$ = $\ket{0}^{\otimes 3}$,
which can be computed in any of the following ways:

    psi_0 = CMatrix.tensor( Sim.ketZero, CMatrix.tensor( Sim.ketZero, Sim.ketZero ) );
    console.log( psi_0.toString() );
    psi_0 = CMatrix.naryTensor( [ Sim.ketZero /*q2*/, Sim.ketZero /*q1*/, Sim.ketZero /*q0*/ ] );
    console.log( psi_0.toString() );
    psi_0 = CMatrix.tensorPower( Sim.ketZero, 3 );
    console.log( psi_0.toString() );

The quantum logic gates for $X$, $Z$, and $H$ each correspond to particular $2 \times 2$ matrices.
The $8 \times 8$ matrix $L_1$ is computed as the tensor product $L_1 = X \otimes H \otimes I$, where $I$ is the $2 \times 2$ identity matrix.  In code, we can do

    let L_1 = CMatrix.naryTensor( [ Sim.X, Sim.H, Sim.I ] ); // these are ordered q2, q1, q0
    console.log( L_1.toString() );

Layers 2 and 4 involve gates with control bits.
The CX (also called controlled-X or controlled-NOT or CNOT) gate in layer 4 is described by a particular $4 \times 4$ matrix,
and the upside-down CX in layer 2 is described by a similar $4 \times 4$ matrix.
Here is how we could compute the $8 \times 8$ matrix for each layer and compute the $8 \times 1$ output of the circuit:

    // View this circuit in Quirk with
    //   https://algassert.com/quirk#circuit={%22cols%22:[[1,%22H%22,%22X%22],[%22X%22,%22%E2%80%A2%22],[%22Z%22],[1,%22%E2%80%A2%22,%22X%22]]}
    let n = 3; // number of qubits
    let psi_0 = CMatrix.tensorPower(Sim.ketZero,n); // initialization: ψ = |0> ⊗ |0> ⊗ |0> = |0>^(⊗3)
    let L_1 = CMatrix.naryTensor( [ Sim.X, Sim.H, Sim.I ] );
    let L_2 = CMatrix.tensor( Sim.I, Sim.CX.reverseEndianness() );
    // This would also work:
    // let L_2 = Sim.expand4x4ForNWires( Sim.CX, 1, 0, n );
    let L_3 = CMatrix.naryTensor( [ Sim.I, Sim.I, Sim.Z ] );
    let L_4 = CMatrix.tensor( Sim.CX, Sim.I );
    // This would also work:
    // let L_4 = Sim.expand4x4ForNWires( Sim.CX, 1, 2, n );
    let psi_f = CMatrix.naryMult( [ L_4, L_3, L_2, L_1, psi_0 ] );
    // Print the output state vector:
    //console.log( psi_f.toString({binaryPrefixes:true}) );
    // Print a summarized description of the computation:
    console.log(StringUtil.concatMultiline(
        L_4.toString(),
        " * ", L_3.toString(),
        " * ", "...", // L_2.toString(),
        " * ", "...", // L_1.toString({decimalPrecision:1}),
        " * ", psi_0.toString(), // input
        " = ", psi_f.toString({binaryPrefixes:true}) // output
    ));

Each L_i matrix takes up $O((2^n)^2)=O(4^n)$ space (one gigabyte for $n=13$ qubits, assuming 16 bytes per complex number), and calling CMatrix.mult() to multiply two such matrices would cost $O((2^n)^3)=O(8^n)$ time.
However, the call to naryMult() above causes the matrices to be multiplied right-to-left, because naryMult() checks the sizes of the matrices to optimize the multiplication order, and the right-most matrix passed to naryMult() is just a column vector of size $2^n \times 1$.  The matrix just before that has size $2^n \times 2^n$, and multiplying the two together costs $O((2^n)^2)=O(4^n)$ time and produces another column vector, which gets multiplied by the next matrix before them, etc. 
Hence, in the above approach, the space and time requirements of each layer of the circuit are $O((2^n)^2)=O(4^n)$.

Notice that in a call to toString() above, we can optionally pass in {binaryPrefixes:true}; this causes bit strings like |000> to be printed in front of the matrix, as a reminder of the association between base states and matrix rows.
The output of the above code is:

    [1,_,_,_,_,_,_,_]   [1,0 ,0,0 ,0,0 ,0,0 ]               [1]   |000>[0     ]
    [_,1,_,_,_,_,_,_]   [0,-1,0,0 ,0,0 ,0,0 ]               [_]   |001>[0     ]
    [_,_,_,_,_,_,1,_]   [0,0 ,1,0 ,0,0 ,0,0 ]               [_]   |010>[0     ]
    [_,_,_,_,_,_,_,1] * [0,0 ,0,-1,0,0 ,0,0 ] * ... * ... * [_] = |011>[-0.707]
    [_,_,_,_,1,_,_,_]   [0,0 ,0,0 ,1,0 ,0,0 ]               [_]   |100>[0.707 ]
    [_,_,_,_,_,1,_,_]   [0,0 ,0,0 ,0,-1,0,0 ]               [_]   |101>[0     ]
    [_,_,1,_,_,_,_,_]   [0,0 ,0,0 ,0,0 ,1,0 ]               [_]   |110>[0     ]
    [_,_,_,1,_,_,_,_]   [0,0 ,0,0 ,0,0 ,0,-1]               [_]   |111>[0     ]

A second approach to simulating the same circuit is to not compute any explicit matrices of size $2^N \times 2^N$.  Instead, we only store the state vector of size $2^N \times 1$, and update it for each layer of the circuit, using qubit-wise multiplication.  The following code does this:

    let n = 3; // number of qubits
    let psi_0 = CMatrix.tensorPower(Sim.ketZero,n); // initialization: ψ = |0> ⊗ |0> ⊗ |0> = |0>^(⊗3)
    let step1 = Sim.qubitWiseMultiply(Sim.H,1,n,psi_0);
    step1 = Sim.qubitWiseMultiply(Sim.X,2,n,step1);
    let step2 = Sim.qubitWiseMultiply(Sim.X,0,n,step1,[[1,true]] /*one control bit on wire 1*/ );
    let step3 = Sim.qubitWiseMultiply(Sim.Z,0,n,step2);
    let psi_f = Sim.qubitWiseMultiply(Sim.X,2,n,step3,[[1,true]] /*one control bit on wire 1*/ );
    // Print the output state vector:
    console.log( psi_f.toString({binaryPrefixes:true}) );
    // Print a summarized description of the computation:
    console.log(StringUtil.concatMultiline(
        psi_0.toString(), // input
        " -> ", step1.toString(),
        " -> ", step2.toString(),
        " -> ", step3.toString(),
        " -> ", psi_f.toString({binaryPrefixes:true}) // output
    ));

In this second approach, the space and time requirements of each layer (or step) of the circuit are $O(2^n)$, so, much better than in the previous approach.
The Sim.qubitWiseMultiply() routine takes $O(2^n)$ time, and accepts an optional last parameter listing an arbitrary combination of control and anti-control bits, making it easy to implement, for example, Toffoli gates or other gates involving multiple control bits.

<!--
The key algorithm enabling this is in Sim.qubitWiseMultiply() method, which is inspired by Quirk's source code https://github.com/Strilanc/Quirk/ , in particular, Quirk's applyToStateVectorAtQubitWithControls() method in src/math/Matrix.js (<a href="https://github.com/Strilanc/Quirk/blob/master/src/math/Matrix.js#L678">link to specific line</a>).  This is essentially the "qubit-wise multiplication" algorithm described in chapter 6 of the book Viamontes, G. F., Markov, I. L., & Hayes, J. P. (2009) "Quantum circuit simulation", although their pseudocode contains errors and does not support control bits.

More explanation and code examples appear in the slides under the doc folder of the repository.
-->

To implement SWAP gates, consider another example circuit, containing a SWAP gate and a controlled SWAP (or Fredkin) gate:

![Example circuit 2](/doc/exampleCircuit-2.png)

We can compute its output with this:

    // View this circuit in Quirk with
    //   https://algassert.com/quirk#circuit={%22cols%22:[[%22H%22],[%22Swap%22,1,%22Swap%22],[1,%22X%22,%22%E2%97%A6%22],[%22X%22,%22%E2%80%A2%22],[%22Y%22],[%22%E2%80%A2%22,%22Swap%22,%22Swap%22],[1,%22Z%22]]}
    n = 3; // number of qubits
    let ψ = CMatrix.tensorPower(Sim.ketZero,n); // initialization: ψ = |0> ⊗ |0> ⊗ |0> = |0>^(⊗3)
    ψ = Sim.qubitWiseMultiply(Sim.H,0,n,ψ);
    ψ = Sim.applySwap(0,2,n,ψ);
    ψ = Sim.qubitWiseMultiply(Sim.X,1,n,ψ,[[2,false]] /*one anti-control bit on wire 2*/ );
    ψ = Sim.qubitWiseMultiply(Sim.X,0,n,ψ,[[1,true]] /*one control bit on wire 1*/ );
    ψ = Sim.qubitWiseMultiply(Sim.Y,0,n,ψ);
    ψ = Sim.applySwap(1,2,n,ψ,[[0,true]] /*one control bit on wire 0*/ );
    ψ = Sim.qubitWiseMultiply(Sim.Z,1,n,ψ);
    console.log( ψ.toString({binaryPrefixes:true}) );

Consider another example, where we will compute partial traces.
To simulate this circuit...

![Example circuit 3](/doc/exampleCircuit-3.png)

... we can use the following code snippet:

    // View this circuit in Quirk with
    //   https://algassert.com/quirk#circuit={%22cols%22:[[%22H%22,1,%22H%22],[%22%E2%80%A2%22,%22X%22]]}
    n = 3; // number of qubits
    let ψ = CMatrix.tensorPower(Sim.ketZero,n); // initialization: ψ = |0> ⊗ |0> ⊗ |0> = |0>^(⊗3)
    ψ = Sim.qubitWiseMultiply(Sim.H,0,n,ψ);
    ψ = Sim.qubitWiseMultiply(Sim.H,2,n,ψ);
    ψ = Sim.qubitWiseMultiply(Sim.X,1,n,ψ,[[0,true]] /*one control bit on wire 0*/ );
    console.log( ψ.toString({binaryPrefixes:true}) );

Let rho_210 be the $8 \times 8$ density matrix for all 3 qubits, and rho_ij be the $4 \times 4$ reduced density matrix for qubits i and j, and rho_i be the $2 \times 2$ reduced density matrix for qubit i.  Here is one way to compute all the reduced matrices:

    let rho_210 = Sim.computeDensityMatrix( n, ψ );
    let rho_0 = Sim.partialTrace( n, rho_210, true, null, [1,2], true /* the preceding list says which qubits to trace out */ );
    let rho_1 = Sim.partialTrace( n, rho_210, true, null, [0,2], true );
    let rho_2 = Sim.partialTrace( n, rho_210, true, null, [0,1], true );
    let rho_10 = Sim.partialTrace( n, rho_210, true, null, [2], true );
    let rho_20 = Sim.partialTrace( n, rho_210, true, null, [1], true );
    let rho_21 = Sim.partialTrace( n, rho_210, true, null, [0], true );

Rather than passing in a list of qubits to trace out, we can instead pass in a list of qubits to keep,
which can simplify things, e.g.

    let rho_0 = Sim.partialTrace( n, rho_210, true, null, [0], false /* the preceding list says which qubits to keep */ );

In the above examples, we explicitly compute the full density matrix rho_210.  However, computing rho_210 from the state vector requires $O((2^n)^2) = O(4^n)$ time and space which is expensive for large $n$.
Another approach is to pass in the state vector to the partialTrace() routine,
which saves a huge amount of time for large $n$:

    let rho_0 = Sim.partialTrace( n, null, true, ψ, [0], false );
    let rho_1 = Sim.partialTrace( n, null, true, ψ, [1], false );
    let rho_2 = Sim.partialTrace( n, null, true, ψ, [2], false );
    let rho_10 = Sim.partialTrace( n, null, true, ψ, [2], true );
    let rho_20 = Sim.partialTrace( n, null, true, ψ, [1], true );
    let rho_21 = Sim.partialTrace( n, null, true, ψ, [0], true );

Printing the reduced density matrices can be done with statements like

    console.log(StringUtil.concatMultiline(
        "rho_2 = ", rho_2.toString(),
        ", rho_10 = ", rho_10.toString()
    ));
    console.log(StringUtil.concatMultiline(
        "rho_0 = ", rho_0.toString(),
        ", rho_21 = ", rho_21.toString()
    ));
    console.log( "rho_20 =\n" + rho_20.toString() );
    console.log( "rho_1 =\n" + rho_1.toString() );

Given the $2 \times 2$ reduced density matrix rho_i for qubit i, we can compute several statistics associated with qubit i.
The source code for the routine computeStatsFor2x2DensityMatrix() shows how to compute
the qubit's phase and purity,
its Bloch sphere coordinates,
its linear entropy and von Neumann entropy,
and the probability of measuring a 1 on that qubit.

Given the $4 \times 4$ reduced density matrix rho_ij for qubits i and j, we can compute several statistics associated with the pair of qubits i and j.
For more about this, in the source code, see the routines
computePairwiseQubitCorrelations(),
computePairwiseQubitConcurrences(),
computePairwiseQubitPurity(),
computePairwiseQubitVonNeumannEntropy()
which compute statistics for every pair of qubits.

We can also compute the Second Stabilizer Rényi Entropy (SSRE) to quantify magic of a set of qubits.
We do this by finding the $2^k \times 2^k$ reduced density matrix for the $k$ qubits of interest,
and then call the computeSSREMagic() routine.
See the comments preceding the definition of that routine in the source code for an example of how to call it.


## Circuit and Qubit Statistics

Here we consider in more detail a particular circuit and the statistics that can be computed for it, and compare these to the output of IBM Quantum Composer and of Quirk, to provide convincing evidence that Muqcs correctly computes these statistics.

From the amplitudes of an output state vector, we can easily find the probability of each computational basis state.
In addition, muqcs can compute the ($2^N \times 2^N$) density matrix for a give state vector,
and also compute the (2×2) reduced density matrix (using the partial trace) for each qubit, from which we can compute
the phase, Bloch sphere coordinates, and purity (also called 'reduced purity' or 'purity of reduced state') for each qubit.
The Bloch sphere coordinates are a way to describe the qubit's 'local state'.
The purity for a single qubit varies from 0.5 to 1.0 and indicates how entangled the qubit is with the rest of the system:
0.5 means maximally entangled, 1.0 means not entangled, and an intermediate value means partially mixed.
Here is an example computing these statistics with muqcs:

    let N = 4; // total qubits
    input = CMatrix.tensorPower(Sim.ketZero,N);
    step1 = CMatrix.naryTensor( [ Sim.RY(45) /*q3*/, Sim.RX_90deg /*q2*/,
                                  Sim.RX_90deg /*q1*/, Sim.RX(45) /*q0*/ ] );
    step2 = CMatrix.naryTensor( [ Sim.RX(45) /*q3*/, Sim.RZ(120) /*q2*/,
                                  Sim.RZ(100) /*q1*/, Sim.I /*q0*/ ] );
    output = CMatrix.naryMult([ step2, step1, input ]);
    output = Sim.qubitWiseMultiply(Sim.RZ_90deg,2,N,output,[[1,true]]/*list of control qubits*/);
    output = Sim.qubitWiseMultiply(Sim.RY(45),2,N,output,[[3,true]]/*list of control qubits*/);
    baseStateProbabilities = new CMatrix( output._rows, 1 );
    for ( let i=0; i < output._rows; ++i ) baseStateProbabilities.set( i, 0, output.get(i,0).mag()**2 );
    console.log(StringUtil.concatMultiline(
        "Output: ", output.toString({binaryPrefixes:true}), ", Probabilities: ", baseStateProbabilities.toString()
    ));
    Sim.printAnalysisOfEachQubit(N,output);

![Qubit statistics in Muqcs](/doc/qubit-stats-muqcs-1.png)

![More qubit statistics in Muqcs](/doc/qubit-stats-muqcs-2.png)

<!-- previously https://quantum-computing.ibm.com/composer -->
... and now the same circuit in <a href="https://quantum.cloud.ibm.com/composer">IBM Quantum Composer</a>:

    // Copy-paste the below instructions into IBM’s website at https://quantum.cloud.ibm.com/composer to recreate the circuit

    OPENQASM 2.0;
    include "qelib1.inc";

    qreg q[4];
    rx(pi / 2) q[1];
    rx(pi / 2) q[2];
    rx(pi/4) q[0];
    ry(pi/4) q[3];
    rz(1.7453292519943295) q[1];
    rz(2.0943951023931953) q[2];
    rx(pi/4) q[3];
    crz(pi / 2) q[1], q[2];
    cry(pi/4) q[3], q[2];

![Qubit statistics in IBM Quantum Composer](/doc/qubit-stats-ibm-quantum-composer.png)

... and the same circuit in Quirk:

https://algassert.com/quirk#circuit=%7B%22cols%22%3A%5B%5B%7B%22id%22%3A%22Rxft%22%2C%22arg%22%3A%22pi%2F4%22%7D%2C%7B%22id%22%3A%22Rxft%22%2C%22arg%22%3A%22pi%2F2%22%7D%2C%7B%22id%22%3A%22Rxft%22%2C%22arg%22%3A%22pi%2F2%22%7D%2C%7B%22id%22%3A%22Ryft%22%2C%22arg%22%3A%22pi%2F4%22%7D%5D%2C%5B1%2C%7B%22id%22%3A%22Rzft%22%2C%22arg%22%3A%221.7453292519943295%22%7D%2C%7B%22id%22%3A%22Rzft%22%2C%22arg%22%3A%222.0943951023931953%22%7D%2C%7B%22id%22%3A%22Rxft%22%2C%22arg%22%3A%22pi%2F4%22%7D%5D%2C%5B%5D%2C%5B%5D%2C%5B%5D%2C%5B1%2C%22%E2%80%A2%22%2C%7B%22id%22%3A%22Rzft%22%2C%22arg%22%3A%22pi%2F2%22%7D%5D%2C%5B1%2C1%2C%7B%22id%22%3A%22Ryft%22%2C%22arg%22%3A%22pi%2F4%22%7D%2C%22%E2%80%A2%22%5D%2C%5B%22Chance4%22%5D%2C%5B%22Density4%22%5D%2C%5B%5D%2C%5B%5D%2C%5B%5D%2C%5B%22Density%22%2C%22Density%22%2C%22Density%22%2C%22Density%22%5D%5D%7D

![Qubit statistics in Quirk](/doc/qubit-stats-quirk.png)


## Conventions

In a circuit with N qubits, the wires are numbered 0 for the top wire to (N-1) for the bottom wire.  The top wire encodes the Least-Significant Bit (LSB).

## Limitations

There is currently no support for measurement gates.

The code depends on <a href="https://mathjs.org/">mathjs</a>,
but only in one subroutine (Sim.eigendecomposition()) which is used to compute concurrence and von Neumann entropy.

## Background Notes

Think of bra ($\bra{a}$) as a row vector, and ket ($\ket{a}$) as a column vector equal to the conjugate transpose of the bra.
Then, multiplying a bra by a ket yields a dot product (i.e., $(\bra{a})(\ket{b})$, abbreviated to $\braket{a|b}$, yields a 1×1 matrix);
multiplying a bra by its corresponding ket ($\braket{a|a}$) yields a dot product equal to the
sum of the squared magnitudes of the complex numbers in the bra;
and multiplying a ket by its corresponding bra ($\ket{a}\bra{a}$) yields a square matrix called
the density matrix, whose trace (sum of elements along the diagonal) is equal to $Tr(\ket{a}\bra{a}) = \braket{a|a}$.

Some predefined basis vectors:

| Common names | Muqcs code | Size (rows x columns) | Matrix | Notes |
| :---: | --- | :---: | --- | --- |
|  $\langle 0 \|$  | `Sim.braZero` | 1x2 | <pre>[ 1 0 ]</pre> | |
|  $\| 0 \rangle$  | `Sim.ketZero` | 2x1 | <pre>[ 1 ]<br>[ 0 ]</pre> | |
|  $\langle 1 \|$  | `Sim.braOne` | 1x2 | <pre>[ 0 1 ]</pre> | |
|  $\| 1 \rangle$  | `Sim.ketOne` | 2x1 | <pre>[ 0 ]<br>[ 1 ]</pre> | |
|  $\langle + \|$  | `Sim.braPlus` | 1x2 | <pre>(1/sqrt(2)) [ 1 1 ]</pre> | |
|  $\| + \rangle$  | `Sim.ketPlus` | 2x1 | <pre>(1/sqrt(2)) [ 1 ]<br>            [ 1 ]</pre> | $\ket{+} = \frac{1}{\sqrt{2}}(\ket{0} + \ket{1})$ |
|  $\langle - \|$  | `Sim.braMinus` | 1x2 | <pre>1/sqrt(2) [ 1 -1 ]</pre> | |
|  $\| - \rangle$  | `Sim.ketMinus` | 2x1 | <pre>(1/sqrt(2)) [  1 ]<br>            [ -1 ]</pre> | |
|  $\bra{+i}$  | `Sim.braPlusI` | 1x2 | <pre>(1/sqrt(2)) [ 1 -i ]</pre> |  |
|  $\ket{+i}$  | `Sim.ketPlusI` | 2x1 | <pre>(1/sqrt(2)) [ 1 ]<br>            [ i ]</pre> | $\ket{+i} = \frac{1}{\sqrt{2}}(\ket{0} + i\ket{1})$ |
|  $\bra{-i}$  | `Sim.braMinusI` | 1x2 | <pre>(1/sqrt(2)) [ 1 i ]</pre> | |
|  $\ket{-i}$  | `Sim.ketMinusI` | 2x1 | <pre>(1/sqrt(2)) [  1 ]<br>            [ -i ]</pre> | |

Consider a circuit of $N$ qubits where the overall state of the circuit is pure (implying that none of the qubits are entangled with the environment).  The state of the $N$ qubits can be described using a $2^N \times 1$ (column) state vector $\| \psi \rangle$, or using a $2^N \times 2^N$ density matrix $D = \ket{\psi}\bra{\psi}$.  To better understand some subset of $M$ qubits within the circuit, we can compute a partial trace of $D$ to "trace out" or "trace over" the other qubits, yielding a $2^M \times 2^M$ reduced density matrix $R$.  The purity of $R$ is given by the trace of $R^2$, and one minus that purity gives the linear entropy, which is an approximation of the von Neumann entropy (https://www.quantiki.org/wiki/linear-entropy) of the subset of $M$ qubits.  Purity ranges from $1/(2^M)$ to 1.0, linear entropy ranges from 0.0 to $1-1/(2^M)$, and von Neumann entropy ranges from 0.0 to M.  Entropy is a measure of the mixedness (the opposite of purity) of the subset of qubits.  Concurrence is a measure of how much the qubits are entangled with other qubits within the same subset.  There's a nice table at https://physics.stackexchange.com/questions/643578/what-are-the-relations-between-mixed-pure-and-separable-entangled-states showing types of states, by crossing {pure, mixed} $\times$ {product, separable, entangled}.  For more explanation of mixed and entangled states, you might like the explanations in <a href="https://arxiv.org/abs/2506.08142">our tutorial paper</a>.

A matrix $M$ is *unitary* if its inverse is equal to its conjugate transpose, i.e., $M^{-1} = M^{\dagger}$

A matrix $M$ is *hermitian* if it is equal to its own conjugate transpose, i.e., $M = M^{\dagger}$, which implies that the diagonal elements are real, and the off-diagonal elements are conjugates of each other (i.e., diagonally-opposite entries are complex conjugates).

A matrix $M$ is *involutory* if it is equal to its own inverse, $M = M^{-1}$

Any two of the above properties implies the third.  All valid quantum gates are described by matrices that are unitary.
Some of them (like I, X, Y, Z, H, CX, SWAP) are described by matrices that are additionally hermitian and involutory.

A density matrix is always hermitian, and its diagonal elements are real-valued and sum to 1.

Circuits consisting only of Clifford gates (which includes I, H, X, Y, Z, SX, SY, SZ, CX, SWAP) can be simulated in polynomial time on a classical computer, by the Gottesman-Knill theorem.  Thus, mere superposition (which can be created with H gates) and entanglement (CX gates) are not sufficient to explain the speedup offered by quantum computers.

Matrices encoding the effect of a quantum gate:

| Common names | Muqcs code | Qubits | Size | Notes |
| --- | --- | :---: | :---: | --- |
| zero, 0        | `Sim.ZERO` | 1 | 2x2 | not unitary |
| identity, I    | `Sim.I`    | 1 | 2x2 | no-op <br> I = Phase(0) |
| Hadamard, H    | `Sim.H`    | 1 | 2x2 |  |
| Pauli X, NOT   | `Sim.X`    | 1 | 2x2 | bit flip <br> X = -iYZ = iZY = HZH |
| Pauli Y        | `Sim.Y`    | 1 | 2x2 | Y = iXZ = -iZX = $\sqrt{Z} X \sqrt{Z}^{-1}$ |
| Pauli Z, Phase($\pi$) | `Sim.Z` or `Sim.Phase(180)` | 1 | 2x2 | phase flip <br> Z = Phase(180) <br> Z = -iXY = iYX = HXH |
| $\sqrt{X}$, SX, $\sqrt{NOT}$, V | `Sim.SX` | 1 | 2x2 | The name SX means 'Square root of X' <br> $\sqrt{X} = H \sqrt{Z} H = \sqrt{Y} \sqrt{Z} \sqrt{Y}^{-1}$ <br> $V = H S H = \sqrt{Y} S \sqrt{Y}^{-1}$ |
| $\sqrt{Y}$, SY        | `Sim.SY` | 1 | 2x2 | $\sqrt{Y} = H Z e^{i\pi/4} = X H e^{i\pi/4} = \sqrt{X}^{-1} \sqrt{Z} \sqrt{X}$ <br> $\sqrt{Y} = V^{-1} S V$ |
| $\sqrt{Z}$, SZ, Phase($\pi/2$), S | `Sim.SZ` or `Sim.Phase(90)` | 1 | 2x2 | SZ = Phase(90) <br> $\sqrt{Z} = H \sqrt{X} H$ <br> $S = H V H$ |
| $\sqrt[4]{X}$         | `Sim.SSX` | 1 | 2x2 | The name SSX means 'Square root of Square root of X' <br> $\sqrt[4]{X} = \sqrt{Y} \sqrt[4]{Z} \sqrt{Y}^{-1}$ <br> $\sqrt[4]{X} = \sqrt{Y} T \sqrt{Y}^{-1}$ |
| $\sqrt[4]{Y}$         | `Sim.SSY` | 1 | 2x2 | $\sqrt[4]{Y} = \sqrt{X}^{-1} \sqrt[4]{Z} \sqrt{X}$ <br> $\sqrt[4]{Y} = V^{-1} T V$ |
| $\sqrt[4]{Z}$, Phase($\pi/4$), T, $\pi/8$ | `Sim.SSZ` or `Sim.Phase(45)` | 1 | 2x2 | SSZ = Phase(45) |
| global phase shift    | `Sim.GlobalPhase (angleInDegrees)` |  1 | 2x2 | Has the same effect regardless of which qubit it is applied to; causes an equal phase shift in all amplitudes. Cannot be physically measured. |
| phase shift    | `Sim.Phase (angleInDegrees)` | 1 | 2x2 | Z = Phase(180) |
| $R_x$          | `Sim.RX (angleInDegrees)`    | 1 | 2x2 | RX(a) = RZ(-90) RY(a) RZ(90) <br> RX(a) = RY(90) RZ(a) RY(-90) |
| $R_y$          | `Sim.RY (angleInDegrees)`    | 1 | 2x2 | RY(a) = RX(-90) RZ(a) RX(90) |
| $R_z$          | `Sim.RZ (angleInDegrees)`    | 1 | 2x2 | Phase(angle) * GlobalPhase( -angle/2 ) = RZ( angle ) <br> Phase(angle) = RZ( angle ) * GlobalPhase( angle/2 ) <br> Z = Phase(180) = RZ(180) * GlobalPhase(90) |
|                | `Sim.RotFreeAxis (ax,ay,az)` | 1 | 2x2 | The angle, in radians, is encoded in the length of the given vector |
|                | `Sim.RotFreeAxisAngle (ax,ay,az, angleInDegrees)` | 1 | 2x2 |  |
|                | `Sim.SWAP_2`      | 2 | 4x4              |  |
|                | `Sim.SWAP(i,j,n)` | 2 | $2^n \times 2^n$ |  |
| CNOT, CX, XOR  | `Sim.CX`          | 2 | 4x4              |  |

## Matrix Definitions

zero, 0, `Sim.ZERO`
```math
\begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix}
```

identity, I, `Sim.I`
```math
\begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}
```

Hadamard, H, `Sim.H`
```math
\frac{1}{\sqrt{2}} \begin{bmatrix} 1 & 1 \\ 1 & -1 \end{bmatrix}
```

Pauli X, NOT, `Sim.X`
```math
\begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix}
```

Pauli Y, `Sim.Y`
```math
\begin{bmatrix} 0 & -i \\ i & 0 \end{bmatrix}
```

Pauli Z, Phase($\pi$), `Sim.Z`, `Sim.Phase(180)`
```math
\begin{bmatrix} 1 & 0 \\ 0 & -1 \end{bmatrix}
```

$\sqrt{X}$, SX, $\sqrt{NOT}$, V, `Sim.SX`
```math
\frac{1}{2} \begin{bmatrix} 1+i & 1-i \\ 1-i & 1+i \end{bmatrix}
```

$\sqrt{X}^{-1}$, $V^{-1}$, `Sim.invSX`
```math
\frac{1}{2} \begin{bmatrix} 1-i & 1+i \\ 1+i & 1-i \end{bmatrix}
```

$\sqrt{Y}$, SY, `Sim.SY`
```math
\frac{1}{2} \begin{bmatrix} 1+i & -1-i \\ 1+i & 1+i \end{bmatrix}
```

$\sqrt{Y}^{-1}$, `Sim.invSY`
```math
\frac{1}{2} \begin{bmatrix} 1-i & 1-i \\ -1+i & 1-i \end{bmatrix}
```

$\sqrt{Z}$, SZ, Phase($\pi/2$), S, `Sim.SZ`, `Sim.Phase(90)`
```math
\begin{bmatrix} 1 & 0 \\ 0 & i \end{bmatrix}
```

$\sqrt{Z}^{-1}$, Phase($-\pi/2$), $S^{-1}$, `Sim.invSZ`, `Sim.Phase(-90)`
```math
\begin{bmatrix} 1 & 0 \\ 0 & -i \end{bmatrix}
```

$\sqrt[4]{X}$, `Sim.SSX`
```math
\frac{1}{2} \begin{bmatrix} 1+e^{i \pi/4} & 1-e^{i \pi/4} \\ 1-e^{i \pi/4} & 1+e^{i \pi/4} \end{bmatrix}
= \begin{bmatrix} (2+\sqrt{2})/4 + i/(2 \sqrt{2}) & (2-\sqrt{2})/4 - i/(2 \sqrt{2}) \\ (2-\sqrt{2})/4 - i/(2 \sqrt{2}) & (2+\sqrt{2})/4 + i/(2 \sqrt{2}) \end{bmatrix}
```

$\sqrt[4]{X}^{-1}$, `Sim.invSSX`
```math
\frac{1}{2} \begin{bmatrix} 1+e^{-i \pi/4} & 1-e^{-i \pi/4} \\ 1-e^{-i \pi/4} & 1+e^{-i \pi/4} \end{bmatrix}
= \begin{bmatrix} (2+\sqrt{2})/4 - i/(2 \sqrt{2}) & (2-\sqrt{2})/4 + i/(2 \sqrt{2}) \\ (2-\sqrt{2})/4 + i/(2 \sqrt{2}) & (2+\sqrt{2})/4 - i/(2 \sqrt{2}) \end{bmatrix}
```

$\sqrt[4]{Y}$, `Sim.SSY`
```math
\frac{1}{2} \begin{bmatrix} 1+e^{i \pi/4} & i(e^{i \pi/4}-1) \\ i(1-e^{i \pi/4}) & 1+e^{i \pi/4} \end{bmatrix}
= \begin{bmatrix} (2+\sqrt{2})/4 + i/(2 \sqrt{2}) & -1/(2 \sqrt{2})-i (2-\sqrt{2})/4 \\ 1/(2 \sqrt{2})+i (2-\sqrt{2})/4 & (2+\sqrt{2})/4 + i/(2 \sqrt{2}) \end{bmatrix}
```

$\sqrt[4]{Y}^{-1}$, `Sim.invSSY`
```math
\frac{1}{2} \begin{bmatrix} 1+e^{-i \pi/4} & i(e^{-i \pi/4}-1) \\ i(1-e^{-i \pi/4}) & 1+e^{-i \pi/4} \end{bmatrix}
= \begin{bmatrix} (2+\sqrt{2})/4 - i/(2 \sqrt{2}) & 1/(2 \sqrt{2})-i (2-\sqrt{2})/4 \\ -1/(2 \sqrt{2})+i (2-\sqrt{2})/4 & (2+\sqrt{2})/4 - i/(2 \sqrt{2}) \end{bmatrix} 
```

$\sqrt[4]{Z}$, Phase($\pi/4$), T, $\pi/8$, `Sim.SSZ`, `Sim.Phase(45)`
```math
\begin{bmatrix} 1 & 0 \\ 0 & e^{i \pi/4} \end{bmatrix}
```

$\sqrt[4]{Z}^{-1}$, Phase($-\pi/4$), $T^{-1}$, `Sim.invSSZ`, `Sim.Phase(-45)`
```math
\begin{bmatrix} 1 & 0 \\ 0 & e^{-i \pi/4} \end{bmatrix}
```

global phase shift, `Sim.GlobalPhase(angleInDegrees)`
```math
e^{i \theta} \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}
```

phase shift, `Sim.Phase(angleInDegrees)`
```math
\begin{bmatrix} 1 & 0 \\ 0 & e^{i \theta} \end{bmatrix}
```

$R_x$, `Sim.RX(angleInDegrees)`
```math
\begin{bmatrix} \cos(\theta/2) & -i \sin(\theta/2) \\ -i \sin(\theta/2) & \cos(\theta/2) \end{bmatrix}
= \cos(\theta/2) I - i \sin(\theta/2) X
```

$R_y$, `Sim.RY(angleInDegrees)`
```math
\begin{bmatrix} \cos(\theta/2) & -\sin(\theta/2) \\ \sin(\theta/2) & \cos(\theta/2) \end{bmatrix}
= \cos(\theta/2) I - i \sin(\theta/2) Y
```

$R_z$, `Sim.RZ(angleInDegrees)`
```math
\begin{bmatrix} e^{-i \theta/2} & 0 \\ 0 & e^{i \theta/2} \end{bmatrix}
= \cos(\theta/2) I - i \sin(\theta/2) Z
```

`Sim.RotFreeAxis(ax,ay,az)`
<p align="center">
  (see definition in source code)
</p>

`Sim.RotFreeAxisAngle(ax,ay,az, angleInDegrees)`
<p align="center">
  (see definition in source code)
</p>

`Sim.XE(k)`,
$X^k = R_x(k \pi) GlobalPhase(k \pi/2)$
```math
\frac{1}{2} \begin{bmatrix} 1+e^{i k \pi} & 1-e^{i k \pi} \\ 1-e^{i k \pi} & 1+e^{i k \pi} \end{bmatrix}
= \begin{bmatrix} \cos(k \pi/2) & -i \sin(k \pi/2) \\ -i \sin(k \pi/2) & \cos(k \pi/2) \end{bmatrix}
e^{i k \pi/2}
```

`Sim.YE(k)`,
$Y^k = R_y(k \pi) GlobalPhase(k \pi/2)$
```math
\frac{1}{2} \begin{bmatrix} 1+e^{i k \pi} & i(e^{i k \pi}-1) \\ i(1-e^{i k \pi}) & 1+e^{i k \pi} \end{bmatrix} = \begin{bmatrix} \cos(k \pi/2) & -\sin(k \pi/2) \\ \sin(k \pi/2) & \cos(k \pi/2) \end{bmatrix} e^{i k \pi/2}
```

`Sim.ZE(k)`,
$Z^k = R_z(k \pi) GlobalPhase(k \pi/2) = Phase(k \pi)$
```math
\begin{bmatrix} 1 & 0 \\ 0 & e^{i k \pi} \end{bmatrix} = \begin{bmatrix} e^{-i k \pi/2} & 0 \\ 0 & e^{i k \pi/2} \end{bmatrix} e^{i k \pi/2}
```

$H^k$
```math
\begin{bmatrix} (2+\sqrt{2})/4 + e^{i k \pi} (2-\sqrt{2})/4 & 1/(2 \sqrt{2}) - e^{i k \pi}/(2 \sqrt{2}) \\ 1/(2 \sqrt{2}) - e^{i k \pi}/(2 \sqrt{2}) & (2-\sqrt{2})/4 + e^{i k \pi} (2+\sqrt{2})/4 \end{bmatrix}
```


## Generalized Gates

These are non-standard operations proposed by McGuffin.

### Generalized Z

`Sim.Z_G(angle1InDegrees, angle2InDegrees)`

#### Definition:

```math
Z_G(a,b) = X \cdot Z^{a/\pi} \cdot X \cdot Z^{b/\pi} = \begin{bmatrix} e^{i a} & 0 \\ 0 & e^{i b} \end{bmatrix}
```

#### Special cases:

```math
I = Z_G(0,0)
```

```math
Z = Z_G(0,\pi)
```

```math
S^{\pm 1} = \sqrt{Z}^{\pm 1} = Z_G(0, \pm \pi/2)
```

```math
T^{\pm 1} = \sqrt[4]{Z}^{\pm 1} = Z_G(0, \pm \pi/4)
```

```math
Phase(a) = Z_G(0,a)
```

```math
GlobalPhase(a) = Z_G(a,a)
```

```math
R_z(a) =  Z_G(-a/2,a/2)
```

#### Identities:

```math
X \cdot Z^{a/\pi} \cdot X \cdot Z^{b/\pi} = Z^{b/\pi} \cdot X \cdot Z^{a/\pi} \cdot X
```

```math
Z_G(a,b) Z_G(c,d) = Z_G(a+c,b+d)
```

```math
Z_G(a,b) = X \cdot Z_G(b,a) \cdot X
```

```math
(Z_G(a,b))^{-1} = Z_G(-a,-b)
```

### Generalized Y

`Sim.Y_G(angle1InDegrees, angle2InDegrees)`

#### Definition:

```math
Y_G(a,b) = X \cdot Z_G(a,b) = Z^{a/\pi} \cdot X \cdot Z^{b/\pi} = \begin{bmatrix} 0 & e^{i b} \\ e^{i a} & 0 \end{bmatrix}
```

#### Special cases:

```math
X = Y_G(0,0)
```

```math
Y = Y_G(\pi/2,-\pi/2)
```

#### Identities:

```math
Y_G(a,b) Y_G(c,d) = Z_G(b+c,a+d)
```

```math
Y_G(a,b) Z_G(c,d) = Z_G(d,c) Y_G(a,b) = Y_G(a+c,b+d)
```

```math
(Y_G(a,b))^{-1} = Y_G(-b,-a)
```

### Generalized Hadamard

`Sim.H_G(angle1InDegrees, angle2InDegrees)`

#### Definition:

```math
H_G(a,b) = H \cdot Z_G(a,b) = \frac{1}{\sqrt{2}} \begin{bmatrix} e^{i a} & e^{i b} \\ e^{i a} & -e^{i b} \end{bmatrix}
```

#### Special case:

```math
H = H_G(0,0)
```

#### Identities:

```math
H_G(a,a) H_G(b,b) = Z_G(a+b,a+b)
```

```math
(H_G(a,b))^{-1} = Z_G(-a,-b) \cdot H
```


## Example code for Wolfram Language

It's useful to be able to check matrix math using a symbolic math package like Mathematica or wolframcloud.com.  Here is some code to get started:

    (* identity *)
    (* id = {{1, 0}, {0, 1}}; *)
    id = IdentityMatrix[2];

    (* Hadamard *)
    H = ((1/(2^0.5))*{{1, 1}, {1, -1}});

    (* Pauli *)
    X={{0,1},{1, 0}};
    Y={{0,-I},{I, 0}};
    Z={{1,0},{0, -1}};

    (* Square roots of Pauli: X^0.5, Y^0.5, Z^0.5
       SX, SY, SZ mean Square root of X, Y, Z
    *)
    SX = (0.5 * {{1+I, 1-I}, {1-I, 1+I}});
    SY = (0.5 * {{1+I, -1-I}, {1+I, 1+I}});
    SZ = ({{1, 0}, {0, I}});
    (* inverses X^-0.5, Y^-0.5, Z^-0.25 *)
    invSX = (0.5 * {{1-I, 1+I}, {1+I, 1-I}});
    invSY = (0.5 * {{1-I, 1-I}, {-1+I, 1-I}}) ;
    invSZ = ({{1, 0}, {0, -I}});

    (* fourth roots and inverses *)
    SSX = (0.5 * {{1+E^(I*Pi/4),1-E^(I*Pi/4)},{1-E^(I*Pi/4),1+E^(I*Pi/4)}});
    SSY = (0.5 * {{1+E^(I*Pi/4),I*(E^(I*Pi/4)-1)},{I*(1-E^(I*Pi/4)),1+E^(I*Pi/4)}});
    SSZ = ({{1,0},{0,E^(I*Pi/4)}});
    invSSX = {{(2+2^0.5)/4-I/(2*2^0.5),(2-2^0.5)/4+I/(2*2^0.5)},{(2-2^0.5)/4+I/(2*2^0.5),(2+2^0.5)/4-I/(2*2^0.5)}};
    invSSY = {{(2+2^0.5)/4-I/(2*2^0.5),1/(2*2^0.5)-I*(2-2^0.5)/4},{-1/(2*2^0.5)+I*(2-2^0.5)/4,(2+2^0.5)/4-I/(2*2^0.5)}};
    invSSZ = ({{1,0},{0,E^(-I*Pi/4)}});

    GlobalPhase[\[Theta]_] := E^(I*\[Theta]) * IdentityMatrix[2];

    Phase[\[Theta]_] := {{1,0},{0,E^(I*\[Theta])}};

    (* Pauli exponentials X^k, Y^k, Z^k *)
    XE[k_] := (0.5 * {{1+E^(I*k*Pi), 1-E^(I*k*Pi)}, {1-E^(I*k*Pi), 1+E^(I*k*Pi)}});
    YE[k_] := (0.5 * {{1+E^(I*k*Pi), I*(E^(I*k*Pi)-1)}, {I*(1-E^(I*k*Pi)), 1+E^(I*k*Pi)}});
    ZE[k_] := ({{1, 0}, {0,E^(I*k*Pi)}});

    (* rotation *)
    RX[\[Theta]_] := {{Cos[\[Theta]/2],-I*Sin[\[Theta]/2]},{-I*Sin[\[Theta]/2],Cos[\[Theta]/2]}};
    RY[\[Theta]_] := {{Cos[\[Theta]/2],-Sin[\[Theta]/2]},{Sin[\[Theta]/2],Cos[\[Theta]/2]}};
    RZ[\[Theta]_] := {{E^(-I*\[Theta]/2),0},{0,E^(I*\[Theta]/2)}};
    (* alternative definitions:
    RXalt[\[Theta]_] := XE[\[Theta]/Pi] . GlobalPhase[-\[Theta]/2];
    RYalt[\[Theta]_] := YE[\[Theta]/Pi] . GlobalPhase[-\[Theta]/2];
    RZalt[\[Theta]_] := ZE[\[Theta]/Pi] . GlobalPhase[-\[Theta]/2];
    *)

    (* Hadamard exponential H^k *)
    HE[k_] := {
      { (2+2^0.5)/4 + (E^(I*k*Pi))*(2-2^0.5)/4 , 1/(2*2^0.5) - (E^(I*k*Pi))/(2*2^0.5) },
      { 1/(2*2^0.5) - (E^(I*k*Pi))/(2*2^0.5)   , (2-2^0.5)/4 + (E^(I*k*Pi))*(2+2^0.5)/4  }
    };

    (* generalized gates *)
    Zgeneralized[a_,b_] := {{E^(I*a),0},{0,E^(I*b)}};
    Ygeneralized[a_,b_] := {{0,E^(I*b)},{E^(I*a),0}};
    Hgeneralized[a_,b_] := ((1/2^0.5)*{{E^(I*a),E^(I*b)},{E^(I*a),-E^(I*b)}});

    (* Example computation: we test if X == HZH, by checking if the below expression yields zero *)
    X - H.Z.H // Chop

    (* Another example computation: we test if H^k == (Y^-0.25) (X^k) (Y^0.25), by checking if the below expression yields zero *)
    HE[k] - invSSY . XE[k] . SSY  // FullSimplify // Chop



