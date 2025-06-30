# Part 2: Using MCP Servers - Building MyMonkey App

> **Navigation**: [← Part 1: Setup](part1-setup.md) | [Next: Part 3 - Lunch Server →](part3-lunch-server.md)

> **Based on**: [MyMonkeyAppMCP Repository](https://github.com/jamesmontemagno/MyMonkeyAppMCP)

## Overview
In this section, you'll learn how to use existing MCP servers by building a fun monkey-themed application that integrates with GitHub. This hands-on tutorial will teach you how MCP servers work from a consumer perspective before you build your own.

## Step-by-Step Walkthrough

### 2.1 Project Setup

#### Create a new .NET project
- Initialize a new console app inside of VS Code named **MyMonkeyApp**
-- Or run the following command in your terminal:

```bash
dotnet new console -n MyMonkeyApp
```

- Add a new .gitignore file to exclude unnecessary files from your repository by running the following command:

```bash 
dotnet new gitignore
```

#### Push to GitHub
- Create a new GitHub repository
- Connect your local project
- Make your first commit
- This can be done through the VS Code Source Control panel
- Ensure your repository is public so Copilot can access it

### 2.2 MCP Server Configuration and Initial Setup


#### Configure MonkeyMCP
- Install and configure the MonkeyMCP server in VS Code
- Understanding the MonkeyMCP API endpoints
- Testing the MCP server connection
- Verify you can retrieve monkey data through Copilot

Create a new folder named **.vscode** and create a filed named **mcp.json** with the following content:

```json
{
    "inputs": [],
    "servers": {
        "monkeymcp": {
            "command": "docker",
            "args": [
                "run",
                "-i",
                "--rm",
                "jamesmontemagno/monkeymcp"
            ],
            "env": {}
        }
    }
}
```

#### Configure GitHub MCP
- Install GitHub MCP server in VS Code
- Set up GitHub authentication tokens
- Configure repository access permissions
- Verify GitHub MCP functionality by creating the issue above

Update the **mcp.json** with a new server, this time will be a remote server for GitHub:

```json
"github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
```

Your final **mcp.json** should look like this:

```json
{
    "inputs": [],
    "servers": {
        "monkeymcp": {
            "command": "docker",
            "args": [
                "run",
                "-i",
                "--rm",
                "jamesmontemagno/monkeymcp"
            ],
            "env": {}
        },
        "github": {
            "type": "http",
            "url": "https://api.githubcopilot.com/mcp/"
        }
    }
}
```

When the GitHub MCP server runs you will be prompted to authenticate with your GitHub account. Follow the instructions to complete the authentication process.

#### Setup Copilot Instructions
After creating your repository and pushing the initial commit, let's set up Copilot with project-specific context.

**Task**: Create `.github/copilot-instructions.md` file
**Steps**:
1. Create `.github` folder in your repository root
2. Add `copilot-instructions.md` file with the following content:

```markdown
This project is .NET 9 and uses C# 13.

Make sure all code generated is inside of the MyMonkeyApp project, which may be a subfolder inside of the main folder.

It is on GitHub at https://github.com/YOUR_USERNAME/YOUR_REPO_NAME

## Project Context
This is a console application that manages monkey species data and integrates with GitHub through MCP servers.

## C# Coding Standards
- Use PascalCase for class names, method names, and properties
- Use camelCase for local variables and parameters
- Use descriptive names that clearly indicate purpose
- Add XML documentation comments for public methods and classes
- Use `var` for local variables when the type is obvious
- Prefer explicit types when it improves readability
- Use async/await for asynchronous operations
- Follow the repository pattern for data access
- Use proper exception handling with try-catch blocks
- Implement IDisposable when managing resources
- Use nullable reference types to avoid null reference exceptions
- use file-scoped namespaces for cleaner code organization

## Naming Conventions
- Classes: `MonkeyHelper`, `Monkey`, `Program`
- Methods: `GetMonkeys()`, `GetRandomMonkey()`, `GetMonkeyByName()`
- Properties: `Name`, `Location`, `Population`
- Variables: `selectedMonkey`, `monkeyCount`, `userInput`
- Constants: `MAX_MONKEYS`, `DEFAULT_POPULATION`

## Architecture
- Console application with interactive menu
- Static helper class for data management
- Model classes for data representation
- Separation of concerns between UI and business logic
```

**Important**: Replace `YOUR_USERNAME` and `YOUR_REPO_NAME` with your actual GitHub username and repository name.

#### Create GitHub Issue for Project Planning
Before we start coding, let's use the GitHub MCP server to create an issue that outlines our project goals.

**Task**: Use VS Code with GitHub MCP to create a new issue
**Sample Copilot Command**:
> "Create a new GitHub issue in my repository titled 'Implement Monkey Console Application' with the following requirements: Create a C# console app that can list all available monkeys, get details for a specific monkey by name, and pick a random monkey. The app should use a Monkey model class and include ASCII art for visual appeal. Add appropriate labels like 'enhancement' and 'good first issue'. Add some details about how we may go about implementing this and a checklist for what we will need to do."

**Sample Issue Content** it may create:
```markdown
# Implement Monkey Console Application

## Description
Create a fun and interactive console application for exploring monkey species data.

## Requirements
- [ ] Create `Monkey.cs` model class with properties for Name, Location, Details, Image, Population, Latitude, Longitude
- [ ] Implement `MonkeyHelper.cs` static class for data management
- [ ] Build interactive console menu with options:
  - List all monkeys
  - Get details for specific monkey
  - Pick random monkey
  - Exit application
- [ ] Include ASCII monkey art when displaying details
- [ ] Track access counts for each monkey
- [ ] Handle error cases gracefully

## Technical Details
- .NET 9 Console Application
- C# 13 features
- Follow C# naming conventions
- Include XML documentation comments
```


### 2.3 Interactive Development with Copilot

#### Get Monkey Data
- **Task**: Ask Copilot to retrieve a list of monkeys from MonkeyMCP
- Learn how Copilot integrates with MCP servers
- Understand the data structure returned
- Handle API responses and error cases

#### Generate Monkey Model
- **Task**: Ask Copilot to create a C# model based on monkey data from MonkeyMCP
- **Expected Output**: A `Monkey.cs` class file with the following structure:

```csharp
namespace MonkeyApp
{
    public class Monkey
    {
        public string Name { get; set; }
        public string Location { get; set; }
        public string Details { get; set; }
        public string Image { get; set; }
        public int Population { get; set; }
        public double Latitude { get; set; }
        public double Longitude { get; set; }
    }
}
```

**Key Properties Explained**:
- `Name` - The monkey species name (e.g., "Proboscis Monkey")
- `Location` - Geographic habitat (e.g., "Borneo")
- `Details` - Descriptive information about the species
- `Image` - URL or path to monkey image
- `Population` - Estimated population count
- `Latitude` / `Longitude` - Geographic coordinates for mapping

#### Build the Random Monkey App
- **Task**: Create an application that selects random monkeys from the collection
- **Expected Features**:
  - Console application that displays a random monkey
  - Show monkey details (Name, Location, Population, etc.)
  - Option to "pick another monkey" 
  - Display geographic coordinates
  - Handle empty monkey collections gracefully

**Sample Output**:
```
🐒 Random Monkey Picker!
========================
Name: Proboscis Monkey
Location: Borneo
Population: 7,000
Details: Known for their distinctive large nose and reddish fur...
Coordinates: 1.5°N, 110.0°E

Press 'R' for another monkey, 'Q' to quit...
```

**Implementation Notes**:
- Use `Random` class for selection
- Implement user interaction loop
- Add emoji and formatting for better UX
- Consider displaying monkey images if supported

#### Create MonkeyHelper Service
- **Task**: Ask Copilot to create a `MonkeyHelper.cs` static class for managing monkey data
- **Expected Output**: A static helper class with the following functionality:

**Key Methods**:
- `GetMonkeys()` - Returns a List<Monkey> with 12+ different monkey species
- `GetRandomMonkey()` - Selects and tracks access to a random monkey
- `GetMonkeyByName(string name)` - Finds a specific monkey by name
- `GetMonkeyAccessCount(string name)` - Returns how many times a monkey was accessed

**Sample Copilot Command**:
> "Create a MonkeyHelper static class that manages a collection of monkey data. Include methods to get all monkeys, get a random monkey, find by name, and track access counts. The monkey data should include the monkeys from the monkey mcp server."

**Expected Data**:
The helper should contain monkeys like:
- Baboon (Africa & Asia) - Population: 10,000
- Capuchin Monkey (Central & South America) - Population: 23,000
- Blue Monkey (Central and East Africa) - Population: 12,000
- Proboscis Monkey (Borneo) - Population: 15,000
- Japanese Macaque (Japan) - Population: 1,000
- And 7+ more species with realistic data

#### Build the Console Application
- **Task**: Ask Copilot to create the main `Program.cs` with an interactive menu system
- **Expected Output**: A console application with the following menu structure:

**Sample Copilot Command**:
> "Update the Program.cs file that displays an interactive menu for the monkey app. The menu should have options to: 1) List all monkeys, 2) Get details for a specific monkey by name, 3) Get a random monkey, 4) Exit. Include ASCII art of a monkey face when displaying monkey details. Use the MonkeyHelper class methods and show access counts for each monkey."

**Expected Console Output**:
```
Monkey App Menu:
1. List all monkeys
2. Get details for a specific monkey
3. Get a random monkey
4. Exit
Choose an option: 3

  //\ 
 (o o)
(  V  )
--m-m--

Random Monkey: Proboscis Monkey
Location: Borneo
Details: The proboscis monkey or long-nosed monkey, known as the bekantan in Malay, is a reddish-brown arboreal Old World monkey that is endemic to the south-east Asian island of Borneo.
Population: 15000
Lat/Lon: 0.961883, 114.55485
Image: https://raw.githubusercontent.com/jamesmontemagno/app-monkeys/master/borneo.jpg
Accessed: 1 times
```

**Key Features**:
- Interactive menu loop until user exits
- ASCII monkey art for visual appeal
- Detailed monkey information display
- Access tracking and display
- Error handling for invalid input and missing monkeys

### 2.4 GitHub Integration

#### Create GitHub Issues
- **Task**: Use Copilot to create GitHub issues programmatically
- Specify your repository name
- Create an issue for "Add new monkeys feature" or have it create a few new ideas
- Understand GitHub API integration through MCP

#### Verify on GitHub.com
- Navigate to your repository
- Review the automatically created issue
- Examine the issue details and formatting
- Understand the MCP → GitHub workflow

## Learning Outcomes
- How to configure and use existing MCP servers
- Integration patterns with AI assistants
- Real-world API interactions through MCP
- GitHub automation possibilities

---

> **Next Step**: Now that you've learned how to use MCP servers, let's build your own from scratch!

**Continue to**: [Part 3 - Building Your Own MCP Server →](part3-lunch-server.md)
