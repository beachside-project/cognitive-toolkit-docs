---
title:   NuGet Package
author:    chrisbasoglu
ms.author:   cbasoglu
ms.date:   06/22/2017
ms.custom:   cognitive-toolkit
ms.topic:   conceptual
ms.service:  Cognitive-services
ms.devlang:   csharp, cpp
---

# NuGet Package

## Overview

The CNTK Eval NuGet package is a NuGet package containing the necessary libraries and assemblies to enable .NET and Windows C++ applications to perform CNTK model evaluation. There are 3 NuGet packages:

* **CNTK.CPUOnly**: provides [CNTK C#/.NET Managed Eval Library](./CNTK-Library-Managed-API.md) and [C++ Library](./CNTK-Library-Native-Eval-Interface.md) for CPU only machines.
* **CNTK.GPU**: provides [CNTK C#/.NET Managed Eval Library](./CNTK-Library-Managed-API.md) and [C++ Library](./CNTK-Library-Native-Eval-Interface.md) for GPU enabled machines.

## Installation
The package may be obtained through the NuGet Package Manager inside Visual Studio by searching for "CNTK", or downloaded directly from NuGet:

* [https://www.nuget.org/packages/CNTK.CPUOnly/2.0.0](https://www.nuget.org/packages/CNTK.CPUOnly/2.0.0)   
* [https://www.nuget.org/packages/CNTK.GPU/2.0.0](https://www.nuget.org/packages/CNTK.GPU/2.0.0)

The current version is `2.0.0`.

The CNTK NuGet packages may be installed on a Visual C++ or .NET project (C#, VB.Net, F#, ...). The NuGet package contains both debug and release versions of C++ library and DLLs, and the release version of C# assembly and its dependent DLLs. Once installed the project will contain a reference to the managed DLL and the required dependent binary libraries will be copied to the output directory after building the project. 

For instructions on how to install a NuGet package, please refer to the NuGet documentation at:
[https://docs.nuget.org/consume/installing-nuget](https://docs.nuget.org/consume/installing-nuget)

## Current Release
The current release of CNTK Eval NuGet Packages support the following interfaces
* [CNTK Library Managed Eval Interface](./CNTK-Library-Managed-API.md)
* [CNTK Library C++ Eval Interface](./CNTK-Library-Native-Eval-Interface.md)

## Linux
There is a Linux equivalent set of libraries (albeit not available through NuGet) that enables CNTK model evaluations in Linux using C++. Refer to the [CNTK Evaluation on Linux](./CNTK-Library-Evaluation-on-Linux.md) page for details.

## Legacy applications using CNTK EvalDLL interface
For applications that are still using CNTK EvalDLL interface, which only supports the [model-v1 format](./CNTK-model-format.md), please use  the **Microsoft.Research.CNTK.CpuEval-mkl** NuGet package:
* [https://www.nuget.org/packages/Microsoft.Research.CNTK.CpuEval-mkl/2.0.0](https://www.nuget.org/packages/Microsoft.Research.CNTK.CpuEval-mkl/2.0.0): supports CPU Only, implements [EvalDll C# Interface](./EvalDll-Managed-API.md) and [EvalDll C++ Interface](./EvalDll-Native-API.md).