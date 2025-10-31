# Editor Structure and Collaboration Logic

## Overview

The Stateflow editor is built on **TipTap**, a modern, extensible rich text editor framework based on ProseMirror. It provides real-time collaborative editing capabilities through Socket.IO, enabling multiple users to edit pages simultaneously with live cursors, presence indicators, and conflict-free updates.

## Editor Architecture

### Core Components

#### 1. RichTextEditor (`rich-text-editor.tsx`)

The main editor component that orchestrates all editing functionality.

**Key Responsibilities:**

- Initializes and manages the TipTap editor instance
- Handles editor content updates and synchronization
- Integrates collaboration features (cursors, presence, document updates)
- Manages AI features (autocomplete, floating menu)
- Handles voice input, comments, and whiteboard features

**Props:**

```typescript
interface RichTextEditorProps {
  content?: string; // Initial HTML content
  onUpdate?: (html: string) => void; // Callback when content changes
  onSave?: (html: string) => void; // Manual save callback
  className?: string;
  editable?: boolean; // Enable/disable editing
  autoSave?: boolean; // Auto-save enabled
  autoSaveDelay?: number; // Delay in ms (default: 2000)
  pageId?: string; // Required for collaboration
  enableCollaboration?: boolean; // Enable real-time collaboration
  placeholder?: string;
}
```

#### 2. TipTap Extensions

The editor uses several TipTap extensions to provide rich functionality:

**Built-in Extensions:**

- **StarterKit**: Core formatting (bold, italic, headings, lists, blockquotes, code blocks)
- **Link**: Hyperlink support
- **Image**: Image embedding
- **Table**: Full table support with resizable columns
- **TaskList & TaskItem**: Checkbox task lists
- **Mention**: @mention support for users

**Custom Extensions:**

- **SlashCommand**: `/` command palette for quick formatting
- **Comment**: Inline commenting system
- **AIFloatingMenu**: AI actions on text selection
- **AIAutocomplete**: Inline AI suggestions

### Editor Initialization Flow

```typescript
const editor = useEditor({
  extensions: [
    /* all extensions */
  ],
  content, // Initial content
  editable, // Editing mode
  onUpdate: ({ editor }) => {
    const html = editor.getHTML();
    onUpdate?.(html);

    // Send update to collaborators
    if (enableCollaboration) {
      collaboration.sendDocumentUpdate({
        type: "update",
        content: html,
      });
    }
  },
  onSelectionUpdate: ({ editor }) => {
    // Broadcast cursor position
    if (enableCollaboration) {
      collaboration.sendCursorUpdate({
        // cursor position data
      });
    }
  },
});
```

## Collaboration System

### Architecture Overview

The collaboration system uses **Socket.IO** for real-time bidirectional communication between clients and the server. The system is designed to support:

1. **Document Synchronization**: Real-time content updates across all connected clients
2. **Presence Tracking**: See who's currently viewing/editing a page
3. **Live Cursors**: Visual indicators showing where other users are editing
4. **Conflict Resolution**: Automatic handling of simultaneous edits

### Collaboration Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Client (Browser 1)                       │
│                                                             │
│  User Types                                                 │
│      │                                                      │
│      ▼                                                      │
│  Editor.onUpdate()                                          │
│      │                                                      │
│      ├──→ Extract HTML content                              │
│      │                                                      │
│      └──→ collaboration.sendDocumentUpdate({ content })     │
│                  │                                          │
│                  ▼                                          │
│          Socket.IO Client                                   │
│                  │                                          │
│                  │ emit('document-update', {                │
│                  │   pageId,                                │
│                  │   update: { content }                    │
│                  │ })                                       │
└──────────────────┼──────────────────────────────────────────
                   │
                   │ WebSocket
                   ▼
┌─────────────────────────────────────────────────────────────
│              Backend Collaboration Service                   │
│                                                              │
│  Socket.IO Server                                            │
│      │                                                       │
│      ├──→ Join page room: socket.join(`page:${pageId}`)      │
│      │                                                       │
│      ├──→ Receive document-update event                      │
│      │   │                                                   │
│      │   ├──→ Validate update                                │
│      │   │                                                   │
│      │   ├──→ Broadcast to other users in room               │
│      │   │   socket.to(`page:${pageId}`).emit(...)           │
│      │   │                                                   │
│      │   └──→ Detect mentions & trigger notifications         │
│      │                                                       │
│      └──→ Handle cursor/selection updates                    │
│          (similar broadcast pattern)                         │
└──────────────────┼───────────────────────────────────────────┘
                   │
                   │ WebSocket (broadcast)
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    Client (Browser 2)                       │
│                                                             │
│  Socket.IO Client                                           │
│      │                                                      │
│      ├──→ Listen: socket.on('document-update')              │
│      │   │                                                  │
│      │   └──→ Extract update data                           │
│      │       │                                              │
│      │       └──→ Emit CustomEvent('collaboration-update')  │
│      │           │                                          │
│      │           ▼                                          │
│      │   RichTextEditor                                     │
│      │       │                                              │
│      │       ├──→ Listen: window.addEventListener(          │
│      │       │       'collaboration-update',                │
│      │       │       handleCollaborationUpdate              │
│      │       │   )                                          │
│      │       │                                              │
│      │       └──→ Apply update to editor                    │
│      │           editor.commands.setContent(newContent)     │
│      │                                                      │
│      └──→ Update LiveCursors component                      │
│          (show other users' cursors)                        │
└─────────────────────────────────────────────────────────────┘
```

### useCollaboration Hook

The `useCollaboration` hook manages all collaboration functionality:

**Key Features:**

```typescript
export const useCollaboration = ({ workspaceId, pageId, token }) => {
  // State
  const [socket, setSocket] = useState<Socket | null>(null);
  const [connectedUsers, setConnectedUsers] = useState<User[]>([]);
  const [cursors, setCursors] = useState<Map<string, CursorPosition>>(
    new Map()
  );
  const [isConnected, setIsConnected] = useState(false);

  // Methods
  return {
    // Connection management
    isConnected,
    connectedUsers,

    // Document updates
    sendDocumentUpdate: (update) => {
      socket.emit("document-update", {
        pageId,
        update,
      });
    },

    // Cursor tracking
    sendCursorUpdate: (cursor) => {
      socket.emit("cursor-update", {
        pageId,
        cursor,
      });
    },

    // Selection tracking
    sendSelectionUpdate: (selection) => {
      socket.emit("selection-update", {
        pageId,
        selection,
      });
    },

    // Live cursors for display
    cursors,
  };
};
```

### Connection Flow

1. **Socket Connection**

   ```typescript
   const socket = io(VITE_API_URL, {
     auth: { token }, // JWT token for authentication
     transports: ["websocket", "polling"],
     reconnection: true,
   });
   ```

2. **Authentication**

   - Token is sent in the `auth` object
   - Backend middleware validates the token
   - User information is attached to the socket connection

3. **Room Joining**

   ```typescript
   // Join workspace room (for presence)
   socket.emit("join-workspace", { workspaceId });

   // Join page room (for document collaboration)
   if (pageId) {
     socket.emit("join-page", { pageId });
   }
   ```

4. **Presence Updates**
   - Server broadcasts user join/leave events
   - Client updates `connectedUsers` state
   - `PresenceIndicator` component displays active users

### Document Update Mechanism

#### Sending Updates

When a user types, the editor's `onUpdate` handler:

1. Extracts HTML content from the editor
2. Calls `collaboration.sendDocumentUpdate({ content })`
3. Socket.IO emits `document-update` event to the server

**Note**: The system uses a flag `isApplyingRemoteUpdate` to prevent feedback loops:

```typescript
if (!isApplyingRemoteUpdate.current) {
  collaboration.sendDocumentUpdate({ content });
}
```

#### Receiving Updates

When another user's update arrives:

1. Socket.IO client receives `document-update` event
2. Checks if update is from self (using socket ID or user ID)
3. If from another user, dispatches a custom event:
   ```typescript
   window.dispatchEvent(
     new CustomEvent("collaboration-update", {
       detail: { content, userId, timestamp, socketId },
     })
   );
   ```
4. Editor listens for this event and applies the update:
   ```typescript
   window.addEventListener("collaboration-update", (event) => {
     isApplyingRemoteUpdate.current = true;
     editor.commands.setContent(newContent);
     setTimeout(() => {
       isApplyingRemoteUpdate.current = false;
     }, 100);
   });
   ```

### Cursor and Selection Tracking

#### Cursor Position Updates

- Triggered on mouse movement within the editor
- Sent via `collaboration.sendCursorUpdate({ x, y })`
- Backend broadcasts to other users in the same page room
- `LiveCursors` component renders other users' cursors with avatars

#### Selection Updates

- Triggered on text selection changes
- Includes selection range (from, to) and selected text
- Used for showing what other users are highlighting

### Conflict Resolution

The current implementation uses a **last-write-wins** strategy with HTML content synchronization:

1. **Optimistic Updates**: Local changes appear immediately
2. **Broadcast**: Changes are sent to all connected clients
3. **Merge on Receive**: When receiving an update, the entire HTML is replaced
4. **No Operational Transforms**: The system doesn't use OT or CRDTs (YJS is installed but not actively used)

**Limitations:**

- Simultaneous edits may result in one user's changes overwriting another's
- No fine-grained conflict resolution at the character level

### Presence System

#### How It Works

1. **User Joins**

   - Client emits `join-workspace` event
   - Server adds user to workspace room
   - Server broadcasts `user-joined` to other users

2. **User Leaves**

   - Socket disconnects (handled automatically)
   - Server removes user from rooms
   - Server broadcasts `user-left` event

3. **Presence Indicator**
   - Displays avatars of currently active users
   - Shows user names on hover
   - Updates in real-time as users join/leave

### Comment System

The editor includes an inline commenting system:

1. **Comment Extension**: Custom TipTap extension for comment marks
2. **Comment Overlay**: UI overlay showing comment bubbles
3. **Comments Panel**: Side panel for managing all comments
4. **API Integration**: Comments stored via REST API, synchronized via Socket.IO

**Comment Flow:**

```
User selects text
    │
    ▼
Right-click or comment button
    │
    ▼
Create comment mark in editor
    │
    ├──→ Save comment via API
    │   └──→ Store in MongoDB
    │
    └──→ Broadcast comment creation via Socket.IO
            └──→ Other users see comment bubble
```

### Backend Collaboration Service

Located in `backend/src/services/collaboration.service.ts`, the `CollaborationServer` class:

**Responsibilities:**

- Manages Socket.IO server instance
- Handles socket authentication
- Manages room memberships (workspace rooms, page rooms)
- Broadcasts document updates, cursor updates, selection updates
- Detects mentions in content and triggers notifications
- Emits activity events for activity feed

**Key Methods:**

```typescript
class CollaborationServer {
  // Room management
  private handleJoinWorkspace(socket, workspaceId);
  private handleJoinPage(socket, pageId);

  // Document collaboration
  private handleDocumentUpdate(socket, data);
  private detectMentions(content, pageId, userId, userName);

  // Presence
  private handleUserJoined(socket, workspaceId);
  private handleUserLeft(socket, workspaceId);

  // Activity feed
  public emitActivityEvent(workspaceId, activity);
}
```

## Editor Features

### AI Integration

1. **AI Floating Menu**

   - Appears when text is selected
   - Options: Improve, Summarize, Expand, Rewrite
   - Calls backend AI service (Gemini API)

2. **AI Autocomplete**
   - Optional inline suggestions
   - Triggered by typing
   - Can be enabled/disabled

### Voice Input

- Uses Web Speech API (SpeechRecognition)
- Converts speech to text
- Inserts text at current cursor position
- Browser compatibility: Chrome, Edge (not Firefox/Safari)

### Whiteboard

- Separate modal for collaborative drawing
- Socket.IO for stroke synchronization
- Multiple users can draw simultaneously
- Supports different colors and brush sizes

## Performance Considerations

### Optimizations

1. **Debouncing**: Document updates are debounced to reduce network traffic
2. **Auto-save**: Configurable delay (default: 2 seconds) before saving
3. **Selective Updates**: Only sends updates when content actually changes
4. **Socket ID Filtering**: Prevents processing own updates

### Scalability

- Current implementation broadcasts to all users in a page room
- For large numbers of concurrent editors, consider:
  - Operational Transform (OT) or CRDT (YJS)
  - Delta updates instead of full HTML
  - Rate limiting on updates
  - Message queuing for high-frequency updates

## Security

1. **Authentication**: Socket connections require valid JWT token
2. **Authorization**: Users can only join rooms for workspaces/pages they have access to
3. **Input Validation**: Content is validated before broadcasting
4. **Mention Detection**: Prevents spam notifications

## Future Improvements

See `known-limitations-future-improvements.md` for planned enhancements to the collaboration system.
