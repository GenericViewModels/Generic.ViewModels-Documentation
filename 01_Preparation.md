# Preparations to use Generic.ViewModels with a Windows App

## Create a .NET Standard Library

1. Create a .NET Standard Library named BooksLib with models, services, and view-models

![Create a .NET Standard Library](Images/01_CreateLib.PNG)

2. Add the NuGet package to the the Generic.ViewModels Library

![Add the NuGet Package](Images/02_AddPackages.PNG)

> Generic.ViewModels is currently not available on the NuGet server. This will change as the library proceeds. Now, you can either clone the repository with the following command, and a) include the project in your solution and add a project reference, or b) create a NuGet package and configure the NuGet package manager to use your local directory.

`git clone https://github.com/GenericViewModels/Generic.ViewModels`

Create a NuGet package with the dotnet CLI:

`dotnet pack`

3. Create the directories `Models`, `Services`, and `ViewModels`.

4. Reference the Prism.Core Library

Now you're ready to [implement the Books library](02_BooksLib.md).