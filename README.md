# MEF Extensions

The purpose of this library is to extend the functionality
of MEF to support its use as an IOC container.

## Child Containers

This allows you to create derived containers from a "parent
container" with the option of "inheriting" from the parent
container if the export doesn't exist in the child container.

Usage:

    CompositionContainer parentContainer = // your container
    var inheritingChildContainer = 
      parentContainer.CreateChildContainer("ChildContainerName");

    // you can also disable inheritance with the optional parameter.
    var noninheritingChildContainer = 
      parentContainer.CreateChildContainer("ChildContainerName", inherits: false);

## Container Scopes

Child containers are defined by "scopes." Scopes can be placed at 
three levels: exporting member, declaring type(s), or assembly, and 
the scope will affect all exports beneath it. By default, exports are
not included in a scope.

### Export Member Scopes

These are the most basic scope definitions as they are applied directly
to the member or type providing the export.

Examples:

    public class SomeType
    {
      [Export, CompositionContainerScope(ScopeNames.Foo)]
      public int Test1 { get { return 5; } } // This will be included

      [Export]
      public int Test2 { get { return 6; } } // This will not be included
    }

    [Export, CompositionContainerScope(ScopeNames.Foo)]
    public class SomeOtherType { } // This will be included in the Foo container

### Declaring Type Scopes

Types containing exports can be used as a scope, as well. This is a 
good way to include multiple exported members (and nested types) that
are in the same class.

Example:

    [CompositionContainerScope(ScopeNames.Foo)]
    public class SomeType
    {
      [Export]
      public int Test1 { get { return 5; } } // This will be included

      public class SomeChildType
      {
        [Export]
        public int Test1 { get { return 5; } } // This will be included

        [Export]
        public class SomeChildChildType { } // This will also be included
      }
    }

Bear in mind that this is simultaneously applied as an export member scope
if the type is exported.

### Assembly Scopes

Finally, scopes can be placed at the assembly level. There are two kinds
of supported scopes at this time.

#### Namespace Scopes

Namespaces can be used as a way to include entire subsystems into a container.
Despite the fact that namespaces transcend assemblies, all namespace scopes are
limited to the assembly in which they are defined.

Example:

    [assembly: NamespaceCompositionContainerScope("Foo", "NorthHorizon.Samples")]

#### Assembly Scopes

The most broad scoping mechanism available scopes the entire assembly into a
container.

Example:

    [assembly: CompositionContainerScope("Foo")]

## Container Scope Exclusions

Scopes can opt out of participating in a container. This is usually
accomplished by user request via an `IsExcluded = true` optional
parameter.

    [Export, CompositionContainerScope(ScopeNames.Foo, IsExcluded = true)]
    public int Test2 { get { return 6; } } // This will not be included

## Container Scope Resolution

Because scopes can be excluded, they have the potential to conflict. In
the event of a conflict, the most specific scope will take precedence,
however all declarations at that level must agree.

Examples:

    [CompositionContainerScope(ScopeNames.Foo, IsExcluded = false)]
    public class TestType
    {
      // This scope definition is more specific, so
      // ChildType will be included.
      [Export, CompositionContainerScope(ScopeNames.Foo)]
      public class ChildType { }
    }

    [assembly: NamespaceCompositionContainerScope("Foo", "NorthHorizon.Samples")]
    // This is a more specific namespace, so it will be used.
    [assembly: NamespaceCompositionContainerScope("Foo", "NorthHorizon.Samples.Foo", IsExcluded = true)]
