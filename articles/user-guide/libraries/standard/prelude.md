---
author: bradben
description: Learn about the intrinsic operations and functions in the QDK, including classical functions and unitary, rotation and measurement operations.
ms.author: brbenefield
ms.date: 06/13/2023
ms.service: azure-quantum
ms.subservice: qsharp-guide
ms.topic: conceptual
no-loc: ['Q#', '$$v', Quantum Development Kit, target, targets]
title: Intrinsic operations and functions in the QDK
uid: microsoft.quantum.libraries.overview.standard.prelude
---

# The Prelude

The Q# compiler and the target machines included with the Quantum Development Kit provide a set of intrinsic functions and operations that can be used when writing quantum programs in Q#.

## Intrinsic operations and functions

The intrinsic operations defined in the standard library roughly fall into one of several categories:

- Essential classical functions, collected in the [`Microsoft.Quantum.Core`](xref:Microsoft.Quantum.Core) namespace.
- Operations representing unitaries composed of [Clifford and $T$ gates](xref:microsoft.quantum.concepts.qubit).
- Operations representing rotations about various operators.
- Operations implementing measurements.

Since the Clifford + $T$ gate set is [universal](xref:microsoft.quantum.concepts.multiple-qubits) for quantum computing, these operations suffice to approximately implement any quantum algorithm within negligibly small error.
By providing rotations as well, Q# allows the programmer to work within the single qubit unitary and CNOT gate library. This library is easier to think about because it does not require the programmer to express the Clifford + $T$ decomposition directly, and because highly efficient methods exist for compiling single qubit unitaries into Clifford and $T$ gates. See [Approaches for synthesizing circuits out of H, T and CNOT gates](xref:microsoft.quantum.more-information#approaches-for-synthesizing-circuits-out-of-h-t-and-cnot-gates) for more information.

Where possible, any operation that acts on qubits allows for applying the `Controlled` variant, such that the target machine performs the appropriate decomposition.

Many of the functions and operations defined in this portion of the prelude are in the [Microsoft.Quantum.Intrinsic](xref:Microsoft.Quantum.Intrinsic) namespace, such that most Q# source files have an `open Microsoft.Quantum.Intrinsic;` directive immediately following the initial namespace declaration.
The [Microsoft.Quantum.Core](xref:Microsoft.Quantum.Core) namespace is automatically opened, so that functions such as [Length](xref:Microsoft.Quantum.Core.Length) can be used without an `open` statement at all.

### Common single-qubit unitary operations

The prelude also defines many common [single-qubit operations](xref:microsoft.quantum.concepts.qubit#single-qubit-operations).
All of these operations allow both the `Controlled` and `Adjoint` functors.

#### Pauli operators

The [X](xref:Microsoft.Quantum.Intrinsic.X) operation implements the Pauli $X$ operator.
This is sometimes also known as the `NOT` gate.
It has signature `(Qubit => Unit is Adj + Ctl)`.
It corresponds to the single-qubit unitary:

\begin{equation}
    \begin{bmatrix}
        0 & 1 \\\\ % FIXME: this currently uses the quadwhack hack.
        1 & 0
    \end{bmatrix}
\end{equation}

The [Y](xref:Microsoft.Quantum.Intrinsic.Y) operation implements the Pauli $Y$ operator.
It has signature `(Qubit => Unit is Adj + Ctl)`.
It corresponds to the single-qubit unitary:

\begin{equation}
    \begin{bmatrix}
        0 & -i \\\\ % FIXME: this currently uses the quadwhack hack.
        i & 0
    \end{bmatrix}
\end{equation}

The [Z](xref:Microsoft.Quantum.Intrinsic.Z) operation implements the Pauli $Z$ operator.
It has signature `(Qubit => Unit is Adj + Ctl)`.
It corresponds to the single-qubit unitary:

\begin{equation}
    \begin{bmatrix}
        1 & 0 \\\\ % FIXME: this currently uses the quadwhack hack.
        0 & -1
    \end{bmatrix}
\end{equation}

In the following representation, you see these transformations map to the [Bloch sphere](xref:microsoft.quantum.concepts.qubit#visualizing-qubits-and-transformations-using-the-bloch-sphere) (the rotation axis in each case is highlighted red):

![Pauli operations mapped onto the Bloch sphere](~/media/prelude_pauliBloch.png)

It is important to note that applying the same Pauli gate twice to the same qubit cancels out the operation. This is because you have now performed a full rotation of 2π (360°) over the surface to the Bloch Sphere, thus arriving back at the starting point, which brings up the following identity:

$$
X^2=Y^2=Z^2=\boldone
$$

This can be visualized on the Bloch sphere:

![XX = I](~/media/prelude_blochIdentity.png)

#### Other single-qubit Cliffords

The [H](xref:Microsoft.Quantum.Intrinsic.H) operation implements the Hadamard gate.
This interchanges the Pauli $X$ and $Z$ axes of the target qubit, such that $H\ket{0} = \ket{+} \mathrel{:=} (\ket{0} + \ket{1}) / \sqrt{2}$ and $H\ket{+} = \ket{0}$.
It has signature `(Qubit => Unit is Adj + Ctl)`,
and corresponds to the single-qubit unitary:

\begin{equation}
    \frac{1}{\sqrt{2}}
    \begin{bmatrix}
        1 & 1 \\\\ % FIXME: this currently uses the quadwhack hack.
        1 & -1
    \end{bmatrix}
\end{equation}

The Hadamard gate is particularly important as it can be used to create a superposition of the $\ket{0}$ and $\ket{1}$ states. In the Bloch sphere representation, it is easiest to think of this as a rotation of $\ket{\psi}$ around the x-axis by $\pi$ radians ($180^\circ$) followed by a (clockwise) rotation around the y-axis by $\pi/2$ radians ($90^\circ$):

![Hadamard operation mapped onto the Bloch sphere](~/media/prelude_hadamardBloch.png)

The [S](xref:Microsoft.Quantum.Intrinsic.S) operation implements the phase gate $S$, which is the matrix square root of the Pauli $Z$ operation.
That is, $S^2 = Z$.
It has signature `(Qubit => Unit is Adj + Ctl)`,
and corresponds to the single-qubit unitary:

\begin{equation}
    \begin{bmatrix}
        1 & 0 \\\\ % FIXME: this currently uses the quadwhack hack.
        0 & i
    \end{bmatrix}
\end{equation}

#### Rotations

In addition to the Pauli and Clifford operations described earlier, the Q# prelude provides various ways of expressing rotations.
As described in [single-qubit operations](xref:microsoft.quantum.concepts.qubit#single-qubit-operations), the ability to rotate is critical to quantum algorithms.

Start by recalling that you can express any single-qubit operation using the $H$ and $T$ gates, where $H$ is the Hadamard operation, and where 
\begin{equation}
    T \mathrel{:=}
    \begin{bmatrix}
        1 & 0 \\\\ % FIXME: this currently uses the quad back whack hack.
        0 & e^{i \pi / 4}
    \end{bmatrix}
\end{equation}
This is the square root of the [S](xref:Microsoft.Quantum.Intrinsic.S) operation, such that $T^2 = S$.
The $T$ gate is in turn implemented by the [T](xref:Microsoft.Quantum.Intrinsic.X) operation, and has signature `(Qubit => Unit is Adj + Ctl)`, indicating that it is a unitary operation on a single-qubit.

Even though this is in principle sufficient to describe any arbitrary single-qubit operation, different target machines may have more efficient representations for rotations about Pauli operators, such that the prelude includes various ways to conveniently express such rotations.
The most basic of these ways is the [R](xref:Microsoft.Quantum.Intrinsic.R) operation, which implements a rotation around a specified Pauli axis,
\begin{equation}
    R(\sigma, \phi) \mathrel{:=}
    \exp(-i \phi \sigma / 2),
\end{equation}
where $\sigma$ is a Pauli operator, $\phi$ is an angle, and where $\exp$ represents the matrix exponential.
It has signature `((Pauli, Double, Qubit) => Unit is Adj + Ctl)`, where the first two parts of the input represent the classical arguments $\sigma$ and $\phi$ needed to specify the unitary operator $R(\sigma, \phi)$.
You can partially apply $\sigma$ and $\phi$ to obtain an operation whose type is that of a single-qubit unitary.
For example, `R(PauliZ, PI() / 4, _)` has type `(Qubit => Unit is Adj + Ctl)`.

> [!NOTE]
> The [R](xref:Microsoft.Quantum.Intrinsic.R) operation divides the input angle by 2 and multiplies it by -1.
> For $Z$ rotations, this means that the $\ket{0}$ eigenstate is rotated by $-\phi / 2$ and the
> $\ket{1}$ eigenstate is rotated by $\phi / 2$, so that the $\ket{1}$ eigenstate is rotated by $\phi$
> relative to the $\ket{0}$ eigenstate.
>
> In particular, this means that `T` and `R(PauliZ, PI() / 8, _)` differ only by an irrelevant global phase.
> For this reason, $T$ is sometimes known as the $\frac{\pi}{8}$-gate.
>
> Note also that rotating around `PauliI` simply applies a global phase of $\phi / 2$. While such phases are irrelevant, as argued in [the conceptual documents](xref:microsoft.quantum.concepts.qubit), they are relevant for controlled `PauliI` rotations.

Within quantum algorithms, it is often useful to express rotations as dyadic fractions, such that $\phi = \pi k / 2^n$ for some $k \in \mathbb{Z}$ and $n \in \mathbb{N}$.
The [RFrac](xref:Microsoft.Quantum.Intrinsic.RFrac) operation implements a rotation around a specified Pauli axis using this convention.
It differs from the [R](xref:Microsoft.Quantum.Intrinsic.R) operation in that the rotation angle is specified as two inputs of type `Int`, interpreted as a dyadic fraction.
Thus, `RFrac` has signature `((Pauli, Int, Int, Qubit) => Unit is Adj + Ctl)`.
It implements the single-qubit unitary $\exp(i \pi k \sigma / 2^n)$, where $\sigma$ is the Pauli matrix
corresponding to the first argument, $k$ is the second argument, and $n$ is the third argument.
`RFrac(_,k,n,_)` is the same as `R(_,-πk/2^n,_)`; note that the angle is the *negative*
of the fraction.

The [Rx](xref:Microsoft.Quantum.Intrinsic.Rx) operation implements a rotation around the Pauli $X$ axis.
It has signature `((Double, Qubit) => Unit is Adj + Ctl)`.
`Rx(_, _)` is the same as `R(PauliX, _, _)`.

The [Ry](xref:Microsoft.Quantum.Intrinsic.Ry) operation implements a rotation around the Pauli $Y$ axis.
It has signature `((Double, Qubit) => Unit is Adj + Ctl)`.
`Ry(_, _)` is the same as `R(PauliY,_ , _)`.

The [Rz](xref:Microsoft.Quantum.Intrinsic.Rz) operation implements a rotation around the Pauli $Z$ axis.
It has signature `((Double, Qubit) => Unit is Adj + Ctl)`.
`Rz(_, _)` is the same as `R(PauliZ, _, _)`.

The [R1](xref:Microsoft.Quantum.Intrinsic.R1) operation implements a rotation by the given amount around $\ket{1}$, the
$-1$ eigenstate of $Z$.
It has signature `((Double, Qubit) => Unit is Adj + Ctl)`.
`R1(phi,_)` is the same as `R(PauliZ,phi,_)` followed by `R(PauliI,-phi,_)`.

The [R1Frac](xref:Microsoft.Quantum.Intrinsic.R1Frac) operation implements a fractional rotation by the given amount around the
Z=1 eigenstate.
It has signature `((Int,Int, Qubit) => Unit is Adj + Ctl)`.
`R1Frac(k,n,_)` is the same as `RFrac(PauliZ,-k.n+1,_)` followed by `RFrac(PauliI,k,n+1,_)`.

An example of a rotation operation (around the Pauli $Z$ axis, in this instance) mapped onto the Bloch sphere is shown here:

![Rotation operation mapped onto the Bloch sphere](~/media/prelude_rotationBloch.png)

#### Multi-qubit operations

In addition to the single-qubit operations described earlier, the prelude also defines several multi-qubit operations.

First, the [CNOT](xref:Microsoft.Quantum.Intrinsic.CNOT) operation performs a standard controlled-`NOT` gate,
\begin{equation}
    \operatorname{CNOT} \mathrel{:=}
    \begin{bmatrix}
        1 & 0 & 0 & 0 \\\\
        0 & 1 & 0 & 0 \\\\
        0 & 0 & 0 & 1 \\\\
        0 & 0 & 1 & 0
    \end{bmatrix}.
\end{equation}
It has signature `((Qubit, Qubit) => Unit is Adj + Ctl)`, representing that $\operatorname{CNOT}$ acts unitarily on two individual qubits.
`CNOT(q1, q2)` is the same as `(Controlled X)([q1], q2)`.
Since the `Controlled` functor allows for controlling on a register, you use the array literal `[q1]` to indicate that you want only the one control.

The [CCNOT](xref:Microsoft.Quantum.Intrinsic.CCNOT) operation performs doubly-controlled NOT gate, sometimes also known as the Toffoli gate.
It has signature `((Qubit, Qubit, Qubit) => Unit is Adj + Ctl)`.
`CCNOT(q1, q2, q3)` is the same as `(Controlled X)([q1, q2], q3)`.

The [SWAP](xref:Microsoft.Quantum.Intrinsic.SWAP) operation swaps the quantum states of two qubits.
That is, it implements the unitary matrix
\begin{equation}
    \operatorname{SWAP} \mathrel{:=}
    \begin{bmatrix}
        1 & 0 & 0 & 0 \\\\
        0 & 0 & 1 & 0 \\\\
        0 & 1 & 0 & 0 \\\\
        0 & 0 & 0 & 1
    \end{bmatrix}.
\end{equation}
It has signature `((Qubit, Qubit) => Unit is Adj + Ctl)`.
`SWAP(q1,q2)` is equivalent to `CNOT(q1, q2)` followed by `CNOT(q2, q1)` and then `CNOT(q1, q2)`.

> [!NOTE]
> The SWAP gate is *not* the same as rearranging the elements of a variable with type `Qubit[]`.
> Applying `SWAP(q1, q2)` causes a change to the state of the qubits referred to by `q1` and `q2`, while `let swappedRegister = [q2, q1];` only affects how you refer to those qubits.
> Moreover, `(Controlled SWAP)([q0], (q1, q2))` allows for `SWAP` to be conditioned on the state of a third qubit, which you cannot represent by rearranging elements.
> The controlled-SWAP gate, also known as the Fredkin gate, is powerful enough to include all classical computation.

Finally, the prelude provides two operations for representing exponentials of multi-qubit Pauli operators.
The [Exp](xref:Microsoft.Quantum.Intrinsic.Exp) operation performs a rotation based on a tensor product of Pauli matrices, as represented by the multi-qubit unitary
\begin{equation}
    \operatorname{Exp}(\vec{\sigma}, \phi) \mathrel{:=}
    \exp\left(i \phi \sigma_0 \otimes \sigma_1 \otimes \cdots \otimes \sigma_n \right),
\end{equation}
where $\vec{\sigma} = (\sigma_0, \sigma_1, \dots, \sigma_n)$ is a sequence of single-qubit Pauli operators, and where $\phi$ is an angle.
The `Exp` rotation represents $\vec{\sigma}$ as an array of `Pauli` elements, such that it has signature `((Pauli[], Double, Qubit[]) => Unit is Adj + Ctl)`.

The [ExpFrac](xref:Microsoft.Quantum.Intrinsic.ExpFrac) operation performs the same rotation, using the dyadic fraction notation discussed earlier.
It has signature `((Pauli[], Int, Int, Qubit[]) => Unit is Adj + Ctl)`.

> [!WARNING]
> Exponentials of the tensor product of Pauli operators are not the same as tensor products of the exponentials of Pauli operators.
> That is, $e^{i (Z \otimes Z) \phi} \ne e^{i Z \phi} \otimes e^{i Z \phi}$.

### Measurements

When measuring, the +1 eigenvalue of the operator being measured corresponds to a `Zero` result, and the -1 eigenvalue to a `One` result.

> [!NOTE]
> While this convention might seem odd, it has two very nice advantages.
> First, observing the outcome $\ket{0}$ is represented by the `Result` value `Zero`, while observing $\ket{1}$ corresponds to `One`.
> Second, you can write out that the eigenvalue $\lambda$ corresponding to a result $r$ is $\lambda = (-1)^r$.

Measurement operations support neither the `Adjoint` nor the `Controlled` functor.

The [Measure](xref:Microsoft.Quantum.Intrinsic.Measure) operation performs a joint measurement of one or more qubits in the specified product of Pauli operators.
If the Pauli array and qubit array are different lengths,
then the operation fails.
`Measure` has signature `((Pauli[], Qubit[]) => Result)`.

Note that a joint measurement is not the same as measuring each qubit individually.
For example, consider the state $\ket{11} = \ket{1} \otimes \ket{1} = X\otimes X \ket{00}$.
Measuring $Z_0$ and $Z_1$ each individually, you get the results $r_0 = 1$ and $r_1 = 1$.
Measuring $Z_0 Z_1$, however, you get the single result $r_{\textrm{joint}} = 0$, representing that the parity of $\ket{11}$ is positive.
Put differently, $(-1)^{r_0 + r_1} = (-1)^r_{\textrm{joint}})$.
Critically, since you *only* learn the parity from this measurement, any quantum information represented in the superposition between the two two-qubit states of positive parity, $\ket{00}$ and $\ket{11}$, is preserved.
This property is essential to quantum error correction.

For convenience, the prelude also provides two other operations for measuring qubits.
First, since performing single-qubit measurements is quite common, the prelude defines a shorthand for this case.
The [M](xref:Microsoft.Quantum.Intrinsic.M) operation measures the Pauli $Z$ operator on a single qubit, and has signature `(Qubit => Result)`.
`M(q)` is equivalent to `Measure([PauliZ], [q])`.

The [MultiM](xref:Microsoft.Quantum.Measurement.MultiM) operation measures the Pauli $Z$ operator *separately* on each of an array of qubits, returning the *array* of `Result` values obtained for each qubit.
In some cases this can be optimized. 
It has signature (`Qubit[] => Result[])`.
`MultiM(qs)` is equivalent to:

```qsharp
mutable rs = [Zero, size = Length(qs)];
for (index in 0..Length(qs)-1)
{
    set rs[index] = M(qs[index]);
}
return rs;
```

## Extension functions and operations

In addition, the prelude defines a rich set of mathematical and type conversion functions at the .NET level for use within Q# code.
For instance, the [`Microsoft.Quantum.Math`](xref:Microsoft.Quantum.Math) namespace defines useful operations such as the [Sin](xref:Microsoft.Quantum.Math.Sin) and [Log](xref:Microsoft.Quantum.Math.Log) functions.
The implementation provided by the Quantum Development Kit uses the classical .NET base class library, and thus may involve an additional communications round trip between quantum programs and their classical drivers.
While this does not present a problem for a local simulator, this can be a performance issue when using a remote simulator or actual hardware as a target machine.
That said, an individual target machine may mitigate this performance impact by overriding these operations with versions that are more efficient for that particular system.

### Math

The [`Microsoft.Quantum.Math`](xref:Microsoft.Quantum.Math) namespace provides many useful functions from the .NET base class library's [`System.Math` class](/dotnet/api/system.math?view=netframework-4.7.1&preserve-view=true).
These functions can be used in the same manner as any other Q# functions:

```qsharp
open Microsoft.Quantum.Math;
// ...
let y = Sin(theta);
```

Where a .NET static method has been overloaded based on the type of its arguments, the corresponding Q# function is annotated with a suffix indicating the type of its input:

```qsharp
let x = AbsI(-3); // x : Int = 3
let y = AbsD(-PI()); // y : Double = 3.1415...
```

### Bitwise operations

Finally, the [`Microsoft.Quantum.Bitwise`](xref:Microsoft.Quantum.Bitwise) namespace provides several useful functions for manipulating integers through bitwise operators.
For instance, the [Parity](xref:Microsoft.Quantum.Bitwise.Parity) function returns the bitwise parity of an integer as another integer.
