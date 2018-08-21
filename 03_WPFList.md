# Implement the WPF Application

This application shows a list of books.

Before implementing this application, [prepare the projects](01_Preparation.md), and [implement the Books library](02_BooksLib.md).

## Create a WPF Project named BooksApp-WPF

Create a new WPF project.

Add NuGet packages and references:

    * Reference the BooksLib Library
    * Add the NuGet package *Microsoft.Extensions.DependencyInjection*
    * Add the NuGet package *Microsoft.Extensions.Logging* and *Microsoft.Extensions.Logging.Debug*


## Define the dependency injection container

Implement the `AppServices` class that registers services and view-models in the dependency injection container. The property `ServiceProvider` returns `IServiceProvider` to allow accessing the container provider.

```csharp
using BooksLib.Models;
using BooksLib.Services;
using BooksLib.ViewModels;
using Generic.ViewModels.Services;
using GenericViewModels.Services;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;

namespace BooksApp_Windows
{
    public class AppServices
    {
        private AppServices()
        {
            var services = new ServiceCollection();

            // view-models
            services.AddTransient<BooksViewModel>();

            // services
            services.AddTransient<IItemsService<Book>, BooksService>();
            services.AddTransient<IShowProgressInfo, ShowProgressInfo>();

            // stateful services
            services.AddScoped<IItemToViewModelMap<Book, BookItemViewModel>, BookToBookItemViewModelMap>();
            services.AddScoped<ISharedItems<Book>, SharedItems<Book>>();
            services.AddScoped<IRepository<Book, int>, BooksSampleRepository>();

            // logging configuration
            services.AddLogging(builder =>
            {
#if DEBUG
            builder.AddDebug().SetMinimumLevel(LogLevel.Trace);
#endif
        });

            ServiceProvider = services.BuildServiceProvider();
        }

        public IServiceProvider ServiceProvider { get; }

        private static AppServices _instance;
        private static readonly object _instanceLock = new object();
        private static AppServices GetInstance()
        {
            lock (_instanceLock)
            {
                return _instance ?? (_instance = new AppServices());
            }
        }
        public static AppServices Instance => _instance ?? GetInstance();
    }
}

```

## Books Page UI

Implement the user interface for the Main Window. The ListView control is bound to the `Items` property of the `ViewModel`. The `Command` of the refresh button is bound to the `RefreshCommand`:

```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition />
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <Menu Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2">
            <MenuItem Header="Books">
                <MenuItem Header="Refresh" Command="{Binding ViewModel.RefreshCommand}" />
                <MenuItem Header="Add" Command="{Binding ViewModel.AddCommand}" />
            </MenuItem>
        </Menu>
        <ListView Grid.Row="2" ItemsSource="{Binding ViewModel.Items, Mode=OneWay}" 
                 SelectedItem="{Binding ViewModel.SelectedItem, Mode=TwoWay}" >
        </ListView>
    </Grid>
```

## Get the View-Model from the Dependency Injection Container

Create a DI scope accessing the DI container, and retrieve the `BooksViewModel` using this scope:

```csharp
    public sealed partial class MainWindow : Window
    {
        private IServiceScope _scope;

        public BooksPage()
        {
            InitializeComponent();

            _scope = AppServices.Instance.ServiceProvider.CreateScope();
            Closed += (sender, e) => _scope.Dispose();

            ViewModel = _scope.ServiceProvider.GetRequiredService<BooksViewModel>();

            DataContext = this;
        }

        public BooksViewModel ViewModel { get; }
    }
```

## Run the Application

Now you can run the application. Clicking the Refresh menu shows the book list returned from the books repository:

![Show Books](Images/05_ShowBooksListWPF.png)