# 🚀 Advanced Trello MCP Server

> **Enhanced Model Context Protocol Server for Trello integration with Cursor AI**
> 36 tools across Boards, Lists, Cards, Labels, and Actions APIs

[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-blue.svg)](https://www.typescriptlang.org/)
[![Trello API](https://img.shields.io/badge/Trello%20API-Complete-green.svg)](https://developer.atlassian.com/cloud/trello/rest/)
[![MCP Protocol](https://img.shields.io/badge/MCP-Compatible-purple.svg)](https://modelcontextprotocol.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

<a href="https://glama.ai/mcp/servers/@adriangrahldev/advanced-trello-mcp-server">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/@adriangrahldev/advanced-trello-mcp-server/badge" alt="Advanced Trello Server MCP server" />
</a>

## 📋 Overview

This is an **enhanced version** of the Trello MCP Server that provides comprehensive integration between Trello and Cursor AI. Originally supporting 15 basic tools, this version has been expanded to **44+ tools** covering multiple Trello API categories with enterprise-grade functionality.

## ✨ Features

### 🎯 **API Coverage (36 tools)**
- **Lists API**: 10 tools (Complete list management)
- **Cards API**: 13 tools (Enhanced card operations + attachments + comments)
- **Labels API**: 8 tools (Complete label management)
- **Actions API**: 4 tools (Action read/write)
- **Boards API**: 1 tool (Basic board access)

### 🔧 **Enterprise Features**
- **TypeScript Implementation** with strict typing
- **Zod Validation** for all inputs and outputs
- **Batch Operations** for bulk actions
- **Error Handling** with detailed error messages
- **Rate Limiting Ready** architecture
- **Extensible Design** for future API additions

## 🚀 Quick Start

### Prerequisites
- Node.js 18+ 
- Trello API Key and Token
- Cursor AI with MCP support

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/adriangrahldev/advanced-trello-mcp-server.git
   cd advanced-trello-mcp-server
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Build the project**
   ```bash
   npm run build
   ```

4. **Configure environment variables**
   ```bash
   export TRELLO_API_KEY="your_api_key"
   export TRELLO_API_TOKEN="your_api_token"
   ```

5. **Configure Cursor MCP**
   Add to your `~/.cursor/mcp.json`:
   ```json
   {
     "servers": {
       "trello": {
         "command": "/path/to/advanced-trello-mcp-server/build/index.js",
         "env": {
           "TRELLO_API_KEY": "your_api_key",
           "TRELLO_API_TOKEN": "your_api_token"
         }
       }
     }
   }
   ```

## 🛠️ Available Tools

### 📋 **Lists Management (9 tools)**
- `get-lists` - Get all lists from a board
- `create-list` - Create new list
- `update-list` - Update list properties
- `archive-list` - Archive/unarchive lists
- `move-list-to-board` - Move lists between boards
- `get-list-actions` - Get list action history
- `get-list-board` - Get board information from list
- `get-list-cards` - Get cards from list with filtering
- `archive-all-cards-in-list` - Archive all cards in list
- `move-all-cards-in-list` - Move all cards between lists

### 🎯 **Cards Management (13 tools)**
- `create-card` - Create single card
- `create-cards` - Create multiple cards (batch)
- `update-card` - Update card name, description, due date, and/or start date
- `move-card` - Move card between lists
- `move-cards` - Move multiple cards (batch)
- `archive-card` - Archive single card
- `archive-cards` - Archive multiple cards (batch)
- `get-tickets-by-list` - Get cards from specific list
- `add-comment` - Add comment to card
- `add-comments` - Add comments to multiple cards (batch)
- `get-card-comments` - Get comments for a card with optional filters
- `get-card-attachments` - Get attachment list for a card
- `download-card-attachments` - Download card attachments to disk

### 🏷️ **Labels Management (8 tools)** ✅ **COMPLETE**
- `create-label` - Create single label
- `create-labels` - Create multiple labels (batch)
- `add-label` - Add label to card
- `add-labels` - Add labels to multiple cards (batch)
- `get-label` - Get detailed label information
- `update-label` - Update label name and color
- `delete-label` - Delete label by ID
- `update-label-field` - Update specific label field

### 📊 **Actions (4 tools)**
- `get-action` - Get detailed action information
- `update-action` - Update action (comments)
- `delete-action` - Delete action (comments only)
- `get-action-reactions` - Get action reactions

### 🏢 **Boards Management (1 tool)**
- `get-boards` - Get all accessible boards

## 📈 Roadmap

Current: **36 tools**. Planned additions — see `docs/2. specifications/S01_gap_closure.md`:

- Restore 12 lost action-tools (traversal, reaction CRUD)
- Handler unification via `trelloGet/Post/Put/Delete`

**Target after gap closure: 48 tools**

## 🔧 Development

### Project Structure
```
advanced-trello-mcp-server/
├── src/                 # TypeScript source code
│   ├── index.ts         # Main MCP server implementation
│   ├── tools/           # Modular tool implementations
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Shared utilities
├── build/               # Compiled JavaScript output
│   └── index.js         # Executable entry point
├── scripts/             # Build and utility scripts
│   └── build.js         # Cross-platform build script
├── package.json         # Dependencies and scripts
├── tsconfig.json        # TypeScript configuration
└── README.md           # This file
```

### Building
```bash
# Full build (TypeScript compilation + shebang)
npm run build

# TypeScript compilation only
npm run compile
```

**Cross-Platform Build**: El script de build es completamente compatible con Windows, macOS y Linux, manejando automáticamente:
- Compilación de TypeScript
- Inyección de shebang (`#!/usr/bin/env node`)
- Permisos de ejecución (en sistemas Unix)

### Development Workflow
1. Make changes in `src/` directory
2. Run `npm run build` to compile
3. Test with Cursor AI
4. Commit changes with conventional commits

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'feat: add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Commit Convention
We use [Conventional Commits](https://www.conventionalcommits.org/):
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `refactor:` - Code refactoring
- `test:` - Adding tests
- `chore:` - Maintenance tasks

## 📚 API Documentation

This server implements tools based on the official [Trello REST API documentation](https://developer.atlassian.com/cloud/trello/rest/). Each tool includes:

- **Zod schema validation** for type safety
- **Comprehensive error handling**
- **Optional parameters** support
- **Batch operations** where applicable
- **Detailed JSDoc comments**

## 🐛 Troubleshooting

### Common Issues

**1. "Trello API credentials are not configured"**
- Ensure `TRELLO_API_KEY` and `TRELLO_API_TOKEN` are set
- Verify the token has appropriate scopes (`read` minimum, `write` for modifications)

**2. "Tool not found" errors**
- Restart Cursor AI to refresh MCP server
- Verify the build was successful (`npm run build`)
- Check MCP configuration in `~/.cursor/mcp.json`

**3. Permission errors**
- Verify your Trello token has access to the boards/cards you're trying to modify
- Some operations require `write` scope, not just `read`

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Original Trello MCP Server by [yairhaimo](https://github.com/yairhaimo/trello-mcp-server)
- [Trello API Documentation](https://developer.atlassian.com/cloud/trello/rest/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Cursor AI](https://cursor.sh/)

## 📊 Stats

- **Total Tools**: 36
- **Lines of Code**: 2,500+ TypeScript
- **Type Safety**: 100% with Zod validation

---

**Built with ❤️ for the Cursor AI community** 