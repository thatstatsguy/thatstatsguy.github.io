---
layout: post
title:  Implementing EF Core For Blazor
date:   2023-05-01
description: Featuring Clean Architecture
tags: EFCore Databases til
categories: Blazor
---
Object Relational Mappers (ORMs) are a really cool way to integrate persistence into a Blazor application. Entity Framework (EF) Core is one such library provided by Microsoft to help you manage how your database tables map to entities in your code. In today's article, we'll have a look at what it takes to implement this in Blazor. To spice things up, we're going to implement it using Clean Architecture.

## Clean Architecture
I'm not going to rehash the whole idea of Clean Architecture here. There's a lot of awesome people out there who have [already covered](https://learn.microsoft.com/en-us/events/dotnetconf-2021/clean-architecture-with-aspnet-core-6) this ad nauseam. If you've never seen the term before, the main thing to keep in mind about Clean Architecture is that we arrange our code for one of several reasons including:
- Ensuring our "Core" application code has **zero** dependencies on IO, UI and external services.
- Putting our core types and contracts in a domain specific project allowing a developer to focus on what's important i.e. the problem that needs to be solved.

## The Final Application
All code for the article is available [here](https://github.com/thatstatsguy/til/tree/main/EFCoreBootstrapProject). We're going to be using the baseline Blazor Project to persist data to a local DB using EF Core. This is going to be done using a `Add New Book` on the UI and `Get Existing Book` will go and retrieve it from the DB.

## Setting up the Project with Clean Architecture

To begin with, create a new Blazor server project in Rider or VS. Ensure that the project is baked into a solution as we'll be adding more projects to the solution next.

After the project has been created, let's go ahead and create two new project folder, `Core` and `Infrastructure`. `Infrastructure` will be used for our EF Core code and `Core` will be used to contain our domain specific code. Let's go ahead and create two new class libraries `Domain` and `Application` under `Core` and a `Persistence` library under `Infrastructure`.

## Setting up Core Code
Time to set up the domain specific logic and contracts. For our example, we've got two entities that need to be captured in the DB, Authors and Books. Since this is strictly required to solve our "business problem" of storing books, this will fall under the Domain project. Adding the code to the projects, the code will look as follows.

```
public class Author
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Surname { get; set; }
}
public class Book
{
    public Guid Id { get; set; }
    public string Title { get; set; }
    public Author Author { get; set; }
}
```
It's time to set up the EF Core code which will will persist information to the DB.

## Setting up EF Core
Before adding in any code, let's add in the following nuget packages.

```
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="7.0.5" />
```

Adding the above package allows us to communicate directly with SQL Server. The first thing that we need to set up is the DB Context which will communicate with the database on our behalf. We're going to call the file `BootstrapDbContext.cs` 

```
using Domain;
using Microsoft.EntityFrameworkCore;

namespace Persistence;

public class BootstrapDbContext : DbContext
{
    
    public DbSet<Author> Authors { get; set; }
    public DbSet<Book> Books { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Data Source=(localdb)\\mssqllocaldb;Initial Catalog=BootstrapDatabase;");
    }
}
```

Breaking down the above code, we're setting up a `DbSet` for both `Book` and `Author`. This is used by EF Core to know which tables can be added, removed and manipulated. The `OnConfiguring` method is used to set up the connection to the DB for our DB context. We could always use dependency injection with `IConfiguration` in order to not hard code the connection string, but it's not needed for our small example which won't be deployed.

Next, we're going to set up the repository to interact with the DB. Recall that we don't want our application to have any dependencies on the DB code. We'll achieve this by coding to interfaces which define the functionality which our repository will implement. To ensure that the direction of dependency is pointing towards the core, we'll define the interface within the `Application` project. 

```
public interface IRepository<T> where T : class
{
    public T GetById(Guid Id);
    public void Add(T newItem);
}
```
While this is a really small example, keeping the domain and core application layers separate for long term success with the Architecture pattern. The `IRepository` defines two basic CRUD methods, adding and reading. Returning to our repository, we'll ensure that these methods are implemented with the help of the DB Context which will be passed in via the constructor.

```
public class BookRepository : IRepository<Book>
{
    private readonly BootstrapDbContext _dbContext;
    public BookRepository(BootstrapDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public Book GetById(Guid id)
    {
        return _dbContext.Books.Find(id) ?? throw new Exception("Id not found!");
    }

    public void Add(Book newItem)
    {
        _dbContext.Books.Add(newItem);
        //without this, you'll see changes locally, but they will not persist to the DB
        _dbContext.SaveChanges();
    }
}
```

Nothing particularly exciting here to talk about, it's important to note that without the save changes method call any new books that are added won't be persisted. We're also assuming that our book will always be found. Lastly, let's create an extension method which will be used to register the services to the IOC container on program startup.

```
public static class PersistenceServiceRegistration
{
    public static IServiceCollection AddPersistenceServices(this IServiceCollection services)
    {
        services.AddDbContext<BootstrapDbContext>();
        services.AddScoped<IRepository<Book>, BookRepository>();
        return services;
    }
}
```

## Performing EF Core Migrations
Before we go any further, we need to get EF core to setup the DB details on our behalf. Open up the command line in the folder containing the Persistence project and enter in the following lines.

```
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet ef migrations add InitialCreate
dotnet ef database update
```
You should see confirmation of the database details being created. This can be confirmed by looking at the Database after these commands have been run. In case you make a mistake and want to hit a hard restart, I've left some notes in the project on how to nuke your existing local DB - do so with caution!

## Using EF Core in Blazor
Only thing left to do is set up the application to use EF Core. In the program startup we'll make use of our newly created extension method.

`builder.Services.AddPersistenceServices();`

We'll add the two buttons to add and retrieve the book on the index page.

```
@page "/"
@using Application.Contracts.Persistence
@using Domain
@inject IRepository<Book> BookRepository

<PageTitle>Index</PageTitle>

<button @onclick="AddNewBook">Add new book</button>

<button @onclick="GetExistingBook">Get existing book</button>

@if (_loadedBook is not null)
{
    <p>Book found! Title: @_loadedBook.Title, Author: @_loadedBook.Author.Name @_loadedBook.Author.Surname </p>
}

<SurveyPrompt Title="How is Blazor working for you?"/>

@code {
    private Guid _bookId = Guid.Parse("60CBED03-F764-4D9E-86C1-E7A33506A484");
    private Guid _authorId =  Guid.Parse("337BD93E-4369-4041-85BE-42614A095B9E");
    private Book? _loadedBook;
    private void AddNewBook()
    {
        BookRepository.Add(new Book()
        {
            Id = _bookId,
            Title = "Test Title",
            Author = new Author() { Id = _authorId, Name = "Author Name", Surname = "Author Surname" }
        });
    }

    private void GetExistingBook()
    {
        _loadedBook = BookRepository.GetById(_bookId);
    }
}
```

Booting up our application, the DB should now reflect that a book (and associated author) is added when we click the `Add New Book` button is clicked. Similarly, when clicking `Get Existing Book` will display the book retrieved from the DB.

## Wrapping Up
Note in the above code that we're injecting an `IRepository` into our page - at this point in the application we have no idea that we're using EF Core to do our data persistence. This is fantastic as it means that testing will be significantly easier and swapping out implementation details can be done without breaking existing implementation details. Hopefully this will be useful in your next Blazor/ASPNet Core application!

Until next time :)