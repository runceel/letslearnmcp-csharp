# Part 3: Building Your Own MCP Server - Lunch Roulette

> **Navigation**: [← Part 2: Monkey App](part2-monkey-app.md) | [Back to Overview](README.md)

> **Based on**: [LunchRouletteMCP Repository](https://github.com/jamesmontemagno/LunchRouletteMCP)

## Overview
Learn to build your own MCP server from scratch by creating a lunch recommendation system. This tutorial will teach you the complete process of designing, implementing, and deploying an MCP server using C#.

## 3.1 Project Foundation

### Create console app

Create a new folder called **LunchTime** and then create a new C# console app named **LunchTimeMCP** inside of the folder. 

### NuGet Package Installation
Essential packages for MCP server development:
```xml
<PackageReference Include="Microsoft.Extensions.Hosting" Version="9.0.6" />
<PackageReference Include="ModelContextProtocol" Version="0.3.0-preview.1" />
<PackageReference Include="System.Text.Json" Version="9.0.6" />
```

### Program.cs Setup
- Configure the host builder
- Set up dependency injection
- Register services and MCP tools
- Initialize MCP server components
- Handle application lifecycle

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using ModelContextProtocol.Server;
using LunchTimeMCP;

var builder = Host.CreateEmptyApplicationBuilder(settings: null);

builder.Services
    .AddSingleton<RestaurantService>()  // Register our service
    .AddMcpServer()
    .WithStdioServerTransport()
    .WithMcpServerToolsFrom<RestaurantTools>();  // Register our MCP tools

await builder.Build().RunAsync();
```

**Key Points**:
- `AddSingleton<RestaurantService>()` registers our service for dependency injection
- `WithMcpServerToolsFrom<RestaurantTools>()` automatically discovers and registers all MCP tools
- The MCP library handles the rest of the server setup and protocol implementation

## 3.2 Core Components

### RestaurantService Implementation
You'll copy the complete **RestaurantService** below into a new file named **RestaurantService.cs**. This service provides the core business logic and includes:

**Key Features:**
- **Data Persistence**: JSON file storage in user's AppData directory
- **Restaurant Management**: Add, retrieve, and track restaurants
- **Visit Tracking**: Count and statistics for restaurant selections
- **Seed Data**: Pre-populated with trendy West Hollywood restaurants
- **Async Operations**: All methods use async/await pattern
- **Error Handling**: Try-catch blocks for file operations

**Core Methods:**
- `GetRestaurantsAsync()` - Retrieve all restaurants
- `AddRestaurantAsync()` - Add new restaurant with validation
- `PickRandomRestaurantAsync()` - Random selection with visit tracking
- `GetVisitStatsAsync()` - Raw visit statistics
- `GetFormattedVisitStatsAsync()` - User-friendly formatted statistics

**Data Models:**
- `Restaurant` - Core restaurant entity with Id, Name, Location, FoodType, DateAdded
- `RestaurantVisitInfo` - Visit tracking with count and last visited date
- `RestaurantData` - Container for JSON serialization of restaurants and visit counts
- `FormattedRestaurantStats` - Presentation layer for statistics display

**Technical Architecture:**
- **File Storage**: JSON persistence in `%AppData%/LunchTimeMCP/restaurants.json`
- **JSON Serialization**: Uses System.Text.Json with source generators (`RestaurantContext`)
- **Memory Management**: In-memory collections with periodic file saves
- **Initialization**: Automatic seeding with 10 trendy LA restaurants if empty

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;

namespace LunchTimeMCP;

public class RestaurantService
{
    private readonly string dataFilePath;
    private List<Restaurant> restaurants = new();
    private Dictionary<string, int> visitCounts = new();

    public RestaurantService()
    {
        // Store data in user's app data directory
        var appDataPath = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
        var appDir = Path.Combine(appDataPath, "LunchTimeMCP");
        Directory.CreateDirectory(appDir);
        
        dataFilePath = Path.Combine(appDir, "restaurants.json");
        LoadData();
        
        // Initialize with trendy West Hollywood restaurants if empty
        if (restaurants.Count == 0)
        {
            InitializeWithTrendyRestaurants();
            SaveData();
        }
    }

    public async Task<List<Restaurant>> GetRestaurantsAsync()
    {
        return await Task.FromResult(restaurants.ToList());
    }

    public async Task<Restaurant> AddRestaurantAsync(string name, string location, string foodType)
    {
        var restaurant = new Restaurant
        {
            Id = Guid.NewGuid().ToString(),
            Name = name,
            Location = location,
            FoodType = foodType,
            DateAdded = DateTime.UtcNow
        };

        restaurants.Add(restaurant);
        SaveData();
        
        return await Task.FromResult(restaurant);
    }

    public async Task<Restaurant?> PickRandomRestaurantAsync()
    {
        if (restaurants.Count == 0)
            return null;

        var random = new Random();
        var selectedRestaurant = restaurants[random.Next(restaurants.Count)];
        
        // Track the visit
        if (visitCounts.ContainsKey(selectedRestaurant.Id))
            visitCounts[selectedRestaurant.Id]++;
        else
            visitCounts[selectedRestaurant.Id] = 1;

        SaveData();
        
        return await Task.FromResult(selectedRestaurant);
    }

    public async Task<Dictionary<string, RestaurantVisitInfo>> GetVisitStatsAsync()
    {
        var stats = new Dictionary<string, RestaurantVisitInfo>();
        
        foreach (var restaurant in restaurants)
        {
            var visitCount = visitCounts.GetValueOrDefault(restaurant.Id, 0);
            stats[restaurant.Name] = new RestaurantVisitInfo
            {
                Restaurant = restaurant,
                VisitCount = visitCount,
                LastVisited = visitCount > 0 ? DateTime.UtcNow : null // In a real app, you'd track actual visit dates
            };
        }

        return await Task.FromResult(stats);
    }

    public async Task<FormattedRestaurantStats> GetFormattedVisitStatsAsync()
    {
        var stats = await GetVisitStatsAsync();
        
        var formattedStats = stats.Values
            .OrderByDescending(x => x.VisitCount)
            .Select(stat => new FormattedRestaurantStat
            {
                Restaurant = stat.Restaurant.Name,
                Location = stat.Restaurant.Location,
                FoodType = stat.Restaurant.FoodType,
                VisitCount = stat.VisitCount,
                TimesEaten = stat.VisitCount == 0 ? "Never" : 
                            stat.VisitCount == 1 ? "Once" : 
                            $"{stat.VisitCount} times"
            })
            .ToList();

        return new FormattedRestaurantStats
        {
            Message = "Restaurant visit statistics:",
            Statistics = formattedStats,
            TotalRestaurants = stats.Count,
            TotalVisits = stats.Values.Sum(x => x.VisitCount)
        };
    }

    private void LoadData()
    {
        if (!File.Exists(dataFilePath))
            return;

        try
        {
            var json = File.ReadAllText(dataFilePath);
            var data = JsonSerializer.Deserialize<RestaurantData>(json, RestaurantContext.Default.RestaurantData);
            
            if (data != null)
            {
                restaurants = data.Restaurants ?? new List<Restaurant>();
                visitCounts = data.VisitCounts ?? new Dictionary<string, int>();
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error loading data: {ex.Message}");
        }
    }

    private void SaveData()
    {
        try
        {
            var data = new RestaurantData
            {
                Restaurants = restaurants,
                VisitCounts = visitCounts
            };
            
            var json = JsonSerializer.Serialize(data, RestaurantContext.Default.RestaurantData);
            File.WriteAllText(dataFilePath, json);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error saving data: {ex.Message}");
        }
    }

    private void InitializeWithTrendyRestaurants()
    {
        var trendyRestaurants = new List<Restaurant>
        {
            new() { Id = Guid.NewGuid().ToString(), Name = "Guelaguetza", Location = "3014 W Olympic Blvd", FoodType = "Oaxacan Mexican", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "Republique", Location = "624 S La Brea Ave", FoodType = "French Bistro", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "Night + Market WeHo", Location = "9041 Sunset Blvd", FoodType = "Thai Street Food", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "Gracias Madre", Location = "8905 Melrose Ave", FoodType = "Vegan Mexican", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "The Ivy", Location = "113 N Robertson Blvd", FoodType = "Californian", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "Catch LA", Location = "8715 Melrose Ave", FoodType = "Seafood", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "Cecconi's", Location = "8764 Melrose Ave", FoodType = "Italian", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "Earls Kitchen + Bar", Location = "8730 W Sunset Blvd", FoodType = "Global Comfort Food", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "Pump Restaurant", Location = "8948 Santa Monica Blvd", FoodType = "Mediterranean", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "Craig's", Location = "8826 Melrose Ave", FoodType = "American Contemporary", DateAdded = DateTime.UtcNow }
        };

        restaurants.AddRange(trendyRestaurants);
    }
}

public partial class Restaurant
{
    public string Id { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public string Location { get; set; } = string.Empty;
    public string FoodType { get; set; } = string.Empty;
    public DateTime DateAdded { get; set; }
}

public partial class RestaurantVisitInfo
{
    public Restaurant Restaurant { get; set; } = new();
    public int VisitCount { get; set; }
    public DateTime? LastVisited { get; set; }
}

public partial class RestaurantData
{
    public List<Restaurant> Restaurants { get; set; } = new();
    public Dictionary<string, int> VisitCounts { get; set; } = new();
}

public class FormattedRestaurantStat
{
    public string Restaurant { get; set; } = string.Empty;
    public string Location { get; set; } = string.Empty;
    public string FoodType { get; set; } = string.Empty;
    public int VisitCount { get; set; }
    public string TimesEaten { get; set; } = string.Empty;
}

public class FormattedRestaurantStats
{
    public string Message { get; set; } = string.Empty;
    public List<FormattedRestaurantStat> Statistics { get; set; } = new();
    public int TotalRestaurants { get; set; }
    public int TotalVisits { get; set; }
}

[JsonSerializable(typeof(List<Restaurant>))]
[JsonSerializable(typeof(Restaurant))]
[JsonSerializable(typeof(RestaurantVisitInfo))]
[JsonSerializable(typeof(Dictionary<string, RestaurantVisitInfo>))]
[JsonSerializable(typeof(RestaurantData))]
[JsonSerializable(typeof(FormattedRestaurantStat))]
[JsonSerializable(typeof(FormattedRestaurantStats))]
internal sealed partial class RestaurantContext : JsonSerializerContext
{
}
```

### Service Architecture
The RestaurantService follows several important patterns:

**Dependency Injection Ready**
- Parameterless constructor for easy DI registration
- No external dependencies, self-contained service
- Singleton-safe with proper state management

**Data Layer Separation**
- Private methods handle file I/O (`LoadData()`, `SaveData()`)
- Public async methods provide business operations
- Clear separation between persistence and business logic

**Error Resilience**
- Graceful handling of missing or corrupt data files
- Console logging for debugging file operations
- Automatic recovery with seed data initialization

**Performance Considerations**
- In-memory operations for fast access
- Lazy loading from file system
- Minimal file I/O with batched saves

**JSON Source Generation**
- `RestaurantContext` provides AOT-friendly serialization
- All model types registered for optimal performance
- Type-safe serialization with compile-time validation

## 3.3 MCP Tools Development

Now we'll implement the MCP tools that expose our restaurant functionality to AI assistants. Create a new file called **RestaurantTools.cs** and implement each tool step by step.

### Tool Implementation Overview
Each tool follows the same pattern:
1. Use `[McpServerTool]` attribute to register with MCP
2. Add descriptive `[Description]` attributes for the AI to understand the tool's purpose
3. Use dependency injection to access the `RestaurantService`
4. Return JSON-serialized responses using the `RestaurantContext`

### Tool 1: Get All Restaurants
**Purpose**: Retrieve a list of all available restaurants
**Parameters**: None
**Implementation**:

```csharp
[McpServerTool, Description("Get a list of all restaurants available for lunch.")]
public async Task<string> GetRestaurants()
{
    var restaurants = await restaurantService.GetRestaurantsAsync();
    return JsonSerializer.Serialize(restaurants, RestaurantContext.Default.ListRestaurant);
}
```

**Key Points**:
- Simple method with no parameters
- Returns JSON-serialized list of restaurants
- Uses `RestaurantContext.Default.ListRestaurant` for optimized serialization

### Tool 2: Add New Restaurant
**Purpose**: Add a new restaurant to the available options
**Parameters**: `name`, `location`, `foodType`
**Implementation**:

```csharp
[McpServerTool, Description("Add a new restaurant to the lunch options.")]
public async Task<string> AddRestaurant(
    [Description("The name of the restaurant")] string name,
    [Description("The location/address of the restaurant")] string location,
    [Description("The type of food served (e.g., Italian, Mexican, Thai, etc.)")] string foodType)
{
    var restaurant = await restaurantService.AddRestaurantAsync(name, location, foodType);
    return JsonSerializer.Serialize(restaurant, RestaurantContext.Default.Restaurant);
}
```

**Key Points**:
- Each parameter has a descriptive `[Description]` attribute
- Calls the service method to add the restaurant
- Returns the newly created restaurant object as JSON

### Tool 3: Pick Random Restaurant
**Purpose**: Select a random restaurant and track the visit
**Parameters**: None
**Implementation**:

```csharp
[McpServerTool, Description("Pick a random restaurant from the available options for lunch.")]
public async Task<string> PickRandomRestaurant()
{
    var selectedRestaurant = await restaurantService.PickRandomRestaurantAsync();
    
    if (selectedRestaurant == null)
    {
        return JsonSerializer.Serialize(new { 
            message = "No restaurants available. Please add some restaurants first!" 
        });
    }

    return JsonSerializer.Serialize(new { 
        message = $"🍽️ Time for lunch at {selectedRestaurant.Name}!",
        restaurant = selectedRestaurant 
    });
}
```

**Key Points**:
- Handles the case when no restaurants are available
- Returns a friendly message with emoji
- Includes both a message and the selected restaurant data
- Automatically tracks the visit in the service

### Tool 4: Get Visit Statistics
**Purpose**: Retrieve formatted statistics about restaurant visits
**Parameters**: None
**Implementation**:

```csharp
[McpServerTool, Description("Get statistics about how many times each restaurant has been visited.")]
public async Task<string> GetVisitStatistics()
{
    var formattedStats = await restaurantService.GetFormattedVisitStatsAsync();
    return JsonSerializer.Serialize(formattedStats, RestaurantContext.Default.FormattedRestaurantStats);
}
```

**Key Points**:
- Uses the formatted stats method for user-friendly output
- Returns comprehensive statistics including visit counts and totals
- Leverages the pre-built formatting logic in the service

### Complete RestaurantTools.cs File Structure

```csharp
using System.ComponentModel;
using System.Text.Json;
using ModelContextProtocol.Server;

namespace LunchTimeMCP;

[McpServerToolType]
public sealed class RestaurantTools
{
    private readonly RestaurantService restaurantService;

    public RestaurantTools(RestaurantService restaurantService)
    {
        this.restaurantService = restaurantService;
    }

    // All four tools implemented here...
}
```

**Architecture Notes**:
- `[McpServerToolType]` marks the class as containing MCP tools
- Constructor injection provides access to `RestaurantService`
- All methods are `async Task<string>` returning JSON
- Consistent error handling and response formatting

## 3.4 Testing Your MCP Server

### Local Testing
1. Build and run your MCP server: `dotnet run`
2. The server will start and listen for MCP protocol connections
3. Test with VS Code by configuring your server in the MCP settings
4. Verify each tool works as expected

### Integration with AI Assistants
1. Configure your MCP server in your preferred AI assistant
2. Test each tool through natural language commands
3. Verify data persistence and error handling
4. Monitor performance and debug any issues

## 3.5 Deployment and Production

### Deployment Strategies
- **Local Development**: Run directly from your development machine
- **Production Deployment**: Deploy to cloud services or on-premises servers
- **Configuration Management**: Use environment variables and configuration files
- **Monitoring and Maintenance**: Implement logging and health checks

## Learning Outcomes
- Complete MCP server architecture understanding
- Tool development patterns and best practices
- Protocol implementation details
- Production deployment considerations
- Integration with AI assistants

---

> **Congratulations!** You've completed the MCP with C# tutorial series. You now have the knowledge to both use existing MCP servers and build your own from scratch.

**Return to**: [Tutorial Overview](README.md)
