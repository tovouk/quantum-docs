---
author: bradben
description: Learn about common and user-defined type conversion functions in the Q# standard libraries.
ms.author: brbenefield
ms.date: 06/21/2023
ms.service: azure-quantum
ms.subservice: qsharp-guide
ms.topic: conceptual
no-loc: ['Q#', '$$v']
title: Type conversions in the Q# standard libraries
uid: microsoft.quantum.libraries.overview.convert
---

# Type conversions #

Q# is a **strongly-typed** language.
In particular, Q# does not implicitly cast between distinct types. For instance, `1 + 2.0` is not a valid Q# expression.
Rather, Q# provides a variety of type conversion functions for constructing new values of a given type.

For example, the [Length](xref:Microsoft.Quantum.Core.Length) function has an output type of `Int`, so its output must first be converted to a `Double` before it can be used as a part of a floating-point expression.
This can be done using the [IntAsDouble](xref:Microsoft.Quantum.Convert.IntAsDouble) function:

```qsharp
open Microsoft.Quantum.Convert as Convert;

function HalfLength<'T>(arr : 'T[]) : Double {
    return Convert.IntAsDouble(Length(arr)) / 2.0;
}
```

The [Microsoft.Quantum.Convert](xref:Microsoft.Quantum.Convert) namespace provides common type conversion functions for working with the basic built-in types, such as `Int`, `Double`, `BigInt`, `Result`, and `Bool`:

```qsharp
let bool = Convert.ResultAsBool(One);        // true
let big = Convert.IntAsBigInt(271);          // 271L
let indices = Convert.RangeAsIntArray(0..4); // [0, 1, 2, 3, 4]
```

The [Microsoft.Quantum.Convert](xref:Microsoft.Quantum.Convert) namespace also provides some more exotic conversions, such as `FunctionAsOperation`, which converts functions `'T -> 'U` into new operations `'T => 'U`.

Finally, the Q# standard library provides a number of user-defined types such as [Complex](xref:Microsoft.Quantum.Math.Complex) and [LittleEndian](xref:Microsoft.Quantum.Arithmetic.LittleEndian).
Along with these types, the standard library provides functions such as [BigEndianAsLittleEndian](xref:Microsoft.Quantum.Arithmetic.BigEndianAsLittleEndian):

```Q#
open Microsoft.Quantum.Arithmetic as Arithmetic;

let register = Arithmetic.BigEndian(qubits);
IncrementByInteger(Arithmetic.BigEndianAsLittleEndian(register));
```