# DotNet6API

This is a .NET 6 API project created for learning purposes. It was primarily built using Visual Studio 2022 for Mac.

## How to Run the Project

You can start this project using either of the following two methods:

### Method 1: Using Visual Studio 2022 for Mac
1. Open Visual Studio 2022 for Mac.
2. Open the `DotNet6API.sln` solution file located in the root directory of the project.
3. Click the "Run" button at the top (or press `Cmd + Enter`) to start the project and launch the browser.

### Method 2: Using the .NET CLI (Terminal)
1. Open your Terminal.
2. Navigate to the directory containing the project source code (the folder containing the `.csproj` file, e.g., the `DotNet6API` subdirectory):
   ```bash
   cd DotNet6API
   ```
3. Run the following command to start the application:
   ```bash
   dotnet run
   ```
   > Once successfully started, the terminal will display the local URL the application is listening on (e.g., `https://localhost:<port>`). You can open this URL in your browser, or append `/swagger` to the URL to view the Swagger UI API documentation and test the endpoints.
