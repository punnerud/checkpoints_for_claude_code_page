# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Diff is a macOS application that provides automatic version control and checkpoint management for Claude Code projects. It integrates with Claude through an MCP (Model Context Protocol) server to track file changes and create checkpoints during development.

## Build and Development Commands

```bash
# Build the project (Debug)
cd "Claude Diff" && xcodebuild -scheme "Claude Diff" -configuration Debug build

# Build the project (Release)
cd "Claude Diff" && xcodebuild -scheme "Claude Diff" -configuration Release build

# Run the app (after building)
open "Claude Diff/build/Debug/Claude Diff.app"

# Clean build
cd "Claude Diff" && xcodebuild -scheme "Claude Diff" clean

# Run tests (if any)
cd "Claude Diff" && xcodebuild -scheme "Claude Diff" test
```

## Architecture Overview

### Core Components

1. **MainView.swift** - Main UI orchestrator that coordinates project selection, checkpoint display, and MCP server status. Handles navigation between different views and manages application state.

2. **MCPService.swift** - Implements the MCP server (port 8765) that communicates with Claude Code. Provides tools for task tracking (`update_task_status`), checkpoint creation, and diff generation.

3. **CheckpointService.swift** - Core checkpoint management system that:
   - Creates and restores checkpoints with full file backup
   - Tracks file changes between checkpoints 
   - Stores checkpoint data in `.claudecheckpoints/` directories
   - Maintains both Core Data persistence and filesystem backups for reliability

4. **GitService.swift** - Monitors file system changes and provides Git integration for diff generation and status tracking.

5. **FileTracker.swift** - Low-level file tracking that computes file hashes and compares states to detect additions, modifications, and deletions.

### Data Flow

1. User selects project folder → MainView sets up services
2. MCPService starts server and registers with Claude CLI
3. Claude Code connects and calls MCP tools (e.g., `update_task_status`)
4. CheckpointService tracks changes and creates checkpoints
5. Checkpoints are persisted to both Core Data and filesystem (`.claudecheckpoints/`)

### MCP Tools Available to Claude

- `update_task_status` - Start or complete a task (automatically creates checkpoints)
- `get_diff_summary` - Get current uncommitted changes
- `create_checkpoint` - Manually create a checkpoint
- `list_checkpoints` - List all checkpoints for the project
- `restore_checkpoint` - Restore to a previous checkpoint

## Key Implementation Details

- Checkpoints are self-contained with full file backups in `.claudecheckpoints/backups/checkpoint_[timestamp]/`
- Each checkpoint stores both current state and previous versions for proper diff generation
- The app uses NotificationCenter for inter-service communication
- File changes are detected via polling (2-second intervals) rather than FSEvents due to sandboxing
- Claude configuration files (CLAUDE.md, .clauderc) are preserved during reverts