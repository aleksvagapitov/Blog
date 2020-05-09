# ASP.NET Core with Tabulator for Creating Dynamic Tables with Pagination, Filtering and Sorting



[Tabulator](http://tabulator.info) is a Javascript framework for creating Data Driven Tables that provides lots of functionality out of the box for server-side as well as front-end: sorting, filtering, exporting and many many more awesome features. Having used tools like: [Handsontable](https://www.handsontable.com), [Datatables](https://www.datatables.net) as well as custom [Razor Pages](https://docs.microsoft.com/en-us/aspnet/core/data/ef-rp/sort-filter-page?view=aspnetcore-3.0); I decided to try Tabulator, and found it to be the optimal solution that provides all of the functionality that I want at no cost, with great flexibility, and requires a relatively small amount of front-end code. In this article, I will give you a tour of how this awesome Javascript framework works effectively with ASP.NET Core and EntityFramework.

All of the code from this article can be found [here](https://github.com/aleksvagapitov/DotnetTabulatorFiltering).
<figure>
  <img src="/images/2019/oct/dotnet-tabulator-filtering.png" height="250" alt="Dotnet-Tabulator-Filtering"/>
  <figcaption>This is what we are building in this article. </figcaption>
</figure>

<!--more-->

## Model for this example

In this example we will use a Database Schema with the following Model:

{{< highlight csharp >}}
public class Contact
{
    [Key]
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
    public string Gender { get; set; }
    public string Email { get; set; }
    public int ZipCode { get; set; }
    public string About { get; set; }
}
{{< / highlight >}}


## Database Context Structure

For this example we will be using an In-Memory Storage with Dependency Injection in the Startup.cs file:

{{< highlight csharp >}}
public class TabulatorContext : DbContext
{
    public DbSet<Contact> Contacts { get; set; }

    public TabulatorContext
      (DbContextOptions<TabulatorContext> options) 
        : base (options) { }

    public void EnsureSeedData (){}
}
{{< / highlight >}}

## Repository Structure

One of the key packages that need to be installed before we get to looking at the Repository is: [EntityFrameworkPaginateCore](https://www.nuget.org/packages/EntityFrameworkPaginateCore), which is required for all the filtering and sorting on the back-end side. We can install it using Dotnet CLI using the following command:

| dotnet add package EntityFrameworkPaginateCore --version 1.1.0

*The sample project on Github contains all the required packages and all you need to do is follow instruction in the README of the project.*

The repository structure relies on the following method signature:

{{< highlight csharp >}}
Task<TabulatorViewModel> GetFilteredData (int page, int size,
  List<Dictionary<string, string>> filters, List<Dictionary<string, string>> sorters)
{{< / highlight >}}

What is important to note here is the type of 'filters' and 'sorters' parameters, which is:

{{< highlight csharp >}}
List<Dictionary<string, string>>
{{< / highlight >}}

since we will be getting filters in the following format:
{{< highlight json >}}
[{field:"age", type:">", value:52}, {field:"height", type:"<", value:142}]
{{< / highlight >}}

and for sorters:

{{< highlight json >}}
[{field: "age", dir: "asc"}]
{{< / highlight >}}

What is also important is the ViewModel that we use to pass the data back to the Tabulator:

{{< highlight csharp >}}
public class TabulatorViewModel
{
    public IEnumerable<dynamic> Data { get; set; }
    public int Last_page { get; set; }
}
{{< / highlight >}}

The full method is described [here](https://github.com/aleksvagapitov/DotnetTabulatorFiltering/blob/master/Models/TabulatorRespository.cs) and mostly comprises of iterating over the list of dictionary items and setting the fields and values for each of the columns described in the Model that was mentioned at the beginning of the blog.

## Front-End Tabulator Code
{{< highlight csharp >}}
var table = new Tabulator("#example-table", {
    pagination: "remote",
    ajaxURL: baseUrl + apiEndpoint,
    paginationSize: 10,
    paginationSizeSelector: [10, 20, 30, 50],
    ajaxFiltering: true,
    ajaxSorting: true,
    columns: [
        { title: "FirstName", field: "firstName", sorter: "string", headerFilter: "input" },
        { title: "LastName", field: "lastName", sorter: "string", headerFilter: "input" },
        { title: "Age", field: "age", sorter: "number", headerFilter: "input" },
        { title: "Gender", field: "gender", sorter: "string", headerFilter: "input" },
        { title: "Email", field: "email", sorter: "string", headerFilter: "input" },
        { title: "ZipCode", field: "zipCode", sorter: "number", headerFilter: "input" },
        { title: "About", field: "about", sorter: "string", headerFilter: "input", width: 500 }
    ]
});
{{< / highlight >}}

The above structure allows us to filter on the data using Ajax and paginate it. What is very handy about this structure is that we can remove columns from the front-end without having to adjust the back-end. **However**, be careful with the objects that you are passing to the front-end and check what your API returns. The way we can ensure that data does not get leaked is by using ViewModels and [AutoMapper](https://automapper.org) or regular object-composition to pass only the relevant data to the front-end. I have left a commented out line in the Repository method implementation of where the AutoMapper clause would go as to not introduce too many concepts in this blog. Lastly, you can use tools like [Postman](https://www.getpostman.com) to query the API and see the data that it returns.

Again, all of the code can be found [here](https://github.com/aleksvagapitov/DotnetTabulatorFiltering) and full documentation on Tabulator can be found [here](http://tabulator.info)

Enjoy and feel free to report any bugs or problems you encounter to: [GitHub issues](https://github.com/aleksvagapitov/DotnetTabulatorFiltering/issues) :)

