---
author: bradben
description: Learn how to use error correcting codes in your Q# programs while protecting the state of the qubits.
ms.author: brbenefield
ms.date: 06/21/2023
ms.service: azure-quantum
ms.subservice: qsharp-guide
ms.topic: conceptual
no-loc: ['Q#', '$$v']
title: Error correction in the Q# standard libraries
uid: microsoft.quantum.libraries.overview.error-correction
---

# Error correction

In classical computing, if one wants to protect a bit against errors, it can often suffice to represent that bit by a *logical bit* by repeating the data bit.
For instance, let $\overline{0} = 000$ be the encoding of the data bit 0, where you use the line above the label 0 to indicate that it is an encoding of a bit in the 0 state.
If you similarly let $\overline{1} = 111$, then you have a simple repetition code that protects against any one bit flip error.
That is, if any of the three bits are flipped, then you can recover the state of the logical bit by taking a majority vote.
Though classical error correction is a much richer subject that this particular example (see [Lint's introduction to coding theory](https://www.springer.com/us/book/9783540641339)), the earlier repetition code already points to a possible problem in protecting quantum information.
Namely, the [no-cloning theorem](xref:microsoft.quantum.concepts.pauli#the-no-cloning-theorem) implies that if you measure each individual qubit and take a majority vote by analogy to classical code here, then you've lost the precise information that you're trying to protect.

In the quantum setting, you see that the measurement is problematic. However, you can still implement the previous encoding.
It is helpful to do so to see how you can generalize error correction to the quantum case.
Thus, let $\ket{\overline{0}} = \ket{000} = \ket{0} \otimes \ket{0} \otimes \ket{0}$, and let $\ket{\overline{1}} = \ket{111}$.
Then, by linearity, you have defined your repetition code for all inputs; for instance, $\ket{\overline{+}} = (\ket{\overline{0}} + \ket{\overline{1}}) / \sqrt{2} = (\ket{000} + \ket{111}) / \sqrt{2}$.
In particular, letting a bit-flip error $X_1$ act on the middle qubit, you see that the correction needed in both branches is precisely $X_1$:
$$
\begin{align}
    X_1 \ket{\overline{+}} & = \frac{1}{\sqrt{2}} \left(
        X_1 \ket{000} + X_1 \ket{111}
    \right) \\\\
    & = \frac{1}{\sqrt{2}} \left(
        \ket{010} + \ket{101}
    \right).
\end{align}
$$

To see how you can identify that this is the case without measuring the very state you are trying to protect, it is helpful to write down what each different bit flip error does to the logical states:

| Error $E$ | $E\ket{\overline{0}}$ | $E\ket{\overline{1}}$ |
| --- | --- | --- |
| $\mathbf{1}$ | $\ket{000}$ | $\ket{111}$ |
| $X_0$ | $\ket{100}$ | $\ket{011}$ |
| $X_1$ | $\ket{010}$ | $\ket{101}$ |
| $X_2$ | $\ket{001}$ | $\ket{110}$ |

In order to protect the state that you're encoding, you need to be able to distinguish the three errors from each other and from the identity $\mathbf{1}$ without distinguishing between $\ket{\overline{0}}$ and $\ket{\overline{1}}$.
For example, if you measure $Z_0$, you get a different result for $\ket{\overline{0}}$ and $\ket{\overline{1}}$ in the no-error case, so that collapses the encoded state.
On the other hand, consider measuring $Z_0 Z_1$, the parity of the first two bits in each computational basis state.
Recall that each measurement of a Pauli operator checks which eigenvalue  the state being measured corresponds to, so for each state $\ket{\psi}$ in the previous table, you can compute $Z_0 Z_1 \ket{\psi}$ to see if you get $\pm\ket{\psi}$.
Note that $Z_0 Z_1 \ket{000} = \ket{000}$ and that $Z_0 Z_1 \ket{111} = \ket{111}$, so that you can conclude that this measurement does the same thing to both encoded states.
On the other hand, $Z_0 Z_1 \ket{100} = - \ket{100}$ and $Z_0 Z_1 \ket{011} = -\ket{011}$, so the result of measuring $Z_0 Z_1$ reveals useful information about which error occurred.

To emphasize this, repeat the previous table, but add the results of measuring $Z_0 Z_1$ and $Z_1 Z_2$ on each row.
You denote the results of each measurement by the sign of the eigenvalue that is observed, either $+$ or $-$, corresponding to the Q# `Result` values of `Zero` and `One`, respectively.

| Error $E$ | $E\ket{\overline{0}}$ | $E\ket{\overline{1}}$ | Result of $Z_0 Z_1$ | Result of $Z_1 Z_2$ |
| --- | --- | --- | --- | --- |
| $\mathbf{1}$ | $\ket{000}$ | $\ket{111}$ | $+$ | $+$ |
| $X_0$ | $\ket{100}$ | $\ket{011}$ | $-$ | $+$ |
| $X_1$ | $\ket{010}$ | $\ket{101}$ | $-$ | $-$ |
| $X_2$ | $\ket{001}$ | $\ket{110}$ | $+$ | $-$ |

Thus, the results of the two measurements uniquely determines which bit-flip error occurred, but without revealing any information about which state you encoded.
These results a called *syndrome*, and the process of mapping a syndrome back to the error that caused it is called *recovery*.
In particular, recovery is a *classical* inference procedure which takes as its input the syndrome which occurred, and returns a prescription for how to fix any errors that may have occurred.

> [!NOTE]
> The bit-flip code shown here can only correct against single bit-flip errors; that is, an `X` operation acting on a single qubit.
> Applying `X` to more than one qubit maps $\ket{\overline{0}}$ to $\ket{\overline{1}}$ following recovery.
> Similarly, applying a phase flip operation `Z` maps $\ket{\overline{1}}$ to $-\ket{\overline{1}}$, and hence maps $\ket{\overline{+}}$ to $\ket{\overline{-}}$.
> More generally, codes can be created to handle larger number of errors, and to handle $Z$ errors as well as $X$ errors.

The insight that you can describe measurements in quantum error correction that act the same way on all code states, is the essence of the *stabilizer formalism*.
Q# provides a framework for describing encoding into and decoding from stabilizer codes, and for describing how one recovers from errors.
This section describes this framework and its application to a few simple quantum error-correcting codes.

> [!TIP]
> A full introduction to the stabilizer formalism is beyond the scope of this section.
> To learn more, see [Gottesman 2009](https://arxiv.org/abs/0904.2557).

## Representing error correcting codes in Q\#

To help specify error correcting codes, the Q# libraries provide several distinct user-defined types:

- [LogicalRegister](xref:Microsoft.Quantum.ErrorCorrection.LogicalRegister) `= Qubit[]`: Denotes that a register of qubits should be interpreted as the code block of an error-correcting code.
- [Syndrome](xref:Microsoft.Quantum.ErrorCorrection.Syndrome) `= Result[]`: Denotes that an array of measurement results should be interpreted as the syndrome measured on a code block.
- [RecoveryFn](xref:Microsoft.Quantum.ErrorCorrection.RecoveryFn) `= (Syndrome -> Pauli[])`: Denotes that a *classical* function should be used to interpret a syndrome and return a correction that should be applied.
- [EncodeOp](xref:Microsoft.Quantum.ErrorCorrection.EncodeOp) `= ((Qubit[], Qubit[]) => LogicalRegister)`: Denotes that an operation takes qubits representing data along with fresh auxiliary qubits in order to produce a code block of an error-correcting code.
- [DecodeOp](xref:Microsoft.Quantum.ErrorCorrection.DecodeOp) `= (LogicalRegister => (Qubit[], Qubit[]))`: Denotes than an operation decomposes a code block of an error correcting code into the data qubits and the auxiliary qubits used to represent syndrome information.
- [SyndromeMeasOp](xref:Microsoft.Quantum.ErrorCorrection.SyndromeMeasOp) `= (LogicalRegister => Syndrome)`: Denotes an operation that should be used to extract syndrome information from a code block, without disturbing the state protected by the code.

Finally, Q# provides the xref:Microsoft.Quantum.ErrorCorrection.QECC) to collect the other types required to define a quantum error-correcting code. Associated with each stabilizer quantum code is the code length $n$, the number $k$ of logical qubits, and the minimum distance $d$, often conveniently grouped together in the notation ⟦$n$, $k$, $d$⟧. For example, the [BitFlipCode](xref:Microsoft.Quantum.ErrorCorrection.BitFlipCode) function defines the ⟦3, 1, 1⟧ bit flip code:
Finally, Q# provides the [QECC](xref:Microsoft.Quantum.ErrorCorrection.QECC) to collect the other types required to define a quantum error-correcting code. Associated with each stabilizer quantum code is the code length $n$, the number $k$ of logical qubits, and the minimum distance $d$, often conveniently grouped together in the notation ⟦$n$, $k$, $d$⟧. For example, the [BitFlipCode](xref:Microsoft.Quantum.ErrorCorrection.BitFlipCode) function defines the ⟦3, 1, 1⟧ bit flip code:

```qsharp
let encodeOp = EncodeOp(BitFlipEncoder);
let decodeOp = DecodeOp(BitFlipDecoder);
let syndMeasOp = SyndromeMeasOp(MeasureStabilizerGenerators([
    [PauliZ, PauliZ, PauliI],
    [PauliI, PauliZ, PauliZ]
], _, MeasureWithScratch));
let code = QECC(encodeOp, decodeOp, syndMeasOp);
```

Notice that the `QECC` type does *not* include a recovery function.
This allows you to change the recovery function that is used in correcting errors without changing the definition of the code itself; this ability is in particular useful when incorporating feedback from characterization measurements into the model assumed by recovery.

Once a code is defined in this way, you can use the [Recover](xref:Microsoft.Quantum.ErrorCorrection.Recover) operation to recover from errors:

```qsharp
let code = BitFlipCode();
let fn = BitFlipRecoveryFn();
let X0 = ApplyPauli([PauliX, PauliI, PauliI], _);
use scratch = Qubit[nScratch];
let logicalRegister = encode(data, scratch);
// Cause an error.
X0(logicalRegister);
Recover(code, fn, logicalRegister);
let (decodedData, decodedScratch) = decode(logicalRegister);
ApplyToEach(Reset, decodedScratch);

```
Aside from the bit-flip code, the Q# libraries are provided with implementations of the [five-qubit perfect code](https://arxiv.org/abs/quant-ph/9602019), and the [seven-qubit code](https://arxiv.org/abs/quant-ph/9705052), both of which can correct an arbitrary single-qubit error.
