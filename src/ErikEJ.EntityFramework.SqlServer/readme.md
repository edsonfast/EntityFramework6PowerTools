# Entity Framework 6 SQL Server provider based on Microsoft.Data.SqlClient

This Entity Framework 6 provider is a runtime replacement provider for the built-in SQL Server provider. 

This provider depends on the modern Microsoft.Data.SqlClient ADO.NET provider, see my [blog post here](https://erikej.github.io/ef6/sqlserver/2021/08/08/ef6-microsoft-data-sqlclient.html) for why that can be desirable. One of the main reasons is the additional Azure Active Directory authentication methods [that are now available.](https://learn.microsoft.com/sql/connect/ado-net/sql/azure-active-directory-authentication)

The latest build of this package is available from [NuGet](https://www.nuget.org/packages/ErikEJ.EntityFramework.SqlServer/)

## Configuration

There are various ways to configure Entity Framework to use this provider.

You can register the provider in code using an attribute:

````csharp
    [DbConfigurationType(typeof(MicrosoftSqlDbConfiguration))]
    public class SchoolContext : DbContext
    {
        public SchoolContext() : base()
        {
        }

        public DbSet<Student> Students { get; set; }
    }
````
If you have multiple classes inheriting from DbContext in your solution, add the DbConfigurationType attribute to all of them.

Or you can use the SetConfiguration method before any data access calls:
````csharp
 DbConfiguration.SetConfiguration(new MicrosoftSqlDbConfiguration());
````
Or you can add the following lines to your existing DbConfiguration class:
````csharp
SetProviderFactory(MicrosoftSqlProviderServices.ProviderInvariantName, Microsoft.Data.SqlClient.SqlClientFactory.Instance);
SetProviderServices(MicrosoftSqlProviderServices.ProviderInvariantName, MicrosoftSqlProviderServices.Instance);
// Optional
SetExecutionStrategy(MicrosoftSqlProviderServices.ProviderInvariantName, () => new MicrosoftSqlAzureExecutionStrategy());
````
You can also use XML/App.Config based configuration:

````xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <configSections>
        <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />    
    </configSections>
    <entityFramework>
        <providers>		
            <provider invariantName="Microsoft.Data.SqlClient" type="System.Data.Entity.SqlServer.MicrosoftSqlProviderServices, ErikEJ.EntityFramework.SqlServer" />
        </providers>
    </entityFramework>
    <system.data>
        <DbProviderFactories>
           <add name="SqlClient Data Provider"
             invariant="Microsoft.Data.SqlClient"
             description=".NET Framework Data Provider for SqlServer"
             type="Microsoft.Data.SqlClient.SqlClientFactory, Microsoft.Data.SqlClient" />
        </DbProviderFactories>
    </system.data>
</configuration>
````
If you use App.Config with a .NET Core / .NET 5 or later app, you must remove the `<system.data>` section above and register the DbProviderFactory in code once:

````csharp
DbProviderFactories.RegisterFactory(MicrosoftSqlProviderServices.ProviderInvariantName, Microsoft.Data.SqlClient.SqlClientFactory.Instance);
````

## EDMX usage

If you use an EDMX file, update the `Provider` name in the EDMX file:

````xml
<edmx:Edmx Version="3.0" xmlns:edmx="http://schemas.microsoft.com/ado/2009/11/edmx">
  <edmx:Runtime>
    <edmx:StorageModels>
      <Schema Namespace="ChinookModel.Store" Provider="Microsoft.Data.SqlClient" >
````

Also update the provider name inside the EntityConnection connection string - sample of working App.Config file:

````xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
    <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
  </configSections>
  <entityFramework>
    <providers>
      <provider invariantName="Microsoft.Data.SqlClient" type="System.Data.Entity.SqlServer.MicrosoftSqlProviderServices, ErikEJ.EntityFramework.SqlServer" />
    </providers>
  </entityFramework>
  <system.data>
    <DbProviderFactories>
      <add name="SqlClient Data Provider" invariant="Microsoft.Data.SqlClient" description=".NET Framework Data Provider for SqlServer" type="Microsoft.Data.SqlClient.SqlClientFactory, Microsoft.Data.SqlClient" />
    </DbProviderFactories>
  </system.data>
  <connectionStrings>
    <add name="ChinookEntities" connectionString="metadata=res://*/Model1.csdl|res://*/Model1.ssdl|res://*/Model1.msl;provider=Microsoft.Data.SqlClient;provider connection string=&quot;data source=.\SQLEXPRESS;initial catalog=Chinook;integrated security=True;App=EntityFramework&quot;" providerName="System.Data.EntityClient" />
  </connectionStrings>
</configuration>
````

> In order to use the EDMX file with the Visual Studio designer, use Visual Studio 17.6 or later. Your project must be a "classic" .NET Framework project. Make sure to restore packages and build before opening the designer. You may need to restart Visual Studio after building for the first time. For designer issues, report [here](https://developercommunity.visualstudio.com/). An interesting hack for some designer issues [is described here](https://github.com/dotnet/SqlClient/issues/2020#issuecomment-1528907830).


## Code changes

In order to use the provider in an existing solution, a few code changes are required (as needed).

`using System.Data.SqlClient;` => `using Microsoft.Data.SqlClient;`

The following classes have been renamed to avoid conflicts with classes in the existing SQL Server provider:

`SqlAzureExecutionStrategy` => `MicrosoftSqlAzureExecutionStrategy`

`SqlConnectionFactory` => `MicrosoftSqlConnectionFactory`

`SqlDbConfiguration` => `MicrosoftSqlDbConfiguration`

`SqlProviderServices` => `MicrosoftSqlProviderServices`

`SqlServerMigrationSqlGenerator` => `MicrosoftSqlServerMigrationSqlGenerator`

`SqlSpatialServices` => `MicrosoftSqlSpatialServices`

## Known issues

**EntityFramework.dll installed in GAC**

If an older version of EntityFramework.dll is installed in the .NET Framework GAC (Global Assembly Cache), you might get this runtime error:

`The 'PrimitiveTypeKind' attribute is invalid - The value 'HierarchyId' is invalid according to its datatype`

Solution is to remove the .dll from the GAC, see [this for more info](https://github.com/ErikEJ/EntityFramework6PowerTools/issues/93#issuecomment-1063269072)

## Feedback

Please report any issues, questions and suggestions [here](https://github.com/ErikEJ/EntityFramework6PowerTools/issues)

## Release notes

### 6.6.6

- Fix for .NET resource load bug

### 6.6.5

- Uses Microsoft.Data.SqlClient 5.1.1

### 6.6.4

- Fix wrong resource reference (to legacy provider assembly)

### 6.6.3

- Uses Microsoft.Data.SqlClient 5.1.0

### 6.6.2

- Uses Microsoft.SqlServer.Types 160.1000.6
- Requires .NET Framework 4.6.2

### 6.6.1

- Uses Microsoft.Data.SqlClient 5.0.1

### 6.6.0

- Uses Microsoft.Data.SqlClient 5.0.0
- Added support for spatial on .NET (Core) with new Microsoft.SqlServer.Types package

### 6.5.0

- Uses Microsoft.Data.SqlClient 4.0.1
- Removed duplicate spatial and Sql functions classes

### 6.4.x

- Uses Microsoft.Data.SqlClient 2.1.4
