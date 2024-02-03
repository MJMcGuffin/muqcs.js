# <a href="https://mjmcguffin.github.io/muqcs.js/">muqcs.js</a>

Muqcs (pronounced mucks) is McGuffin's Useless Quantum Circuit Simulator
(named in an allusion to mush, Moser's Useless SHell).  It is written in JavaScript, and allows one to simulate circuits programmatically or from a command line.  It has no graphical front end, does not leverage the GPU for computations, and does not import any special libraries, making it much easier for others to understand the core algorithms.  On many personal computers, it can simulate circuits of 20+ qubits, if no explicit matrices are used as part of the simulation (search for 'second approach' below).

The code is contained entirely in a single file, and defines a small class for complex numbers, a class for complex matrices (i.e., matrices storing complex numbers), and a few utility classes.  These classes take up a bit more than a thousand lines of code.  The rest of the code consists of a regression test (in the function performRegressionTest()) followed by some performance tests.  Having a relatively small amount of source code means that the code can be more easily understood by others.

To run the code, load the html file into a browser like Chrome, and then open a console (in Chrome, this is done by selecting 'Developer Tools').  From the console prompt, you can call functions in the code and see output printed to the console.

**Creating and Manipulating Matrices**

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
    console.log(StringUtil.concatenateMultilineStrings(m1.toString()," + ",m2.toString()," = ",m3.toString()));

which produces this output:

    [10,20] + [1i  ,2+3i ] = [10+1i,22+3i]
    [30,40]   [5+7i,-1-3i]   [35+7i,39-3i]

Similarly, there are static methods in the CMatrix class for subtracting matrices (diff(m1,m2)), multiplying matrices (mult(m1,m2) and naryMult([m1,m2,...])), and for computing their tensor product (tensor(m1,m2) and naryTensor([m1,m2,...])).
The CMatrix class also has some predefined vectors and matrices.  For example,

    console.log(CMatrix.ketOne.toString());

prints the column vector |1>:

    [_]
    [1]

and

    console.log(CMatrix.gate4x4cnot.toString());

prints the 4x4 matrix for the CNOT gate:

    [1,_,_,_]
    [_,_,_,1]
    [_,_,1,_]
    [_,1,_,_]

Notice that zeros are replaced with underscores to make it easier for a human to read sparse matrices (to change this behavior, you can call toString({suppressZeros:false}) or toString({charToReplaceSuppressedZero:'.'})).  You might also notice that the matrix for the CNOT gate looks different from the way it is usually presented in textbooks or other sources.  This is related to the ordering of bits and ordering of tensor products.  Search for the usingTextbookConvention flag in the source code for comments that explain this, and set that flag to true if you prefer the textbook ordering.  We can also call a method on a matrix (or a vector) to change its ordering:

    console.log(CMatrix.gate4x4cnot.reverseEndianness().toString());

prints the 4x4 matrix for the CNOT gate in its more usual form:

    [1,_,_,_]
    [_,1,_,_]
    [_,_,_,1]
    [_,_,1,_]

**Simulating a Quantum Circuit**

To simulate a circuit, there are two approaches.  The first involves storing one explicit matrix for each stage of the circuit.  In a circuit with N qubits, the matrices will have size 2^N x 2^N.  Here we see an example of how to simulate a 3-qubit circuit with this first approach:

    // Simulate a circuit on three qubits
    // equivalent to
    //     https://algassert.com/quirk#circuit={%22cols%22:[[%22X^%C2%BC%22,%22Y^%C2%BC%22,%22H%22],[1,%22X^%C2%BC%22],[1,%22X%22,%22%E2%80%A2%22]]}
    //
    // qubit q0 |0>----(x^0.25)-------------------------
    //
    // qubit q1 |0>----(y^0.25)-----(x^0.25)----(+)-----
    //                                           |
    // qubit q2 |0>-------H----------------------o------
    //
    input = CMatrix.naryTensor( [ CMatrix.ketZero /*q2*/, CMatrix.ketZero /*q1*/, CMatrix.ketZero /*q0*/ ] );
    step1 = CMatrix.naryTensor( [ CMatrix.gate2x2hadamard /*q2*/, CMatrix.gate2x2fourthrooty /*q1*/, CMatrix.gate2x2fourthrootx /*q0*/ ] );
    step2 = CMatrix.naryTensor( [ CMatrix.gate2x2identity /*q2*/, CMatrix.gate2x2fourthrootx /*q1*/, CMatrix.gate2x2identity /*q0*/ ] );
    step3 = CMatrix.expand4x4ForNWires( CMatrix.gate4x4cnot, 2, 1, 3 );
    output = CMatrix.naryMult([ step3, step2, step1, input ]);
    console.log(StringUtil.concatenateMultilineStrings(
        step3.toString(),
        " * ", step2.toString({decimalPrecision:1}),
        " * ", "...", // step1.toString({decimalPrecision:1}),
        " * ", input.toString(),
        " = ", output.toString({binaryPrefixes:true})
    ));

Each matrix takes up O((2^N)^2) space (half a gigabyte for N=13 qubits, assuming 4 bytes per float), and calling CMatrix.mult() on two such matrices would cost O((2^N)^3) time.
However, the call to naryMult() above causes the matrices to be multiplied right-to-left, because naryMult() checks the sizes of the matrices to optimize the multiplication order, and the right-most matrix passed to naryMult() is just a column vector of size 2^N x 1.  The matrix just before that has size 2^N x 2^N, and multiplying the two together costs O((2^N)^2) and produces another column vector, which gets multiplied by the next matrix before them, etc. 
Hence, in this first approach, the space and time requirements of each step of the circuit are O((2^N)^2).
Notice in the last call to toString() above, we pass in {binaryPrefixes:true}; this causes bit strings like |000> to be printed in front of the matrix, as a reminder of the association between base states and matrix rows.
The output is:

    [1,_,_,_,_,_,_,_]   [0.9+0.4i,0       ,0.1-0.4i,0       ,0       ,0       ,0       ,0       ]         [1]   |000>[0.302+0.479i]
    [_,1,_,_,_,_,_,_]   [0       ,0.9+0.4i,0       ,0.1-0.4i,0       ,0       ,0       ,0       ]         [_]   |001>[0.198-0.125i]
    [_,_,1,_,_,_,_,_]   [0.1-0.4i,0       ,0.9+0.4i,0       ,0       ,0       ,0       ,0       ]         [_]   |010>[0.302+0.125i]
    [_,_,_,1,_,_,_,_] * [0       ,0.1-0.4i,0       ,0.9+0.4i,0       ,0       ,0       ,0       ] * ... * [_] = |011>[0.052-0.125i]
    [_,_,_,_,_,_,1,_]   [0       ,0       ,0       ,0       ,0.9+0.4i,0       ,0.1-0.4i,0       ]         [_]   |100>[0.302+0.125i]
    [_,_,_,_,_,_,_,1]   [0       ,0       ,0       ,0       ,0       ,0.9+0.4i,0       ,0.1-0.4i]         [_]   |101>[0.052-0.125i]
    [_,_,_,_,1,_,_,_]   [0       ,0       ,0       ,0       ,0.1-0.4i,0       ,0.9+0.4i,0       ]         [_]   |110>[0.302+0.479i]
    [_,_,_,_,_,1,_,_]   [0       ,0       ,0       ,0       ,0       ,0.1-0.4i,0       ,0.9+0.4i]         [_]   |111>[0.198-0.125i]

A second approach to simulating the same circuit is to not store any explicit matrices of size 2^N x 2^N.  Instead, we only store the state vector of size 2^N x 1, and update it for each stage of the circuit.  The following code does this:

    input = CMatrix.naryTensor( [ CMatrix.ketZero /*q2*/, CMatrix.ketZero /*q1*/, CMatrix.ketZero /*q0*/ ] );
    step1 = CMatrix.transformStateVectorWith2x2(CMatrix.gate2x2hadamard,2,3,input,[]);
    step1 = CMatrix.transformStateVectorWith2x2(CMatrix.gate2x2fourthrooty,1,3,step1,[]);
    step1 = CMatrix.transformStateVectorWith2x2(CMatrix.gate2x2fourthrootx,0,3,step1,[]);
    step2 = CMatrix.transformStateVectorWith2x2(CMatrix.gate2x2fourthrootx,1,3,step1,[]);
    output = CMatrix.transformStateVectorWith2x2(CMatrix.gate2x2not,1,3,step2,[[2,true]]);
    console.log(StringUtil.concatenateMultilineStrings(
        input.toString(),
        " -> ", step1.toString(),
        " -> ", step2.toString(),
        " -> ", output.toString({binaryPrefixes:true})
    ));

In this second approach, the space and time requirements of each step of the circuit are O(2^N), so, much better than in the first approach.
The magic happens in the CMatrix.transformStateVectorWith2x2() method, which is based on Quirk’s source code https://github.com/Strilanc/Quirk/ , in particular, Quirk's applyToStateVectorAtQubitWithControls() method in src/math/Matrix.js (<a href="https://github.com/Strilanc/Quirk/blob/master/src/math/Matrix.js#L678">link to specific line</a>)

More explanation and code examples appear in the slides under the doc folder of the repository.

**Circuit and Qubit Statistics**

From the amplitudes output by muqcs, we can easily find the probability of each computational basis state.
In addition, muqcs can compute the (2^N x 2^N) density matrix for a give state vector,
and also compute the (2x2) reduced density matrix (using the partial trace) for each qubit, from which we can compute
the phase, Bloch sphere coordinates, and purity (also called 'reduced purity' or 'purity of reduced state') for each qubit.
The Bloch sphere coordinates are a way to describe the qubit's 'local state'.
Purity is a quantity varying from 0.5 to 1.0, indicating how entangled the qubit is with the rest of the system:
0.5 means maximally entangled, 1.0 means not entangled, and an intermediate value means partially mixed.
Here is an example computing these statistics with muqcs:

    let N = 4; // total qubits
    input = CMatrix.naryTensor( [ CMatrix.ketZero /*q3*/, CMatrix.ketZero /*q2*/,
                                  CMatrix.ketZero /*q1*/, CMatrix.ketZero /*q0*/ ] );
    step1 = CMatrix.naryTensor( [ CMatrix.gate2x2ry(45) /*q3*/, CMatrix.gate2x2rx90degrees /*q2*/,
                                  CMatrix.gate2x2rx90degrees /*q1*/, CMatrix.gate2x2rx(45) /*q0*/ ] );
    step2 = CMatrix.naryTensor( [ CMatrix.gate2x2rx(45) /*q3*/, CMatrix.gate2x2rz(120) /*q2*/,
                                  CMatrix.gate2x2rz(100) /*q1*/, CMatrix.gate2x2identity /*q0*/ ] );
    output = CMatrix.naryMult([ step2, step1, input ]);
    output = CMatrix.transformStateVectorWith2x2(CMatrix.gate2x2rz90degrees,2,N,output,[[1,true]]/*list of control qubits*/);
    output = CMatrix.transformStateVectorWith2x2(CMatrix.gate2x2ry(45),2,N,output,[[3,true]]/*list of control qubits*/);
    baseStateProbabilities = new CMatrix( output._rows, 1 );
    for ( let i=0; i < output._rows; ++i ) baseStateProbabilities.set( i, 0, output.get(i,0).mag()**2 );
    console.log(StringUtil.concatenateMultilineStrings(
        "Output: ", output.toString({binaryPrefixes:true}), ", Probabilities: ", baseStateProbabilities.toString()
    ));
    CMatrix.printAnalysisOfEachQubit(N,output);

![Qubit statistics in Muqcs](/doc/qubit-stats-muqcs-1.png)

![More qubit statistics in Muqcs](/doc/qubit-stats-muqcs-2.png)

... and now the same circuit in <a href="https://quantum-computing.ibm.com/composer">IBM Quantum Composer</a>:

    // Copy-paste the below instructions into IBM’s website at https://quantum-computing.ibm.com/composer to recreate the circuit

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


**Conventions**

In a circuit with N qubits, the wires are numbered 0 for the top wire to (N-1) for the bottom wire.  The top wire encodes the Least-Significant Bit (LSB).

**Limitations**

There is currently no support for controlled swap gates.

**Under Construction**

Matrices encoding the effect of a quantum gate:

| Commonly used names | Muqcs code | Number of input qubits | Size of matrix | Notes |
| --- | --- | :---: | :---: | --- |
| zero, 0       | `Sim.ZERO` | 1 | 2x2 | not unitary  |
| identity, I   | `Sim.I`    | 1 | 2x2 | no-op, hermitian |
| Hadamard, H   | `Sim.H`    | 1 | 2x2 |  |
| Pauli X, NOT  | `Sim.X`    | 1 | 2x2 | bit flip |
| Pauli Y       | `Sim.Y`    | 1 | 2x2 |  |
| Pauli Z, Phase($\pi$)       | `Sim.Z`, `Phase(180)`    | 1 | 2x2 | phase flip |
| $\sqrt{X}$, SX, $\sqrt{NOT}$, V | `Sim.SX`   | 1 | 2x2 |  |
| $\sqrt{Y}$, SY | `Sim.SY`   | 1 | 2x2 |  |
| $\sqrt{Z}$, SZ, Phase($\pi/2$), S | `Sim.SZ`, `Phase(90)`   | 1 | 2x2 |  |
| $\sqrt[4]{X}$,  | `Sim.SSX`   | 1 | 2x2 |  |
| $\sqrt[4]{Y}$,  | `Sim.SSY`   | 1 | 2x2 |  |
| $\sqrt[4]{Z}$, Phase($\pi/4$), T, $\pi/8$ | `Sim.SSZ`, `Phase(45)`  | 1 | 2x2 |  |
| global phase shift    | `Sim.GlobalPhsae(angleInDegrees)` |  1 | 2x2 | can be placed on any qubit, causes a phase shift in all qubits |
| phase shift | `Sim.Phase(angleInDegrees)` | 1 | 2x2 |  |
| $R_x$       | `Sim.RX(angleInDegrees)` | 1 | 2x2 |  |
| $R_y$       | `Sim.RY(angleInDegrees)` | 1 | 2x2 |  |
| $R_z$       | `Sim.RZ(angleInDegrees)` | 1 | 2x2 |  |
|             | `Sim.RotFreeAxis( ax, ay, az )` | 1 | 2x2 |  |
|             | `Sim.RotFreeAxisAngle( ax, ay, az, angleInDegrees )` | 1 | 2x2 |  |
|             | `Sim.SWAP_2`      | 2 | 4x4         |  |
|             | `Sim.SWAP(i,j,n)` | 2 | (2^n)x(2^n) |  |
| CX, CNOT, XOR | `Sim.CX`     | 2 | 4x4 |   |


Testing a matrix:

$$\begin{bmatrix}
2 & 3\\
b & c
\end{bmatrix}$$

$$\begin{bmatrix}
1 & 2 & 3\\
a & b & c
\end{bmatrix}$$

$$a = (\begin{matrix}1 & 1\\ 0 & 1\end{matrix})$$ 

$\sqrt[4]{X}$
