# Full-Stack Developer Assessment: Geofencing & Real-time Alert System

## 📋 Overview

This assessment evaluates your ability to build a complete full-stack application with real-time capabilities. You will develop:

1. **Backend API (Go)** - RESTful API for geofencing and vehicle tracking
2. **WebSocket Server (Go)** - Real-time alert notification system
3. **Frontend Application (React)** - Web interface to interact with the system and receive live alerts

## 🎯 Objective

Build a complete geofencing and vehicle tracking system that:
- Manages geofences (virtual boundaries) and vehicle locations
- Tracks when vehicles enter or exit geofenced areas
- Sends real-time alerts via WebSockets
- Provides a user-friendly web interface for system interaction

## 🛠️ Technology Stack

### Backend
- **Language**: GoLang
- **Database**: PostgreSQL with PostGIS extension
- **Real-time**: WebSocket server for live alerts

### Frontend
- **Framework**: React (you may use Next.js, Create React App, or Vite)
- **Styling**: Your choice (Tailwind CSS, Material-UI, CSS Modules, etc.)
- **WebSocket Client**: Native WebSocket API or libraries like socket.io-client

### Infrastructure
- **Containerization**: Docker & Docker Compose
- **Deployment**: Your choice of platform (Render, Railway, Vercel, AWS, etc.)

---

## 📡 Backend Requirements

### API Response Format

**IMPORTANT**: Every API response must include execution time in nanoseconds:

```json
{
  // ... response data
  "time_ns": "1234567"
}
```

Measure time using Go's `time` package at the start and end of each request handler.

---

## 🔌 API Endpoints

> **Note**: Do not modify endpoint paths, HTTP methods, or naming conventions. Assessment evaluation is strict on endpoint structure.

### 1. POST /geofences
Create a new geofence with polygonal boundaries.

**Request Body**:
```json
{
  "name": "Downtown Delivery Zone",
  "description": "Main delivery area for downtown customers",
  "coordinates": [
    [37.7749, -122.4194],
    [37.7849, -122.4194],
    [37.7849, -122.4094],
    [37.7749, -122.4094],
    [37.7749, -122.4194]
  ],
  "category": "delivery_zone"
}
```

**Validation Rules**:
- `coordinates`: Array of `[latitude, longitude]` pairs
- First and last coordinates must be identical (closed polygon)
- Minimum 4 points (3 unique + 1 closing point)
- `category` options: `delivery_zone`, `restricted_zone`, `toll_zone`, `customer_area`

**Response**:
```json
{
  "id": "geo_123",
  "name": "Downtown Delivery Zone",
  "status": "active",
  "time_ns": "1234567"
}
```

---

### 2. GET /geofences
Retrieve all geofences with optional filtering.

**Query Parameters**:
- `category` (optional): Filter by geofence category

**Example**: `GET /geofences?category=delivery_zone`

**Response**:
```json
{
  "geofences": [
    {
      "id": "geo_123",
      "name": "Downtown Delivery Zone",
      "description": "Main delivery area for downtown customers",
      "coordinates": [[37.7749, -122.4194], ...],
      "category": "delivery_zone",
      "created_at": "2025-01-15T10:30:00Z"
    }
  ],
  "time_ns": "987654"
}
```

---

### 3. POST /vehicles
Register a new vehicle in the system.

**Request Body**:
```json
{
  "vehicle_number": "KA-01-AB-1234",
  "driver_name": "John Doe",
  "vehicle_type": "truck",
  "phone": "+1234567890"
}
```

**Response**:
```json
{
  "id": "veh_456",
  "vehicle_number": "KA-01-AB-1234",
  "status": "active",
  "time_ns": "1123456"
}
```

---

### 4. GET /vehicles
Retrieve all registered vehicles.

**Response**:
```json
{
  "vehicles": [
    {
      "id": "veh_456",
      "vehicle_number": "KA-01-AB-1234",
      "driver_name": "John Doe",
      "vehicle_type": "truck",
      "phone": "+1234567890",
      "status": "active",
      "created_at": "2025-01-15T09:00:00Z"
    }
  ],
  "time_ns": "876543"
}
```

---

### 5. POST /vehicles/location
Update vehicle location and check geofence status.

**Request Body**:
```json
{
  "vehicle_id": "veh_456",
  "latitude": 37.7849,
  "longitude": -122.4194,
  "timestamp": "2025-01-15T10:35:00Z"
}
```

**Response**:
```json
{
  "vehicle_id": "veh_456",
  "location_updated": true,
  "current_geofences": [
    {
      "geofence_id": "geo_123",
      "geofence_name": "Downtown Delivery Zone",
      "status": "inside"
    }
  ],
  "time_ns": "2345678"
}
```

**Business Logic**:
- Store location update in database
- Use PostGIS to check if vehicle is inside any geofences
- Detect entry/exit events (compare with previous location state)
- Trigger WebSocket alerts for configured events
- Return all current geofences containing the vehicle

---

### 6. GET /vehicles/location/{vehicle_id}
Get current location and geofence status for a specific vehicle.

**Response**:
```json
{
  "vehicle_id": "veh_456",
  "vehicle_number": "KA-01-AB-1234",
  "current_location": {
    "latitude": 37.7849,
    "longitude": -122.4194,
    "timestamp": "2025-01-15T10:35:00Z"
  },
  "current_geofences": [
    {
      "geofence_id": "geo_123",
      "geofence_name": "Downtown Delivery Zone",
      "category": "delivery_zone"
    }
  ],
  "time_ns": "876543"
}
```

---

### 7. POST /alerts/configure
Configure WebSocket alerts for geofence events.

**Request Body**:
```json
{
  "geofence_id": "geo_123",
  "vehicle_id": "veh_456",
  "event_type": "entry",
  "websocket_url": "ws://example.com/alerts"
}
```

**Configuration Rules**:
- `event_type` options: `entry`, `exit`, `both`
- If `vehicle_id` is omitted, alert applies to all vehicles
- `websocket_url`: WebSocket endpoint to send alerts (optional, for external integrations)

**Response**:
```json
{
  "alert_id": "alert_789",
  "geofence_id": "geo_123",
  "vehicle_id": "veh_456",
  "event_type": "entry",
  "status": "active",
  "time_ns": "1567890"
}
```

---

### 8. GET /alerts
Retrieve all configured alert rules.

**Query Parameters**:
- `geofence_id` (optional): Filter by geofence
- `vehicle_id` (optional): Filter by vehicle

**Response**:
```json
{
  "alerts": [
    {
      "alert_id": "alert_789",
      "geofence_id": "geo_123",
      "geofence_name": "Downtown Delivery Zone",
      "vehicle_id": "veh_456",
      "vehicle_number": "KA-01-AB-1234",
      "event_type": "entry",
      "websocket_url": "ws://example.com/alerts",
      "status": "active",
      "created_at": "2025-01-15T09:15:00Z"
    }
  ],
  "time_ns": "654321"
}
```

---

### 9. GET /violations/history
Retrieve historical geofence entry/exit events.

**Query Parameters**:
- `vehicle_id` (optional): Filter by vehicle
- `geofence_id` (optional): Filter by geofence
- `start_date` (optional): ISO 8601 format (e.g., `2025-01-01T00:00:00Z`)
- `end_date` (optional): ISO 8601 format
- `limit` (optional): Number of records (default: 50, max: 500)

**Example**: `GET /violations/history?vehicle_id=veh_456&limit=100`

**Response**:
```json
{
  "violations": [
    {
      "id": "viol_111",
      "vehicle_id": "veh_456",
      "vehicle_number": "KA-01-AB-1234",
      "geofence_id": "geo_123",
      "geofence_name": "Downtown Delivery Zone",
      "event_type": "entry",
      "latitude": 37.7849,
      "longitude": -122.4194,
      "timestamp": "2025-01-15T10:35:00Z"
    }
  ],
  "total_count": 245,
  "time_ns": "3456789"
}
```

---

## 🔴 WebSocket Server Requirements

### WebSocket Endpoint: `/ws/alerts`

Implement a WebSocket server that:

1. **Accepts connections** from frontend clients
2. **Broadcasts real-time alerts** when geofence events occur
3. **Supports multiple concurrent connections**
4. **Handles connection lifecycle** (connect, disconnect, reconnect)

### WebSocket Message Format

When a vehicle enters/exits a geofence, send this JSON to all connected clients:

```json
{
  "event_id": "evt_999",
  "event_type": "entry",
  "timestamp": "2025-01-15T10:35:00Z",
  "vehicle": {
    "vehicle_id": "veh_456",
    "vehicle_number": "KA-01-AB-1234",
    "driver_name": "John Doe"
  },
  "geofence": {
    "geofence_id": "geo_123",
    "geofence_name": "Downtown Delivery Zone",
    "category": "delivery_zone"
  },
  "location": {
    "latitude": 37.7849,
    "longitude": -122.4194
  }
}
```

### WebSocket Implementation Requirements

- Use Gorilla WebSocket library or similar
- Implement connection pooling/management
- Handle graceful disconnections
- Support message broadcasting to all connected clients
- Optionally support filtering (e.g., alerts for specific vehicles/geofences)

### Integration with Location Updates

When `POST /vehicles/location` detects an entry/exit event:
1. Store the event in database
2. Check for configured alerts matching the event
3. **Immediately broadcast** the alert to all connected WebSocket clients

---

## 🎨 Frontend Requirements

Build a React-based web application with the following features:

### Core Features

#### 1. **Geofence Management**
- Form to create new geofences
  - Input fields for name, description, category
  - Coordinate input (can be textarea with JSON array or point-by-point form)
  - Visual feedback on successful creation
- Display list of all geofences
- Filter geofences by category

#### 2. **Vehicle Management**
- Form to register new vehicles
  - Fields: vehicle number, driver name, vehicle type, phone
- Display list of all vehicles
- View current location and status of each vehicle

#### 3. **Location Updates**
- Form to update vehicle location
  - Select vehicle from dropdown
  - Input latitude and longitude
  - Optional: Add map interface with click-to-select location
- Show immediate feedback on which geofences the vehicle is in

#### 4. **Alert Configuration**
- Form to configure alerts
  - Select geofence
  - Select vehicle (or "all vehicles")
  - Choose event type (entry/exit/both)
- Display all configured alerts
- Option to delete/disable alerts

#### 5. **Real-time Alert Notifications** ⭐
- **Connect to WebSocket server** on component mount
- **Display incoming alerts** in real-time using:
  - Toast notifications
  - Alert banner
  - Dedicated alerts panel/feed
- Show alert details: vehicle info, geofence name, event type, timestamp
- Visual/audio notification for new alerts (optional but nice to have)

#### 6. **Violation History**
- Display historical geofence events
- Filter by vehicle, geofence, date range
- Pagination or infinite scroll for large datasets
- Show event type, timestamp, location

#### 7. **Dashboard/Overview** (Optional but Recommended)
- Summary statistics (total vehicles, geofences, recent alerts)
- Recent activity feed
- Status indicators

#### UI/UX Guidelines
- Clean, intuitive interface
- Responsive design (works on desktop and mobile)
- Loading states for API calls
- Error handling with user-friendly messages
- Form validation
- Clear visual hierarchy

#### State Management
- Use React hooks (useState, useEffect, useContext)
- Or Redux/Zustand for complex state (optional)
- Manage WebSocket connection state

#### API Integration
- Handle loading, success, and error states
- Display API response times (optional)

---

## 📦 Submission Requirements

### 1. Code Repository (GitHub)
- Push your complete source code to GitHub
- Include both backend and frontend code
- Add the following as collaborators (**do not make repo public**):
  - `vedantp@mapup.ai`
  - `ajayap@mapup.ai`
  - `atharvd@mapup.ai`

### 2. Docker Hub
- Build and push Docker images

### 3. Deployment
- Deploy your application to a cloud platform:
  - Backend options: Render, Railway, Fly.io, AWS EC2, Google Cloud Run
  - Frontend options: Vercel, Netlify, Render
  - Or deploy full stack together on platforms like Railway or Render

### 4. Documentation
Include a `SETUP.md` file with:
- Prerequisites and dependencies
- Local setup instructions
- How to run with Docker Compose
- API testing guide with example curl commands
- Frontend usage guide
- Architecture overview (optional)

### 5. Submit via Google Form
Provide the following:
- GitHub repository link
- Docker Hub links (backend and frontend)
- Deployed API base URL
- Deployed frontend URL
- Brief description of your tech choices

---

## 🎯 Evaluation Criteria

### Functionality (40%)
- ✅ All API endpoints working correctly
- ✅ Accurate geospatial calculations using PostGIS
- ✅ WebSocket alerts broadcasting in real-time
- ✅ Frontend properly integrated with backend
- ✅ Real-time notifications working in UI
- ✅ Proper handling of edge cases

### Code Quality (25%)
- ✅ Clean, readable, and well-organized code
- ✅ Go best practices and idioms
- ✅ React best practices (hooks, component structure)
- ✅ Proper error handling throughout
- ✅ Meaningful variable and function names
- ✅ Code comments where necessary

### Performance (15%)
- ✅ Efficient geospatial queries
- ✅ Proper database indexing
- ✅ Fast location update processing
- ✅ WebSocket connection management
- ✅ Frontend performance (render optimization)

### User Experience (10%)
- ✅ Intuitive and user-friendly interface
- ✅ Responsive design
- ✅ Clear visual feedback
- ✅ Proper error messages
- ✅ Real-time alert notifications work smoothly

### Dockerization & Deployment (5%)
- ✅ Clean Dockerfiles
- ✅ Working docker-compose setup
- ✅ Easy to run locally
- ✅ Successfully deployed and accessible

### Database Design (5%)
- ✅ Well-structured schema
- ✅ Proper use of PostGIS
- ✅ Appropriate relationships and constraints

---

## ⚠️ Code Originality

**Your submission must be your original work.** If your code is found to be similar to another candidate's submission, both parties will be disqualified. Do not share your code with others during the assessment period.

---

## 🧪 Testing Guidelines

### Sample Test Flow

1. **Setup**
   ```bash
   docker-compose up -d
   ```

2. **Create a geofence**
   ```bash
   curl -X POST http://localhost:8080/geofences \
     -H "Content-Type: application/json" \
     -d '{
       "name": "Test Zone",
       "description": "Testing area",
       "coordinates": [[37.7749,-122.4194],[37.7849,-122.4194],[37.7849,-122.4094],[37.7749,-122.4094],[37.7749,-122.4194]],
       "category": "delivery_zone"
     }'
   ```

3. **Register a vehicle**
   ```bash
   curl -X POST http://localhost:8080/vehicles \
     -H "Content-Type: application/json" \
     -d '{
       "vehicle_number": "TEST-001",
       "driver_name": "Test Driver",
       "vehicle_type": "car",
       "phone": "+1234567890"
     }'
   ```

4. **Configure alert**
   ```bash
   curl -X POST http://localhost:8080/alerts/configure \
     -H "Content-Type: application/json" \
     -d '{
       "geofence_id": "geo_123",
       "vehicle_id": "veh_456",
       "event_type": "both"
     }'
   ```

5. **Connect to WebSocket** (in frontend or using tools like websocat)
   ```bash
   websocat ws://localhost:8080/ws/alerts
   ```

6. **Update vehicle location**
   ```bash
   curl -X POST http://localhost:8080/vehicles/location \
     -H "Content-Type: application/json" \
     -d '{
       "vehicle_id": "veh_456",
       "latitude": 37.7849,
       "longitude": -122.4194,
       "timestamp": "2025-01-15T10:35:00Z"
     }'
   ```

7. **Verify alert received** via WebSocket

8. **Check violation history**
   ```bash
   curl "http://localhost:8080/violations/history?vehicle_id=veh_456"
   ```

---


## 🌟 Bonus Points (Optional)

These are not required but will be considered favorably:

- Implement rate limiting on location update endpoints
- Add API authentication (JWT or API keys)
- Implement pagination for all list endpoints
- Add unit tests for critical functions
- Create a simple dashboard to visualize geofences and vehicle locations
- Implement geofence buffer zones (alert when vehicle is near boundary)
- Support for multiple polygon types (circles, rectangles, custom polygons)

---

## 📚 Resources

### Backend
- [PostGIS Documentation](https://postgis.net/documentation/)
- [Go PostgreSQL Driver (pgx)](https://github.com/jackc/pgx)
- [Gorilla WebSocket](https://github.com/gorilla/websocket)
- [Go Best Practices](https://golang.org/doc/effective_go)

### Frontend
- [React Documentation](https://react.dev/)
- [WebSocket API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- [React Hooks Guide](https://react.dev/reference/react)

### DevOps
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

---

## ❓ Questions?

If you have any questions about the assessment, please reach out to the hiring team at MapUp.

**Good luck! We're excited to see what you build! 🚀**