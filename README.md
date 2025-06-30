# Let's Learn MCP with C# - Tutorial Series

A comprehensive guide to understanding and building Model Context Protocol (MCP) Servers for C# developers.

## What You'll Build

By the end of this tutorial series, you'll have:

1. **🐒 MyMonkey App** - A fun console application that uses existing MCP servers to manage monkey data and create GitHub issues
2. **🍽️ Lunch Roulette MCP Server** - Your own custom MCP server that helps AI assistants pick restaurants and track dining preferences

## Tutorial Structure

### [Part 1: Prerequisites and Setup](part1-setup.md)
**⏱️ Time: 15-20 minutes**

Set up your development environment and understand MCP fundamentals:
- Install VS Code, .NET 9 SDK, and C# Dev Kit
- Learn what Model Context Protocol is and why it matters
- Understand the client-server architecture
- Verify your development environment

**Start here**: [Part 1: Prerequisites and Setup →](part1-setup.md)

---

### [Part 2: Using MCP Servers - MyMonkey App](part2-monkey-app.md)
**⏱️ Time: 15-30 minutes**

Learn to use existing MCP servers by building a monkey-themed application:
- Configure MonkeyMCP and GitHub MCP servers
- Create C# models for monkey data
- Build an interactive console application with ASCII art
- Integrate with GitHub to create issues programmatically
- Understand how AI assistants interact with MCP servers

**What you'll create**:
```
🐒 Random Monkey Picker!
========================
Name: Proboscis Monkey
Location: Borneo
Population: 15,000
Details: Known for their distinctive large nose...
```

**Continue to**: [Part 2: MyMonkey App →](part2-monkey-app.md)

---

### [Part 3: Building Your Own MCP Server - Lunch Roulette](part3-lunch-server.md)
**⏱️ Time: 30-45 minutes**

Build a complete MCP server from scratch:
- Create a restaurant management service with JSON persistence
- Implement four MCP tools for AI assistant integration
- Handle dependency injection and async operations
- Test and deploy your server
- Understand MCP protocol implementation details

**What you'll build**:
- `GetRestaurants()` - List all available restaurants
- `AddRestaurant()` - Add new dining options
- `PickRandomRestaurant()` - AI-powered lunch selection
- `GetVisitStatistics()` - Track dining patterns

**Continue to**: [Part 3: Lunch Roulette Server →](part3-lunch-server.md)

---

## Quick Start

If you're ready to dive in immediately:

1. **Prerequisites**: Ensure you have VS Code, .NET 9 SDK, and C# Dev Kit installed
2. **Choose your path**:
   - 🆕 **New to MCP?** Start with [Part 1: Setup](part1-setup.md)
   - 🔧 **Want to use MCP?** Jump to [Part 2: MyMonkey App](part2-monkey-app.md)
   - 🏗️ **Ready to build?** Go to [Part 3: Lunch Server](part3-lunch-server.md)

## Learning Outcomes

After completing this tutorial series, you'll understand:

- ✅ **MCP Fundamentals** - What MCP is and how it enables AI-tool integration
- ✅ **Consumer Patterns** - How to use existing MCP servers in your applications
- ✅ **Server Development** - How to build and deploy your own MCP servers
- ✅ **C# Best Practices** - Dependency injection, async patterns, and JSON serialization
- ✅ **AI Integration** - How AI assistants interact with your tools and services

## Technology Stack

This tutorial uses modern C# development practices:

- **.NET 9** - Latest .NET runtime with performance improvements
- **C# 13** - Modern language features and syntax
- **System.Text.Json** - High-performance JSON serialization with source generators
- **Microsoft.Extensions.Hosting** - Dependency injection and application lifecycle
- **ModelContextProtocol** - Official C# SDK for MCP server development

## Repository Structure

```
letslearnmcp-csharp/
├── README.md                 # This overview
├── part1-setup.md           # Prerequisites and environment setup
├── part2-monkey-app.md      # Using existing MCP servers
└── part3-lunch-server.md    # Building your own MCP server
```

## Additional Resources

- 📖 [MCP Official Documentation](https://modelcontextprotocol.io/)
- 🛠️ [C# MCP SDK Repository](https://github.com/modelcontextprotocol/csharp-sdk)
- 🐒 [MyMonkey App Repository](https://github.com/jamesmontemagno/MyMonkeyAppMCP)
- 🍽️ [Lunch Roulette Repository](https://github.com/jamesmontemagno/LunchRouletteMCP)

## Contributing

This tutorial is open source! Feel free to:
- 🐛 Submit improvements and corrections
- 💡 Add more examples and use cases
- 🤝 Share your own MCP server implementations
- 💬 Help others in the discussions

---

**Ready to get started?** Begin with [Part 1: Prerequisites and Setup →](part1-setup.md)

*Happy coding! 🐒🍽️*