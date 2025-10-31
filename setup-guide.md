# Setup Guide - Local Development

This guide will walk you through setting up Stateflow for local development on your machine.

## Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js** 18+ (LTS version recommended)
- **npm** or **pnpm** (package manager)
- **MongoDB** (local installation or MongoDB Atlas account)
- **Git** (for cloning the repository)

### Verify Installation

```bash
node --version  # Should be v18.x.x or higher
npm --version    # or pnpm --version
mongod --version # If using local MongoDB
```

## Project Structure

```
froncort/
├── froncort/
│   ├── backend/    # Node.js/Express API server
│   └── client/     # React frontend application
└── docs/           # Documentation (this folder)
```

## Step 1: Clone and Navigate

```bash
# If you haven't already cloned the repository
git clone <repository-url>
cd froncort

# Navigate to project root
cd froncort
```

## Step 2: Backend Setup

### 2.1 Install Dependencies

```bash
cd backend
npm install
# or
pnpm install
```

### 2.2 Set Up MongoDB

You have two options:

#### Option A: Local MongoDB

1. Install MongoDB Community Edition (if not already installed)
2. Start MongoDB service:
   ```bash
   # macOS
   brew services start mongodb-community
   
   # Linux
   sudo systemctl start mongod
   
   # Windows
   # Start MongoDB service from Services panel
   ```
3. Verify MongoDB is running:
   ```bash
   mongosh
   # Should connect to mongodb://localhost:27017
   ```

#### Option B: MongoDB Atlas (Cloud)

1. Create a free account at [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Create a new cluster
3. Get your connection string (e.g., `mongodb+srv://username:password@cluster.mongodb.net/dbname`)

### 2.3 Configure Environment Variables

Create a `.env` file in the `backend/` directory:

```bash
cd backend
touch .env
```

Add the following variables to `backend/.env`:

```env
# Environment
NODE_ENV=development
PORT=8000
BASE_PATH=/api

# MongoDB
# For local MongoDB:
MONGO_URI=mongodb://localhost:27017/stateflow
# OR for MongoDB Atlas:
# MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/stateflow

# Sessions & JWT
SESSION_SECRET=your-secret-session-key-change-this
SESSION_EXPIRES_IN=7d
JWT_SECRET=your-secret-jwt-key-change-this

# Google OAuth (Optional - for Google Sign-In)
GOOGLE_CLIENT_ID=your-google-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_CALLBACK_URL=http://localhost:8000/api/auth/google/callback

# Frontend URLs (for CORS and Socket.IO)
FRONTEND_ORIGIN=localhost
FRONTEND_GOOGLE_CALLBACK_URL=http://localhost:5173
CLIENT_URL=http://localhost:5173

# AI Features (Optional - for AI autocomplete and suggestions)
GEMINI_API_KEY=your-gemini-api-key
```

**Important Notes:**
- Replace placeholder values with your actual credentials
- Generate secure random strings for `SESSION_SECRET` and `JWT_SECRET`
- For Google OAuth, see [Google OAuth Setup](#google-oauth-setup) section below
- `CLIENT_URL` is critical for Socket.IO connections - ensure it matches your frontend URL

### 2.4 Seed Database (Optional)

Seed default roles for the application:

```bash
npm run seed
# or
pnpm seed
```

This creates default roles (Admin, Member, Viewer) in the database.

### 2.5 Start Backend Server

```bash
# Development mode (with hot reload)
npm run dev
# or
pnpm dev

# The server should start on http://localhost:8000
# API endpoints will be available at http://localhost:8000/api
```

You should see:
```
Server listening on port 8000 in development
WebSocket collaboration server initialized
Database connected successfully
```

## Step 3: Frontend Setup

### 3.1 Install Dependencies

Open a new terminal window and navigate to the client directory:

```bash
cd client
npm install
# or
pnpm install
```

### 3.2 Configure Environment Variables

Create a `.env` file in the `client/` directory:

```bash
cd client
touch .env
```

Add the following variables to `client/.env`:

```env
# Backend API URL
VITE_API_BASE_URL=http://localhost:8000/api
VITE_API_URL=http://localhost:8000

# Optional: Google Client ID (if you want Google Sign-In buttons)
# VITE_GOOGLE_CLIENT_ID=your-google-client-id.apps.googleusercontent.com
```

**Notes:**
- `VITE_API_BASE_URL` is used for REST API calls
- `VITE_API_URL` is used for Socket.IO connections
- Ensure these URLs match your backend server configuration

### 3.3 Start Frontend Development Server

```bash
npm run dev
# or
pnpm dev
```

The frontend should start on `http://localhost:5173` (Vite default port).

You should see:
```
VITE v6.x.x  ready in xxx ms

➜  Local:   http://localhost:5173/
➜  Network: use --host to expose
```

## Step 4: Verify Installation

1. **Backend**: Open `http://localhost:8000` - you should see an error message (this is expected - the root endpoint throws an error intentionally)

2. **Frontend**: Open `http://localhost:5173` - you should see the Stateflow login/signup page

3. **Test Connection**:
   - Try creating an account or logging in
   - Check browser console and backend terminal for any errors

## Google OAuth Setup (Optional)

If you want to enable Google Sign-In:

### 1. Create Google OAuth Credentials

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (or select an existing one)
3. Enable **Google+ API**
4. Go to **Credentials** → **Create Credentials** → **OAuth 2.0 Client ID**
5. Configure consent screen (if prompted):
   - User type: External
   - App name: Stateflow (or your preferred name)
   - Support email: your email
   - Add scopes: `email`, `profile`
6. Create OAuth Client ID:
   - Application type: Web application
   - Authorized redirect URIs:
     - `http://localhost:8000/api/auth/google/callback` (backend)
     - `http://localhost:5173` (frontend, if needed)
7. Copy the **Client ID** and **Client Secret**

### 2. Update Environment Variables

Add the credentials to `backend/.env`:
```env
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_CALLBACK_URL=http://localhost:8000/api/auth/google/callback
FRONTEND_GOOGLE_CALLBACK_URL=http://localhost:5173
```

### 3. Restart Backend

After updating environment variables, restart the backend server.

## Troubleshooting

### Backend Issues

#### Port Already in Use
```
Error: listen EADDRINUSE: address already in use :::8000
```
**Solution**: Either stop the process using port 8000, or change `PORT` in `backend/.env`

#### MongoDB Connection Failed
```
MongooseError: connection error
```
**Solutions**:
- Ensure MongoDB is running (check `mongosh` connection)
- Verify `MONGO_URI` in `.env` is correct
- For Atlas: Check IP whitelist and connection string
- Check firewall settings

#### Socket.IO CORS Errors
```
Access to XMLHttpRequest blocked by CORS policy
```
**Solutions**:
- Ensure `CLIENT_URL` in `backend/.env` matches frontend URL
- Check that `FRONTEND_ORIGIN` is set correctly
- Clear browser cache and restart both servers

### Frontend Issues

#### API Connection Failed
```
Network Error or CORS error
```
**Solutions**:
- Verify backend is running on `http://localhost:8000`
- Check `VITE_API_BASE_URL` in `client/.env`
- Ensure backend CORS is configured correctly

#### Socket.IO Connection Failed
```
Socket.IO connection error
```
**Solutions**:
- Verify `VITE_API_URL` in `client/.env` matches backend URL
- Check backend `CLIENT_URL` environment variable
- Ensure backend Socket.IO server is initialized
- Check browser console for specific error messages

#### Module Not Found Errors
```
Cannot find module 'xxx' or '@xxx'
```
**Solution**: Run `npm install` or `pnpm install` again

### General Issues

#### Changes Not Reflecting
- Clear browser cache
- Restart both frontend and backend servers
- Check for TypeScript compilation errors

#### TypeScript Errors
```bash
# Backend
cd backend
npm run build  # Check for compilation errors

# Frontend
cd client
npm run build  # Check for compilation errors
```

## Development Workflow

### Running Both Servers

You'll need two terminal windows/tabs:

**Terminal 1 - Backend:**
```bash
cd froncort/backend
npm run dev
```

**Terminal 2 - Frontend:**
```bash
cd froncort/client
npm run dev
```

### Hot Reload

Both servers support hot reload:
- **Backend**: Uses `ts-node-dev` - automatically restarts on file changes
- **Frontend**: Vite HMR - updates without full page reload

### Building for Production

**Backend:**
```bash
cd backend
npm run build  # Compiles TypeScript to dist/
npm start      # Runs compiled JavaScript
```

**Frontend:**
```bash
cd client
npm run build  # Creates production build in dist/
npm run preview  # Preview production build locally
```

## Environment Variables Summary

### Backend (`.env` in `backend/`)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `NODE_ENV` | No | `development` | Environment mode |
| `PORT` | No | `5000` | Server port |
| `BASE_PATH` | No | `/api` | API base path |
| `MONGO_URI` | **Yes** | - | MongoDB connection string |
| `SESSION_SECRET` | **Yes** | - | Session encryption secret |
| `SESSION_EXPIRES_IN` | No | `7d` | Session expiration |
| `JWT_SECRET` | **Yes** | - | JWT token secret |
| `GOOGLE_CLIENT_ID` | Optional | - | Google OAuth client ID |
| `GOOGLE_CLIENT_SECRET` | Optional | - | Google OAuth secret |
| `GOOGLE_CALLBACK_URL` | Optional | - | Google OAuth callback |
| `FRONTEND_ORIGIN` | No | `localhost` | Frontend origin for CORS |
| `CLIENT_URL` | **Yes** | - | Frontend URL for Socket.IO |
| `GEMINI_API_KEY` | Optional | - | Google Gemini API key |

### Frontend (`.env` in `client/`)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `VITE_API_BASE_URL` | **Yes** | - | Backend API base URL |
| `VITE_API_URL` | **Yes** | - | Backend URL for Socket.IO |
| `VITE_GOOGLE_CLIENT_ID` | Optional | - | Google OAuth client ID |

## Next Steps

Once everything is set up:

1. Create your first account
2. Create a workspace
3. Invite team members
4. Start creating pages and tasks
5. Explore collaborative editing features

For more information, see:
- [Component Architecture](./component-architecture.md)
- [Editor Structure and Collaboration](./editor-structure-collaboration.md)
- [Known Limitations](./known-limitations-future-improvements.md)

## Getting Help

If you encounter issues not covered in this guide:

1. Check the browser console for errors
2. Check backend terminal logs
3. Verify all environment variables are set correctly
4. Ensure MongoDB is running and accessible
5. Check that ports 8000 and 5173 are not in use by other applications

