# .NET C# - IEnumerable to CSV

This helper -inspired by [Scott Hanselman's function](http://www.hanselman.com/blog/?date=2010-02-04)- allows you to export any `IEnumerable` to a `CSV.`

I added a few features to the original function.

* Possibility to ignore a property during the export with an attribute `[CsvIgnore]`
* Added column header to the exported CSV
* Possibility to custom the column's header (usefull for Excel exports) with the `[CsvColumnName]` attribute

Let's see how it works :

Example of a class to export :

```csharp
public class ClassToExport
{
        public string ImportantProperty { get; set; }

        [CsvColumnName ("FancyColumnName")]
        public string ImportantString { get; set; }

        [CsvIgnore]
        public string NotImportant { get; set; }
}
```

How to use the helper :

```csharp
            IEnumerable<ClassToExport> listToExport = new List<ClassToExport> {
                new ClassToExport { ImportantProperty = "ImportantProperty1", ImportantString = "ImportantString1", NotImportant = "NotImportant1" },
                new ClassToExport { ImportantProperty = "ImportantProperty2", ImportantString = "ImportantString2", NotImportant = "NotImportant2" },
                new ClassToExport { ImportantProperty = "ImportantProperty3", ImportantString = "ImportantString3", NotImportant = "NotImportant3" }
            };

           Console.WriteLine (listToExport.ToCsv());
```

Output :

```csharp
ImportantProperty;FancyColumnName
"ImportantProperty1";"ImportantString1"
"ImportantProperty2";"ImportantString2"
"ImportantProperty3";"ImportantString3"
```

Source Code :

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Helpers {
/// <summary>
/// Inspired by Scott Hanselman function to export linq to csv
/// http://www.hanselman.com/blog/?date=2010-02-04
/// </summary>
public static class LinqToCSV {

    public static string ToCsv<T> (this IEnumerable<T> items) where T : class {
        var csvBuilder = new StringBuilder ();
        var properties = typeof (T).GetProperties ();

        // Column names
        string headers = string.Join (";", properties
            .Where (x => !Attribute.IsDefined (x, typeof (CsvIgnoreAttribute)))
            .Select (p => !Attribute.IsDefined (p, typeof (CsvColumnNameAttribute)) ? p.Name : p.GetCustomAttributes (true).OfType<CsvColumnNameAttribute>().FirstOrDefault().Name));
        csvBuilder.AppendLine (headers);

        // Datas lines
        foreach (T item in items) {
            string line = string.Join (";", properties
                .Where (x => !Attribute.IsDefined (x, typeof (CsvIgnoreAttribute))) // Filter on properties which has the CsvIgnoreAttribute
                .Select (p => p.GetValue (item, null).ToCsvValue ())
                .ToArray ());
            csvBuilder.AppendLine (line);
        }

        return csvBuilder.ToString ();
    }

    private static string ToCsvValue<T> (this T value) {
        if (value == null) return "\"\"";

        if (value is string) {
            return string.Format ("\"{0}\"", value.ToString ().Replace ("\"", "\\\""));
        }

        double dummy;
        if (double.TryParse (value.ToString (), out dummy)) {
            return string.Format ("{0}", value);
        }

        return string.Format ("\"{0}\"", value);
    }
}

// Allow to ignore a property during csv parsing
[AttributeUsage (AttributeTargets.Property, AllowMultiple = false)]
public class CsvIgnoreAttribute : Attribute { }

[AttributeUsage (AttributeTargets.Property, AllowMultiple = false)]
public class CsvColumnNameAttribute : Attribute {
    private string columnName;

    public CsvColumnNameAttribute (string columnName) {
        this.columnName = columnName;
    }

    public virtual string Name {
        get { return columnName; }
    }
}
```

Any enhancement welcome!