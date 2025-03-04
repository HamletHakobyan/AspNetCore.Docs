---
title: Prerender and integrate ASP.NET Core Razor components
author: guardrex
description: Learn about Razor component integration scenarios for Blazor apps, including prerendering of Razor components on the server.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/31/2021
no-loc: [Home, Privacy, Kestrel, appsettings.json, "ASP.NET Core Identity", cookie, Cookie, Blazor, "Blazor Server", "Blazor WebAssembly", "Identity", "Let's Encrypt", Razor, SignalR]
uid: blazor/components/prerendering-and-integration
zone_pivot_groups: blazor-hosting-models
---
# Prerender and integrate ASP.NET Core Razor components

::: zone pivot="webassembly"

::: moniker range=">= aspnetcore-5.0"

Razor components can be integrated into Razor Pages and MVC apps in a hosted Blazor WebAssembly solution. When the page or view is rendered, components can be prerendered at the same time.

## Solution configuration

### Prerendering configuration

To set up prerendering for a hosted Blazor WebAssembly app:

1. Host the Blazor WebAssembly app in an ASP.NET Core app. A standalone Blazor WebAssembly app can be added to an ASP.NET Core solution, or you can use a hosted Blazor WebAssembly app created from the [Blazor WebAssembly project template](xref:blazor/tooling) with the hosted option:

   * Visual Studio: Select the **Advanced** > **ASP.NET Core hosted** checkbox when creating the Blazor WebAssembly app. In this article's examples, the solution is named `BlazorHosted`.
   * Visual Studio Code/.NET CLI command shell: `dotnet new blazorwasm -ho` (use the `-ho|--hosted` option). Use the `-o|--output {LOCATION}` option to create a folder for the solution and set the solution's project namespaces. In this article's examples, the solution is named `BlazorHosted` (`dotnet new blazorwasm -ho -o BlazorHosted`).

   For the examples in this article, the client project's namespace is `BlazorHosted.Client`, and the server project's namespace is `BlazorHosted.Server`.

1. **Delete** the `wwwroot/index.html` file from the Blazor WebAssembly **`Client`** project.

1. In the **`Client`** project, **delete** the following line in `Program.Main` (`Program.cs`):

   ```diff
   - builder.RootComponents.Add<App>("#app");
   ```

1. Add a `Pages/_Host.cshtml` file to the **`Server`** project's `Pages` folder. You can obtain a `_Host.cshtml` file from a project created from the Blazor Server template with the `dotnet new blazorserver -o BlazorServer` command in a command shell (the `-o BlazorServer` option creates a folder for the project). After placing the `Pages/_Host.cshtml` file into the **`Server`** project of the hosted Blazor WebAssembly solution, make the following changes to the file:

   * Provide an [`@using`](xref:mvc/views/razor#using) directive for the **`Client`** project (for example, `@using BlazorHosted.Client`).
   * Update the stylesheet links to point to the WebAssembly project's stylesheets. In the following example, the client project's namespace is `BlazorHosted.Client`:

     ```diff
     - <link href="css/site.css" rel="stylesheet" />
     - <link href="_content/BlazorServer/_framework/scoped.styles.css" rel="stylesheet" />
     + <link href="css/app.css" rel="stylesheet" />
     + <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
     ```

     > [!NOTE]
     > Leave the `<link>` element that requests the Bootstrap stylesheet (`css/bootstrap/bootstrap.min.css`) in place.

   * Update the `render-mode` of the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) to prerender the root `App` component with <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.WebAssemblyPrerendered>:

     ```diff
     - <component type="typeof(App)" render-mode="ServerPrerendered" />
     + <component type="typeof(App)" render-mode="WebAssemblyPrerendered" />
     ```

   * Update the Blazor script source to use the client-side Blazor WebAssembly script:

     ```diff
     - <script src="_framework/blazor.server.js"></script>
     + <script src="_framework/blazor.webassembly.js"></script>
     ```

1. In `Startup.Configure` of the **`Server`** project, change the fallback from the `index.html` file to the `_Host.cshtml` page.

   `Startup.cs`:

   ```diff
   - endpoints.MapFallbackToFile("index.html");
   + endpoints.MapFallbackToPage("/_Host");
   ```

1. Run the **`Server`** project. The hosted Blazor WebAssembly app is prerendered by the **`Server`** project for clients.

### Configuration for embedding Razor components into pages and views

The following sections and examples in this article for embedding Razor components of the client Blazor WebAssembly app into pages and views of the server app require additional configuration.

Use a default Razor Pages or MVC layout file in the **`Server`** project. The **`Server`** project must have the following files and folders.

Razor Pages:

* `Pages/Shared/_Layout.cshtml`
* `Pages/_ViewImports.cshtml`
* `Pages/_ViewStart.cshtml`

MVC:

* `Views/Shared/_Layout.cshtml`
* `Views/_ViewImports.cshtml`
* `Views/_ViewStart.cshtml`

Obtain the preceding files from an app created from the Razor Pages or MVC project template. For more information, see <xref:tutorials/razor-pages/razor-pages-start> or <xref:tutorials/first-mvc-app/start-mvc>.

Update the namespaces in the imported `_ViewImports.cshtml` file to match those in use by the **`Server`** project receiving the files.

Update the imported layout file (`_Layout.cshtml`) to include the **`Client`** project's styles. In the following example, the **`Client`** project's namespace is `BlazorHosted.Client`. The `<title>` element can be updated at the same time.

`Pages/Shared/_Layout.cshtml` (Razor Pages) or `Views/Shared/_Layout.cshtml` (MVC):

```diff
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
-   <title>@ViewData["Title"] - DonorProject</title>
+   <title>@ViewData["Title"] - BlazorHosted</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" />
+   <link href="css/app.css" rel="stylesheet" />
+   <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
</head>
```

The imported layout contains `Home` and `Privacy` navigation links. To make the `Home` link point to the hosted Blazor WebAssembly app, change the hyperlink:

```diff
- <a class="nav-link text-dark" asp-area="" asp-page="/Index">Home</a>
+ <a class="nav-link text-dark" href="/">Home</a>
```

In an MVC layout file:

```diff
- <a class="nav-link text-dark" asp-area="" asp-controller="Home" 
-     asp-action="Index">Home</a>
+ <a class="nav-link text-dark" href="/">Home</a>
```

To make the `Privacy` link lead to a privacy page, add a privacy page to the **`Server`** project.

`Pages/Privacy.cshtml` in the **`Server`** project:

```cshtml
@page
@model BlazorHosted.Server.Pages.PrivacyModel
@{
}

<h1>Privacy Policy</h1>
```

If an MVC-based privacy view is preferred, create a privacy view in the **`Server`** project.

`View/Home/Privacy.cshtml`:

```cshtml
@{
    ViewData["Title"] = "Privacy Policy";
}

<h1>@ViewData["Title"]</h1>
```

In the `Home` controller, return the view.

`Controllers/HomeController.cs`:

```diff
+ public IActionResult Privacy()
+ {
+     return View();
+ }
```

Import static assets to the **`Server`** project from the donor project's `wwwroot` folder:

* `wwwroot/css` folder and contents
* `wwwroot/js` folder and contents
* `wwwroot/lib` folder and contents

If the donor project is created from an ASP.NET Core project template and the files aren't modified, you can copy the entire `wwwroot` folder from the donor project into the **`Server`** project and remove the `favicon.ico` icon file.

> [!NOTE]
> If the **`Client`** and **`Server`** projects contain the same static asset in their `wwwroot` folders (for example, `favicon.ico`), an exception is thrown because the static asset in each folder shares the same web root path:
>
> > The static web asset '...\favicon.ico' has a conflicting web root path '/wwwroot/favicon.ico' with the project file 'wwwroot\favicon.ico'.
>
> Therefore, host a static asset in either `wwwroot` folder, not both.

## Render components in a page or view with the Component Tag Helper

After [configuring the solution](#solution-configuration), including the [additional configuration](#configuration-for-embedding-razor-components-into-pages-and-views), the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) supports two render modes for rendering a component from a Blazor WebAssembly app in a page or view:

* <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.WebAssembly>
* <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.WebAssemblyPrerendered>

In the following Razor Pages example, the `Counter` component is rendered in a page. To make the component interactive, the Blazor WebAssembly script is included in the page's [render section](xref:mvc/views/layout#sections). To avoid using the full namespace for the `Counter` component with the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) (`{APP ASSEMBLY}.Pages.Counter`), add an [`@using`](xref:mvc/views/razor#using) directive for the client project's `Pages` namespace. In the following example, the **`Client`** project's namespace is `BlazorHosted.Client`.

In the **`Server`** project, `Pages/RazorPagesCounter1.cshtml`:

```cshtml
@page
@using BlazorHosted.Client.Pages

<component type="typeof(Counter)" render-mode="WebAssemblyPrerendered" />

@section Scripts {
    <script src="_framework/blazor.webassembly.js"></script>
}
```

Run the **`Server`** project. Navigate to the Razor page at `/razorpagescounter1`. The prerendered `Counter` component is embedded in the page.

<xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configures whether the component:

* Is prerendered into the page.
* Is rendered as static HTML on the page or if it includes the necessary information to bootstrap a Blazor app from the user agent.

For more information on the Component Tag Helper, including passing parameters and <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configuration, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.

Additional work might be required depending on the static resources that components use and how layout pages are organized in an app. Typically, scripts are added to a page or view's `Scripts` render section and stylesheets are added to the layout's `<head>` element content.

## Render components in a page or view with a CSS selector

After [configuring the solution](#solution-configuration), including the [additional configuration](#configuration-for-embedding-razor-components-into-pages-and-views), add root components to the **`Client`** project of a hosted Blazor WebAssembly solution in `Program.Main`. In the following example, the `Counter` component is declared as a root component with a CSS selector that selects the element with the `id` that matches `counter-component`. In the following example, the **`Client`** project's namespace is `BlazorHosted.Client`.

In `Program.cs` of the **`Client`** project, add the namespace for the project's Razor components to the top of the file:

```diff
+ using BlazorHosted.Client.Pages;
```

After the `builder` is established in `Program.Main`, add the `Counter` component as a root component:

```diff
+ builder.RootComponents.Add<Counter>("#counter-component");
```

In the following Razor Pages example, the `Counter` component is rendered in a page. To make the component interactive, the Blazor WebAssembly script is included in the page's [render section](xref:mvc/views/layout#sections).

In the **`Server`** project, `Pages/RazorPagesCounter2.cshtml`:

```cshtml
@page

<div id="counter-component">Loading...</div>

@section Scripts {
    <script src="_framework/blazor.webassembly.js"></script>
}
```

Run the **`Server`** project. Navigate to the Razor page at `/razorpagescounter2`. The prerendered `Counter` component is embedded in the page.

Additional work might be required depending on the static resources that components use and how layout pages are organized in an app. Typically, scripts are added to a page or view's `Scripts` render section and stylesheets are added to the layout's `<head>` element content.

> [!NOTE]
> The preceding example throws a <xref:Microsoft.JSInterop.JSException> if a Blazor WebAssembly app is prerendered and integrated into a Razor Pages or MVC app **simultaneously** with a CSS selector. Navigating to one of the **`Client`** project's Razor components throws the following exception:
>
> > Microsoft.JSInterop.JSException: Could not find any element matching selector '#counter-component'.
>
> This is normal behavior because prerendering and integrating a Blazor WebAssembly app with routable Razor components is incompatible with the use of CSS selectors.

## Additional Blazor WebAssembly resources

* [Hosted Blazor WebAssembly logging](xref:blazor/fundamentals/logging#hosted-blazor-webassembly-logging)
* [State management: Handle prerendering](xref:blazor/state-management#handle-prerendering)
* [Prerendering support with assembly lazy loading](xref:blazor/webassembly-lazy-load-assemblies#assembly-load-logic-in-onnavigateasync)
* Razor component lifecycle subjects that pertain to prerendering
  * [Component initialization (`OnInitialized{Async}`)](xref:blazor/components/lifecycle#component-initialization-oninitializedasync)
  * [After component render (`OnAfterRender{Async}`)](xref:blazor/components/lifecycle#after-component-render-onafterrenderasync)
  * [Stateful reconnection after prerendering](xref:blazor/components/lifecycle#stateful-reconnection-after-prerendering): Although the content in the section focuses on Blazor Server and stateful SignalR *reconnection*, the scenario for prerendering in hosted Blazor WebAssembly apps (<xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.WebAssemblyPrerendered>) involves similar conditions and approaches to prevent executing developer code twice. A *new state preservation feature* is planned for the ASP.NET Core 6.0 release that will improve the management of initialization code execution during prerendering.
  * [Detect when the app is prerendering](xref:blazor/components/lifecycle#detect-when-the-app-is-prerendering)
* Authentication and authorization subjects that pertain to prerendering
  * [General aspects](xref:blazor/security/index#aspnet-core-blazor-authentication-and-authorization)
  * [Support prerendering with authentication](xref:blazor/security/webassembly/additional-scenarios#support-prerendering-with-authentication)
* [Host and deploy: Blazor WebAssembly](xref:blazor/host-and-deploy/webassembly)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Integrating Razor components into Razor Pages and MVC apps in a hosted Blazor WebAssembly solution is supported in ASP.NET Core in .NET 5 or later. Select a .NET 5 or later version of this article.

::: moniker-end

::: zone-end

::: zone pivot="server"

Razor components can be integrated into Razor Pages and MVC apps in a Blazor Server app. When the page or view is rendered, components can be prerendered at the same time.

After [configuring the project](#configuration), use the guidance in the following sections depending on the project's requirements:

* Routable components: For components that are directly routable from user requests. Follow this guidance when visitors should be able to make an HTTP request in their browser for a component with an [`@page`](xref:mvc/views/razor#page) directive.
  * [Use routable components in a Razor Pages app](#use-routable-components-in-a-razor-pages-app)
  * [Use routable components in an MVC app](#use-routable-components-in-an-mvc-app)
* [Render components from a page or view](#render-components-from-a-page-or-view): For components that aren't directly routable from user requests. Follow this guidance when the app embeds components into existing pages and views with the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper).

## Configuration

An existing Razor Pages or MVC app can integrate Razor components into pages and views:

1. In the project's layout file:

   * Add the following `<base>` tag to the `<head>` element in `Pages/Shared/_Layout.cshtml` (Razor Pages) or `Views/Shared/_Layout.cshtml` (MVC):

     ```diff
     + <base href="~/" />
     ```

     The `href` value (the *app base path*) in the preceding example assumes that the app resides at the root URL path (`/`). If the app is a sub-application, follow the guidance in the *App base path* section of the <xref:blazor/host-and-deploy/index#app-base-path> article.

   * Add a `<script>` tag for the `blazor.server.js` script immediately before the `Scripts` render section.

     `Pages/Shared/_Layout.cshtml` (Razor Pages) or `Views/Shared/_Layout.cshtml` (MVC):

     ```diff
         ...
     +   <script src="_framework/blazor.server.js"></script>

         @await RenderSectionAsync("Scripts", required: false)
     </body>
     ```

     The framework adds the `blazor.server.js` script to the app. There's no need to manually add a `blazor.server.js` script file to the app.

1. Add an imports file to the root folder of the project with the following content. Change the `{APP NAMESPACE}` placeholder to the namespace of the project.

   `_Imports.razor`:

   ```razor
   @using System.Net.Http
   @using Microsoft.AspNetCore.Authorization
   @using Microsoft.AspNetCore.Components.Authorization
   @using Microsoft.AspNetCore.Components.Forms
   @using Microsoft.AspNetCore.Components.Routing
   @using Microsoft.AspNetCore.Components.Web
   @using Microsoft.JSInterop
   @using {APP NAMESPACE}
   ```

1. Register the Blazor Server service in `Startup.ConfigureServices`.

   `Startup.cs`:

   ```diff
   + services.AddServerSideBlazor();
   ```

1. Add the Blazor Hub endpoint to the endpoints (`app.UseEndpoints`) of `Startup.Configure`.

   `Startup.cs`:

   ```diff
   + endpoints.MapBlazorHub();
   ```

1. Integrate components into any page or view. For example, add a `Counter` component to the project's `Shared` folder.

   `Pages/Shared/Counter.razor` (Razor Pages) or `Views/Shared/Counter.razor` (MVC):

   ```razor
   <h1>Counter</h1>

   <p>Current count: @currentCount</p>

   <button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

   @code {
       private int currentCount = 0;

       private void IncrementCount()
       {
           currentCount++;
       }
   }
   ```

   **Razor Pages**:

   In the project's `Index` page of a Razor Pages app, add the `Counter` component's namespace and embed the component into the page. When the `Index` page loads, the `Counter` component is prerendered in the page. In the following example, replace the `{APP NAMESPACE}` placeholder with the project's namespace.

   `Pages/Index.cshtml`:

   ```cshtml
   @page
   @using {APP NAMESPACE}.Pages.Shared
   @model IndexModel
   @{
       ViewData["Title"] = "Home page";
   }

   <div>
       <component type="typeof(Counter)" render-mode="ServerPrerendered" />
   </div>
   ```

   In the preceding example, replace the `{APP NAMESPACE}` placeholder with the app's namespace.

   **MVC**:

   In the project's `Index` view of an MVC app, add the `Counter` component's namespace and embed the component into the view. When the `Index` view loads, the `Counter` component is prerendered in the page. In the following example, replace the `{APP NAMESPACE}` placeholder with the project's namespace.

   `Views/Home/Index.cshtml`:

   ```cshtml
   @using {APP NAMESPACE}.Views.Shared
   @{
       ViewData["Title"] = "Home Page";
   }

   <div>
       <component type="typeof(Counter)" render-mode="ServerPrerendered" />
   </div>
   ```

For more information, see the [Render components from a page or view](#render-components-from-a-page-or-view) section.

## Use routable components in a Razor Pages app

*This section pertains to adding components that are directly routable from user requests.*

To support routable Razor components in Razor Pages apps:

1. Follow the guidance in the [Configuration](#configuration) section.

1. Add an `App` component to the project root with the following content.

   `App.razor`:

   ```razor
   @using Microsoft.AspNetCore.Components.Routing

   <Router AppAssembly="@typeof(Program).Assembly">
       <Found Context="routeData">
           <RouteView RouteData="routeData" />
       </Found>
       <NotFound>
           <h1>Page not found</h1>
           <p>Sorry, but there's nothing here!</p>
       </NotFound>
   </Router>
   ```

   [!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

1. Add a `_Host` page to the project with the following content.

   `Pages/_Host.cshtml`:

   ```cshtml
   @page "/blazor"
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   Components use the shared `_Layout.cshtml` file for their layout.

   <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configures whether the `App` component:

   * Is prerendered into the page.
   * Is rendered as static HTML on the page or if it includes the necessary information to bootstrap a Blazor app from the user agent.

   For more information on the Component Tag Helper, including passing parameters and <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configuration, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.

1. In the `Startup.Configure` endpoints of `Startup.cs`, add a low-priority route for the `_Host` page as the last endpoint:

   ```diff
   app.UseEndpoints(endpoints =>
   {
       endpoints.MapRazorPages();
       endpoints.MapBlazorHub();
   +   endpoints.MapFallbackToPage("/_Host");
   });
   ```

1. Add routable components to the project.

   `Pages/RoutableCounter.razor`:

   ```razor
   @page "/routable-counter"

   <h1>Routable Counter</h1>

   <p>Current count: @currentCount</p>

   <button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

   @code {
       private int currentCount = 0;

       private void IncrementCount()
       {
           currentCount++;
       }
   }
   ```

1. Run the project and navigate to the routable `RoutableCounter` component at `/routable-counter`.

For more information on namespaces, see the [Component namespaces](#component-namespaces) section.

## Use routable components in an MVC app

*This section pertains to adding components that are directly routable from user requests.*

To support routable Razor components in MVC apps:

1. Follow the guidance in the [Configuration](#configuration) section.

1. Add an `App` component to the project root with the following content.

   `App.razor`:

   ```razor
   @using Microsoft.AspNetCore.Components.Routing

   <Router AppAssembly="@typeof(Program).Assembly">
       <Found Context="routeData">
           <RouteView RouteData="routeData" />
       </Found>
       <NotFound>
           <h1>Page not found</h1>
           <p>Sorry, but there's nothing here!</p>
       </NotFound>
   </Router>
   ```

   [!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

1. Add a `_Host` view to the project with the following content.

   `Views/Home/_Host.cshtml`:

   ```cshtml
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   Components use the shared `_Layout.cshtml` file for their layout.

   <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configures whether the `App` component:

   * Is prerendered into the page.
   * Is rendered as static HTML on the page or if it includes the necessary information to bootstrap a Blazor app from the user agent.

   For more information on the Component Tag Helper, including passing parameters and <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configuration, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.

1. Add an action to the Home controller.

   `Controllers/HomeController.cs`:

   ```csharp
   public IActionResult Blazor()
   {
      return View("_Host");
   }
   ```

1. In the `Startup.Configure` endpoints of `Startup.cs`, add a low-priority route for the controller action that returns the `_Host` view:

   ```diff
   app.UseEndpoints(endpoints =>
   {
       endpoints.MapControllerRoute(
           name: "default",
           pattern: "{controller=Home}/{action=Index}/{id?}");
       endpoints.MapBlazorHub();
   +   endpoints.MapFallbackToController("Blazor", "Home");
   });
   ```

1. Add routable components to the project.

   `Pages/RoutableCounter.razor`:

   ```razor
   @page "/routable-counter"

   <h1>Routable Counter</h1>

   <p>Current count: @currentCount</p>

   <button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

   @code {
       private int currentCount = 0;

       private void IncrementCount()
       {
           currentCount++;
       }
   }
   ```

1. Run the project and navigate to the routable `RoutableCounter` component at `/routable-counter`.

For more information on namespaces, see the [Component namespaces](#component-namespaces) section.

## Render components from a page or view

*This section pertains to adding components to pages or views, where the components aren't directly routable from user requests.*

To render a component from a page or view, use the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper).

### Render stateful interactive components

Stateful interactive components can be added to a Razor page or view.

When the page or view renders:

* The component is prerendered with the page or view.
* The initial component state used for prerendering is lost.
* New component state is created when the SignalR connection is established.

The following Razor page renders a `Counter` component:

```cshtml
<h1>My Razor Page</h1>

<component type="typeof(Counter)" render-mode="ServerPrerendered" 
    param-InitialValue="InitialValue" />

@functions {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

For more information, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.

### Render noninteractive components

In the following Razor page, the `Counter` component is statically rendered with an initial value that's specified using a form. Since the component is statically rendered, the component isn't interactive:

```cshtml
<h1>My Razor Page</h1>

<form>
    <input type="number" asp-for="InitialValue" />
    <button type="submit">Set initial value</button>
</form>

<component type="typeof(Counter)" render-mode="Static" 
    param-InitialValue="InitialValue" />

@functions {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

For more information, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.

## Component namespaces

When using a custom folder to hold the project's Razor components, add the namespace representing the folder to either the page/view or to the `_ViewImports.cshtml` file. In the following example:

* Components are stored in the `Components` folder of the project.
* The `{APP NAMESPACE}` placeholder is the project's namespace. `Components` represents the name of the folder.

```cshtml
@using {APP NAMESPACE}.Components
```

The `_ViewImports.cshtml` file is located in the `Pages` folder of a Razor Pages app or the `Views` folder of an MVC app.

For more information, see <xref:blazor/components/index#namespaces>.

## Additional Blazor Server resources

* [State management: Handle prerendering](xref:blazor/state-management#handle-prerendering)
* Razor component lifecycle subjects that pertain to prerendering
  * [Component initialization (`OnInitialized{Async}`)](xref:blazor/components/lifecycle#component-initialization-oninitializedasync)
  * [After component render (`OnAfterRender{Async}`)](xref:blazor/components/lifecycle#after-component-render-onafterrenderasync)
  * [Stateful reconnection after prerendering](xref:blazor/components/lifecycle#stateful-reconnection-after-prerendering)
  * [Detect when the app is prerendering](xref:blazor/components/lifecycle#detect-when-the-app-is-prerendering)
* [Authentication and authorization: General aspects](xref:blazor/security/index#aspnet-core-blazor-authentication-and-authorization)
* [Blazor Server rerendering](xref:blazor/fundamentals/handle-errors#blazor-server-prerendering-server)
* [Host and deploy: Blazor Server](xref:blazor/host-and-deploy/server)
* [Threat mitigation: Cross-site scripting (XSS)](xref:blazor/security/server/threat-mitigation#cross-site-scripting-xss)

::: zone-end
