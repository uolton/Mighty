---
title: IEnumerables in Mighty
layout: default
nav_order: 4
---

# IEnumerables in Mighty

> All methods in Mighty which return an `IEnumerable` (or an `IAsyncEnumerable` for async) of data do **no database access at all** until you start to enumerate the enumerable:

```c#
var depts = new MightyOrm(connectionString, "Department", "DepartmentID");

// No database access happens here, instead a contract to perform the database access is returned to you
depts.All(new { Location = "London" });

// The actual database access starts here, i.e. when you start to use the contract you have been given
foreach (var dept in depts)
{
    Console.WriteLine(dept.Name);
}
```

This only applies to methods which return `IEnumerable` (the variants of `Query`, `QueryMultiple` and `All`). All the other database access methods in Mighty (`Insert`, `Update`, `Save`, all the variants of `Execute`, etc.) do all their database access before returning.

If your own method needs to return results directly rather than a contract to provide results (or more generally if you need to force database access to happen by a certain point in your code) then call `.ToList()` on the results. But be aware that this will allocate memory for a new list object containing all the results, before doing anything else with them, and so is probably not what you want for large result sets.

Here is a more complete example showing this. (This example also uses strong types, but strong types are not required for the `.ToList()` trick to work.)

```c#
public class Department
{
    public string Name { get; set; }
    public string Location { get; set; }
}

// There is no database access left to do once this has finished - what is returned is just a list!
public IEnumerable<Department> GetDepartmentsByLocation(string Location)
{
    // No database access happens here
    MightyOrm<Department> db = new MightyOrm<Department>();

    // Select all database rows where Location = @Location
    // All of the database access has finished once .ToList() has completed
    return db.All(new { Location }).ToList();
}
```

> You can apply this to return an `IEnumerable` from Mighty from a .NET API call. The API framework won't accept the `IEnumerable` directly from Mighty, but will accept the different `IEnumerable` produced after applying `.ToList()`.
