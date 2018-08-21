# Implement the UWP Application

This application shows a list of books.

Before implementing this application, [prepare the projects](01_Preparation.md), and [implement the Books library](02_BooksLib.md).

## Create a UWP Project named BooksApp-Windows

Create a new Universal Windows Platform project.

Add NuGet packages and references:

    * Reference the BooksLib Library
    * Add the NuGet package *Microsoft.Extensions.DependencyInjection*
    * Add the NuGet package *Microsoft.Extensions.Logging* and *Microsoft.Extensions.Logging.Debug*
    * Add the NuGet package *Microsoft.UI.Xaml*

The package *Microsoft.UI.Xaml* contains XAML controls for Windows apps that can be used with older Windows 10 versions. These controls need some styles included in the application. Add the `XamlControlResources` element to the file `App.xaml`:

```xml
<Application
    x:Class="BooksApp_Windows.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:BooksApp_Windows">
    <Application.Resources>
        <XamlControlsResources xmlns="using:Microsoft.UI.Xaml.Controls"/>
    </Application.Resources>

</Application>
```

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

## Implement the main navigation

The application makes use of the `NavigationView` from the package `Microsoft.UI.Xaml`. Add this code to `MainPage.xaml`:

```xml
    <ui:NavigationView Background="{ThemeResource ApplicationPageBackgroundThemeBrush}" SelectionChanged="{x:Bind OnNavigationSelectionChanged, Mode=OneTime}">
        <ui:NavigationView.MenuItems>
            <ui:NavigationViewItem Content="Books" Tag="books">
                <ui:NavigationViewItem.Icon>
                    <FontIcon FontFamily="Segoe MDL2 Assets" Glyph="&#xE82D;" />
                </ui:NavigationViewItem.Icon>
            </ui:NavigationViewItem>
            <ui:NavigationViewItem Content="VideoPage" Tag="video" Icon="Video">
            </ui:NavigationViewItem>
        </ui:NavigationView.MenuItems>

        <Frame x:Name="ContentFrame" Margin="24">
            <Frame.ContentTransitions>
                <TransitionCollection>
                    <NavigationThemeTransition/>
                </TransitionCollection>
            </Frame.ContentTransitions>
        </Frame>
    </ui:NavigationView>
```

With the code-behind file, implement the `OnNavigationSelectionChanged` handler method:

```csharp
        public void OnNavigationSelectionChanged(UI.NavigationView sender, UI.NavigationViewSelectionChangedEventArgs args)
        {
            if (args.SelectedItem is UI.NavigationViewItem navigationItem)
            {
                switch (navigationItem.Tag)
                {
                    case "books":
                        ContentFrame.Navigate(typeof(BooksPage));
                        break;
                    case "page2":
                        ContentFrame.Navigate(typeof(VideoPage));
                        break;
                }
            }
        }
```

## Books Page UI

Implement the user interface for the BooksPage. The ListView control is bound to the `Items` property of the `ViewModel`. The `Command` of the refresh button is bound to the `RefreshCommand`:

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
        <StackPanel Orientation="Horizontal" Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2">
            <AppBarButton Icon="Refresh" Command="{x:Bind ViewModel.RefreshCommand}" Label="Get Books" IsCompact="True"  />
            <AppBarButton Icon="Add" Command="{x:Bind ViewModel.AddCommand}" Label="Add Book" IsCompact="True" />
        </StackPanel>
        <ListView Grid.Row="1" Grid.Column="0" ItemsSource="{x:Bind ViewModel.Items, Mode=OneWay}" 
                 SelectedItem="{x:Bind ViewModel.SelectedItem, Mode=TwoWay}" >
        </ListView>

    </Grid>
```

## Get the View-Model from the Dependency Injection Container

Create a DI scope accessing the DI container, and retrieve the `BooksViewModel` using this scope:

```csharp
    public sealed partial class BooksPage : Page
    {
        private IServiceScope _scope;

        public BooksPage()
        {
            InitializeComponent();

            _scope = AppServices.Instance.ServiceProvider.CreateScope();
            Unloaded += (sender, e) => _scope.Dispose();

            ViewModel = _scope.ServiceProvider.GetRequiredService<BooksViewModel>();
        }

        public BooksViewModel ViewModel { get; }
    }
```

## Run the Application

Now you can run the application. Clicking the Refresh button shows the book list returned from the books repository:

![Show Books](Images/04_ShowBooksList.png)