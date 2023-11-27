# muqcs
Mucqs (pronounced mucks) is McGuffin's Useless Quantum Circuit Simulator
(named in an allusion to mush, Moser's Useless Shell).  It is written in JavaScript, and allows one to simulate circuits programmatically or from a command line.

The code is contained entirely in a single file, and defines a small class for complex numbers, a class for complex matrices (i.e., matrices storing complex numbers), and a few utility classes.

To run the code, load the html file into a browser like Chrome, and then open a console (in Chrome, this is done by selecting 'Developer Tools').  From the console prompt, you can call functions in the code and see output printed to the console.

For example, to create a matrix and print out its contents, you can do

  let m1 = CMatrix.create([[1,2],[3,4]]);
  console.log(m1.toString());

which produces this output:

  [1,2]
  [3,4]

Notice that the toString() method returns a string containing newline characters.  In the source code, this is referred to as a 'multiline string'.
Next, we create a second matrix containing complex numbers, add the two matrices together, and print out the matrices and their sum:

  let m2 = CMatrix.create([[new Complex(0,1),new Complex(2,3)],[new Complex(5,7),new Complex(-1,-3)]]);
  let m3 = CMatrix.sum(m1,m2);
  console.log(StringUtil.concatenateMultilineStrings(m1.toString()," + ",m2.toString()," = ",m3.toString()));

which produces this output:

  [1,2] + [1i  ,2+3i ] = [1+1i,4+3i]
  [3,4]   [5+7i,-1-3i]   [8+7i,3-3i]

