# Testing Net7 and the CSharp Notebook 

#### Ceate a new solution

```
mkdir Algorithm
cd Algorithm
dotnet new sln
``` 
#### Add a class library
```
dotnet new classlib -o AlgorithmLibrary
dotnet sln add .\AlgorithmLibrary\AlgorithmLibrary.csproj
```
#### Add a test console
```
dotnet new console -o TestConsole
dotnet sln add .\TestConsole\TestConsole.csproj
```
