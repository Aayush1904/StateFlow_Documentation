# Stateflow Documentation

Welcome to the Stateflow documentation! This folder contains comprehensive documentation for the Stateflow project management platform.

## Documentation Index

### 📐 [Component Architecture Diagram](./component-architecture.md)
A detailed overview of the application's architecture, including:
- High-level system architecture
- Frontend component hierarchy
- Editor component structure
- Data flow diagrams
- Technology stack
- Design patterns

### ✍️ [Editor Structure and Collaboration Logic](./editor-structure-collaboration.md)
Deep dive into the rich text editor and collaboration system:
- TipTap editor architecture
- Collaboration system overview
- Real-time synchronization flow
- Socket.IO implementation
- Conflict resolution strategies
- Presence and cursor tracking
- Comment system
- AI features integration

### 🚀 [Setup Guide - Local Development](./setup-guide.md)
Step-by-step instructions to get Stateflow running locally:
- Prerequisites and installation
- Backend setup (MongoDB, environment variables)
- Frontend setup
- Google OAuth configuration
- Troubleshooting common issues
- Development workflow

### ⚠️ [Known Limitations and Future Improvements](./known-limitations-future-improvements.md)
Current limitations and planned enhancements:
- Current system limitations
- Short-term improvements (3-6 months)
- Medium-term roadmap (6-12 months)
- Long-term vision (12+ months)
- Contribution guidelines

## Quick Start

New to Stateflow? Start here:

1. **Setting Up Locally**: Read the [Setup Guide](./setup-guide.md) to get the application running on your machine
2. **Understanding Architecture**: Review the [Component Architecture](./component-architecture.md) to understand how the system works
3. **Editor Details**: Explore [Editor Structure and Collaboration](./editor-structure-collaboration.md) to understand the collaborative editing features
4. **Known Issues**: Check [Known Limitations](./known-limitations-future-improvements.md) to be aware of current limitations

## Project Overview

**Stateflow** is a B2B project management platform that provides:

- **Workspace Management**: Create and manage workspaces for teams
- **Project Organization**: Organize work into projects with tasks and pages
- **Collaborative Editing**: Real-time collaborative rich text editing
- **Task Management**: Kanban boards and task tables
- **Integrations**: GitHub, Google Calendar, and more
- **AI Features**: AI-powered autocomplete and content suggestions

## Tech Stack Summary

### Frontend
- React 18 + TypeScript
- Vite (build tool)
- TipTap (rich text editor)
- Socket.IO Client (real-time)
- TanStack Query (data fetching)
- Tailwind CSS + shadcn/ui

### Backend
- Node.js + Express + TypeScript
- MongoDB with Mongoose
- Socket.IO Server (real-time collaboration)
- Passport.js (authentication)
- JWT tokens

## Getting Help

- **Setup Issues**: Check the [Setup Guide](./setup-guide.md) troubleshooting section
- **Architecture Questions**: Refer to [Component Architecture](./component-architecture.md)
- **Editor Features**: See [Editor Structure](./editor-structure-collaboration.md)
- **Feature Requests**: Review [Future Improvements](./known-limitations-future-improvements.md)


