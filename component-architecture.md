# Component Architecture Diagram

## Overview

Stateflow is a project management platform built with a modern React frontend and Node.js/Express backend. The application follows a component-based architecture with clear separation of concerns.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client (React + Vite)                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    App.tsx                                │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Routing Layer                          │  │  │
│  │  │  - Protected Routes                                 │  │  │
│  │  │  - Auth Routes                                      │  │  │
│  │  │  - Base Routes                                      │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                                                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Layout Components                      │  │  │
│  │  │  - AppLayout (Workspace Layout)                     │  │  │
│  │  │  - BaseLayout (Auth/Public Layout)                  │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              Context Providers                            │  │
│  │  - AuthProvider (Authentication State)                    │  │
│  │  - NotificationProvider (Real-time Notifications)           │  │
│  │  - ThemeProvider (Dark/Light Mode)                        │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTP/REST + WebSocket
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Backend (Node.js + Express)                │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    Express Server                         │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Middleware Layer                       │  │  │
│  │  │  - CORS                                             │  │  │
│  │  │  - Authentication (JWT, Passport)                   │  │  │
│  │  │  - Session Management                               │  │  │
│  │  │  - Error Handling                                   │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                                                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Route Handlers                         │  │  │
│  │  │  - Auth Routes                                      │  │  │
│  │  │  - Workspace Routes                                 │  │  │
│  │  │  - Page Routes                                      │  │  │
│  │  │  - Task Routes                                      │  │  │
│  │  │  - Member Routes                                    │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                                                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Controllers                            │  │  │
│  │  │  - Request/Response Handling                        │  │  │
│  │  │  - Validation                                       │  │  │
│  │  │  - Service Calls                                    │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                                                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Services (Business Logic)              │  │  │
│  │  │  - Auth Service                                     │  │  │
│  │  │  - Workspace Service                                │  │  │
│  │  │  - Page Service                                     │  │  │
│  │  │  - Collaboration Service                            │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                                                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Socket.IO Server                       │  │  │
│  │  │  - Real-time Collaboration                          │  │  │
│  │  │  - Presence Tracking                                │  │  │
│  │  │  - Activity Feed                                    │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Mongoose ODM
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      MongoDB Database                           │
│  - Users, Workspaces, Projects, Tasks, Pages                    │
│  - Comments, Activities, Notifications                           │
│  - Members, Roles, Permissions                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Frontend Component Hierarchy

```
App.tsx
├── AppRoutes
    ├── BaseLayout (Auth/Public)
    │   ├── LoginPage
    │   ├── SignupPage
    │   └── NotFound
    │
    └── AppLayout (Protected Workspace)
        ├── Header
        │   ├── Logo
        │   ├── SearchDialog
        │   ├── NotificationBadge
        │   └── ThemeToggle
        │
        ├── AsideBar
        │   ├── WorkspaceSwitcher
        │   ├── NavMain (Workspace Navigation)
        │   ├── NavProjects (Project List)
        │   └── WorkspaceStats
        │
        └── Page Components
            ├── DashboardPage
            ├── WorkspacePage
            │   ├── WorkspaceHeader
            │   ├── RecentProjects
            │   ├── RecentTasks
            │   ├── RecentPages
            │   └── WorkspaceAnalytics
            │
            ├── ProjectPage
            │   ├── ProjectHeader
            │   ├── KanbanBoard
            │   ├── TaskTable
            │   └── ProjectAnalytics
            │
            └── PageEditor
                ├── PageHeader
                └── RichTextEditor (with collaboration)
```

## Editor Component Architecture

```
RichTextEditor (rich-text-editor.tsx)
├── Editor Core (TipTap)
│   ├── Extensions
│   │   ├── StarterKit (Basic formatting)
│   │   ├── Link
│   │   ├── Image
│   │   ├── Table (TableRow, TableCell, TableHeader)
│   │   ├── TaskList & TaskItem
│   │   ├── Mention
│   │   ├── SlashCommand
│   │   ├── Comment (Custom extension)
│   │   ├── AIFloatingMenu
│   │   └── AIAutocomplete
│   │
│   └── EditorContent
│
├── EditorToolbar
│   └── Formatting Controls (Bold, Italic, Lists, etc.)
│
├── Collaboration Features
│   ├── useCollaboration Hook
│   │   ├── Socket.IO Connection
│   │   ├── Document Updates
│   │   ├── Cursor Tracking
│   │   └── Presence Management
│   │
│   ├── LiveCursors (Other users' cursors)
│   ├── PresenceIndicator (Who's online)
│   └── CommentOverlay (Inline comments)
│
├── AI Features
│   ├── AIFloatingMenu (Text selection AI actions)
│   └── AIAutocomplete (Inline suggestions)
│
└── Additional Features
    ├── Voice Input (Speech Recognition)
    ├── WhiteboardModal
    └── ContentInsights
```

## Data Flow Diagram

### Authentication Flow

```
User Action
    │
    ▼
Login/Signup Component
    │
    ▼
Auth Service (API Call)
    │
    ▼
Backend Auth Controller
    │
    ▼
Auth Service (Business Logic)
    │
    ├──→ JWT Token Generation
    │
    └──→ Cookie Session
            │
            ▼
    Token stored in localStorage
            │
            ▼
    AuthProvider Context Updated
            │
            ▼
    Protected Routes Enabled
```

### Collaboration Flow

```
User Types in Editor
    │
    ▼
Editor onUpdate Handler
    │
    ├──→ Local State Update
    │
    └──→ useCollaboration.sendDocumentUpdate()
            │
            ▼
    Socket.IO Client
            │
            ▼
    Backend Collaboration Service
            │
            ├──→ Broadcast to other users in room
            │   │
            │   └──→ Socket.IO emit to room
            │
            └──→ Mention Detection
                    │
                    └──→ Notification Service
                            │
                            └──→ User Notifications
```

### Page Editing Flow

```
User Opens Page
    │
    ▼
PageEditor Component
    │
    ├──→ Fetch Page Data (TanStack Query)
    │   │
    │   └──→ Backend Page Controller
    │           │
    │           └──→ Page Service
    │                   │
    │                   └──→ MongoDB Query
    │
    ├──→ Initialize Editor with Content
    │
    ├──→ Connect to Collaboration Socket (page room)
    │
    └──→ Set up Auto-save
            │
            └──→ Periodic Save to Backend
```

## Key Design Patterns

### 1. **Container/Presenter Pattern**

- **Presenters**: UI components in `components/` (RichTextEditor, KanbanBoard, etc.)
- **Containers**: Pages in `page/` that orchestrate data fetching and state management

### 2. **Custom Hooks Pattern**

- `useCollaboration`: Manages Socket.IO connections and real-time updates
- `useComments`: Handles comment fetching and mutations
- `useWorkspaceId`: Extracts workspace ID from route params
- API hooks: `useGetWorkspaceMembers`, `useCreatePage`, etc.

### 3. **Context Pattern**

- `AuthProvider`: Global authentication state
- `NotificationProvider`: Real-time notification management
- `ThemeProvider`: Dark/light mode theming

### 4. **Service Layer Pattern (Backend)**

- Controllers handle HTTP requests/responses
- Services contain business logic
- Models define data structures
- Clear separation allows for easier testing and maintenance

## Technology Stack

### Frontend

- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite
- **Routing**: React Router v7
- **State Management**:
  - TanStack Query (Server state)
  - Zustand (Client state)
  - React Context (Auth, Notifications, Theme)
- **UI Library**: shadcn/ui (Radix UI primitives)
- **Styling**: Tailwind CSS
- **Editor**: TipTap (ProseMirror-based)
- **Real-time**: Socket.IO Client
- **Forms**: React Hook Form + Zod validation

### Backend

- **Runtime**: Node.js with TypeScript
- **Framework**: Express.js
- **Database**: MongoDB with Mongoose
- **Authentication**: Passport.js (Google OAuth, Local)
- **Real-time**: Socket.IO Server
- **Validation**: Zod
- **Session**: cookie-session

## Component Dependencies

### Core Dependencies

```
RichTextEditor
    ├── depends on → useCollaboration
    ├── depends on → useComments
    ├── depends on → EditorToolbar
    ├── depends on → LiveCursors
    ├── depends on → PresenceIndicator
    └── depends on → CommentOverlay

useCollaboration
    ├── depends on → Socket.IO Client
    ├── depends on → useGetWorkspaceMembers
    └── provides → Collaboration API

PageEditor
    ├── depends on → RichTextEditor
    ├── depends on → TanStack Query
    └── depends on → useParams (React Router)
```

## State Management Flow

```
┌─────────────────────────────────────────┐
│         Server State (TanStack Query)   │
│  - Pages, Projects, Tasks, Workspaces   │
│  - Automatic Caching & Refetching       │
└─────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│         Client State (Zustand/Context)  │
│  - UI State (Modals, Dialogs)           │
│  - Auth State (Current User)            │
│  - Theme State                          │
└─────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│         Real-time State (Socket.IO)     │
│  - Collaboration Updates                │
│  - Presence Information                 │
│  - Live Cursors                         │
└─────────────────────────────────────────┘
```
