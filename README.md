# Transformer Thermal Inspection System

A full-stack web application designed to automate and digitize the thermal inspection workflow for power distribution transformers. This system provides a centralized platform for managing transformer data, analyzing thermal images, and generating maintenance records.

## Table of Contents

- [Project Overview](#project-overview)
- [Features by Phase](#features-by-phase)
  - [Phase 1: Transformer and Baseline Image Management](#phase-1-transformer-and-baseline-image-management)
  - [Phase 2: Automated Anomaly Detection](#phase-2-automated-anomaly-detection)
  - [Phase 3: Interactive Annotation & Feedback](#phase-3-interactive-annotation--feedback)
  - [Phase 4: Maintenance Record Sheet Generation](#phase-4-maintenance-record-sheet-generation)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Database Schema](#database-schema)
- [API Endpoints](#api-endpoints)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Supabase Setup](#1-supabase-setup)
  - [Backend Setup (Spring Boot)](#2-backend-setup-spring-boot)
  - [Frontend Setup (React)](#3-frontend-setup-react)
- [Project Structure](#project-structure)
- [Usage](#usage)
- [Contributing](#contributing)

---

## Project Overview

Power utilities conduct routine thermal inspections of distribution transformers to proactively identify potential failures such as overheating, load imbalances, or insulation degradation. The current process relies heavily on manual comparison of thermal images, which is time-consuming, subjective, and prone to human error.

This system aims to solve this by providing a robust software solution that automates anomaly detection, digitizes record-keeping, and ensures traceability. The system allows administrators to manage a database of transformers, upload baseline and maintenance thermal images, and leverage computer vision to automatically flag potential issues. This streamlines the entire inspection workflow, leading to increased efficiency, accuracy, and reliability.

The project is organized into four phases, each building upon the previous one to create a comprehensive thermal inspection management system.

---

## Features by Phase

### Phase 1: Transformer and Baseline Image Management

**Status:** ✅ Implemented

This phase establishes the foundational data management capabilities of the system.

#### FR1.1: Admin Interface for Transformer Management
- **Description:** Build an admin interface to manage transformer records
- **Implemented Features:**
  - Full CRUD (Create, Read, Update, Delete) interface for managing transformer records
  - Add new transformer records with unique ID, physical location, and power capacity
  - View all transformers in a searchable/sortable table
  - Edit existing transformer details
  - Delete transformer entries when necessary
  - Transformer metadata storage in relational database

#### FR1.2: Thermal Image Upload and Tagging
- **Description:** Enable uploading of thermal images tagged to transformers
- **Implemented Features:**
  - Upload thermal images and associate them with specific transformer records
  - Images tagged as either **Baseline** (reference image under normal conditions) or **Maintenance** (new image from routine inspection)
  - Automatic metadata recording for each image:
    - Upload date/time
    - Image type (Baseline/Maintenance)
    - Uploader information (admin user ID or name)
  - Efficient image storage using Supabase S3-compatible storage
  - Database storage of image URLs and metadata for easy retrieval

#### FR1.3: Categorization by Environmental Conditions
- **Description:** Categorize baseline images by environmental conditions
- **Implemented Features:**
  - Baseline images tagged with observed environmental condition during upload:
    - ☀️ Sunny
    - ☁️ Cloudy
    - 🌧️ Rainy
  - Dropdown selection interface for environmental condition tagging
  - Environmental condition metadata crucial for accurate comparisons in later phases

---

### Phase 2: Automated Anomaly Detection

**Status:** ✅ Implemented

This phase provides an AI-based engine that automatically detects thermal anomalies in transformer images by comparing maintenance images with baseline references.

#### FR2.1: AI-Based Anomaly Detection Engine
- **Description:** Computer vision model for comparing new maintenance images with baseline images to detect thermal anomalies
- **Implemented Features:**
  - AWS Lambda-based deep learning model for automated anomaly detection
  - Model inference triggered through backend when reference image is uploaded
  - Image conversion to Base64 format for Lambda invocation
  - Detection outputs include confidence scores and bounding box coordinates
  - Configurable thresholds for detection sensitivity:
    - Default threshold: 0.1 (confidence score)
    - IOU (Intersection over Union) threshold: 0.2 (for duplicate detection filtering)
  - Adaptive threshold support: can be customized per inspection via `threshold` parameter
  - Detected anomalies automatically assigned unique IDs and marked as "AI-generated"
  - All detections logged in anomaly log for model improvement tracking

**Technical Implementation:**
- **Lambda Integration:** Backend calls AWS Lambda endpoint (`/prod/`) with image payload
- **Coordinate Conversion:** Lambda returns bounding boxes in `[x1, y1, x2, y2]` format (top-left and bottom-right corners)
- **Automatic Coordinate Transformation:** Backend converts to `[x_center, y_center, width, height]` format for frontend storage and display
- **Anomaly Logging:** Each detection creates a log entry with detection metadata for Phase 3 feedback integration

#### FR2.2: Side-by-Side Image Comparison View
- **Description:** Interactive side-by-side display of new and baseline images with comparison controls
- **Implemented Features:**
  - Frontend displays baseline and reference (maintenance) images side by side
  - Interactive controls: zoom, pan (click and drag), reset
  - Visual highlighting of anomaly regions using bounding boxes
  - Display of detected anomalies with confidence scores
  - Responsive image comparison interface
  - Support for multiple anomaly visualizations per image

#### FR2.3: Automatic Anomaly Marking
- **Description:** Automatic annotation of images with detected anomalies and visual overlays
- **Implemented Features:**
  - Automatic visualization of detected anomalies on images
  - Color-coded or highlighted bounding boxes for anomaly regions
  - Anomaly metadata display: bounding box coordinates, severity, confidence score
  - Confidence score (0-1 range) for uncertainty flagging
  - Metadata fields: anomaly ID, detection type, class name, confidence
  - Clear visual representation of AI-detected hotspots and deviations

---

### Phase 3: Interactive Annotation & Feedback

**Status:** ✅ Implemented

This phase extends Phase 2 by enabling human-in-the-loop feedback with interactive annotation tools for validating and correcting detected anomalies.

#### FR3.1: Interactive Annotation Tools
- **Description:** Interactive tools for users to validate, correct, and annotate detected anomalies
- **Implemented Features:**
  - Adjust existing anomaly markers (resize, reposition)
  - Delete incorrectly detected anomalies
  - Add new anomaly markers by drawing bounding boxes or polygonal regions
  - Annotation metadata:
    - Annotation type (added/edited/deleted)
    - Optional comments or notes
    - Timestamp and user ID
    - "madeBy" field: distinguishes between "AI-generated" and "User" annotations
  - Intuitive and user-friendly annotation interface

**Coordinate System & Conversion:**
- **Frontend Format:** Anomalies are stored and displayed using `[x_center, y_center, width, height]` format
  - `x_center, y_center`: Center point of the anomaly bounding box
  - `width, height`: Dimensions of the bounding box
- **Backend Lambda Output Format:** AWS Lambda detection model returns coordinates as `[x1, y1, x2, y2]`
  - `x1, y1`: Top-left corner of bounding box
  - `x2, y2`: Bottom-right corner of bounding box
- **Automatic Conversion:** Backend automatically converts Lambda output to frontend format using:
  - `x_center = (x1 + x2) / 2`
  - `y_center = (y1 + y2) / 2`
  - `width = |x2 - x1|`
  - `height = |y2 - y1|`
- **User Annotations:** When users add or edit anomalies in the frontend, coordinates are sent in the storage format `[x_center, y_center, width, height]`

#### FR3.2: Metadata and Annotation Persistence
- **Description:** Storage and retrieval of all user annotations with complete metadata tracking
- **Implemented Features:**
  - Capture and save all annotation changes in the backend
  - Store comprehensive metadata:
    - User ID (who made the annotation)
    - Timestamp (when annotation was made)
    - Image ID (which inspection)
    - Transformer ID (which transformer)
    - Action taken (add/edit/delete)
  - Automatic reloading of existing annotations when revisiting an image
  - Structured and queryable annotation storage in Supabase PostgreSQL database
  - Separate anomalies list and anomalies log for tracking:
    - `anomalies`: Current state of all anomalies
    - `anomaliesLog`: Historical record of all changes for audit trail

#### FR3.3: Feedback Integration for Model Improvement
- **Description:** Feedback log system for capturing user corrections and annotations for model improvement
- **Implemented Features:**
  - Feedback log including:
    - Original AI-generated detections (with confidence scores)
    - Final user-modified annotations (validated, corrected, or rejected)
    - User comments and remarks
  - Training/validation data generation for model improvement
  - Exportable feedback log for batch training pipeline
  - Anomaly log structure:
    - Tracks all user corrections and modifications
    - Records original AI detection vs. final user decision
    - Enables identification of model weaknesses
  - **Automatic Model Retraining:** 
    - User annotations are captured in the anomaly log
    - Anomaly data can be exported for AWS Batch training job
    - Retraining triggered via `/api/retrain` endpoint
    - Backend sends fire-and-forget POST to AWS Batch training endpoint
    - Batch job processes annotated images and anomaly logs
    - Improved model deployed back to Lambda for Phase 2 detection
  - Feedback loops enable continuous model improvement from field data

---

### Phase 4: Maintenance Record Sheet Generation

**Status:** ✅ Implemented

This phase implements functionality to generate transformer-specific digital maintenance records based on thermal inspection results and user annotations.

#### FR4.1: Generate Maintenance Record Form
- **Description:** Automatic generation of pre-filled maintenance record forms for each inspection
- **Implemented Features:**
  - Auto-generated form containing:
    - Transformer metadata (ID, location, capacity)
    - Inspection timestamp
    - Thermal image thumbnail with anomaly markers (from Phase 3)
    - List of detected/annotated anomalies with metadata

#### FR4.2: Editable Engineer Input Fields
- **Description:** Editable form fields for engineers to input inspection findings and corrective actions
- **Implemented Features:**
  - Editable fields for authorized users (maintenance engineers):
    - Inspector name
    - Transformer status (OK / Needs Maintenance / Urgent Attention)
    - Electrical readings (voltage, current, etc.)
    - Recommended corrective actions
    - Additional remarks and comments
  - Form field types: text inputs, dropdowns, date pickers
  - Clear separation between system-generated and user-editable content

#### FR4.3: Save and Retrieve Completed Records
- **Description:** Storage, retrieval, and management of completed maintenance records
- **Implemented Features:**
  - Save completed maintenance records to database
  - Associate each record with specific transformer and inspection timestamp
  - Support easy retrieval, filtering, and exports
  - Record history viewer showing all past maintenance records for a transformer
  - Record versioning/timestamping for full traceability
  - PDF-ready design for printable records

---

## Architecture

The application is built on a modern, decoupled three-tier architecture, ensuring scalability and maintainability across all four phases.

### System Components

1. **Frontend (Client-Side):** A responsive and interactive user interface built with **React/Next.js**. It handles all user interactions and communicates with the backend via a REST API using Axios HTTP client.

2. **Backend (Server-Side):** A robust RESTful API developed with **Java Spring Boot**. It manages all business logic, data processing, validation, and orchestration between frontend and external services.

3. **External Services:**
   - **Supabase (Database & Storage):** PostgreSQL database and S3-compatible storage
   - **AWS Lambda:** Real-time anomaly detection inference
   - **AWS Batch:** Batch training for model improvement

### Request-Response Flow & Layer Interaction

#### Layer 1: Frontend (React/Next.js)
The frontend initiates all operations by making HTTP requests to the backend.

**Example Request Flow:**
```
Frontend Action → API Call → HTTP Request → Backend Controller
```

**Frontend Implementation Details:**
- Uses Axios to construct and send requests (GET, POST, PUT, DELETE, PATCH)
- Includes request headers with authentication tokens
- Sends request body as JSON or multipart/form-data
- Handles response parsing and error handling
- Updates component state based on response

#### Layer 2: Backend Controller (Spring Boot)
Controllers act as the entry point for all frontend requests. They receive HTTP requests, validate input, and delegate to services.

**Controller Responsibilities:**
- Extract parameters from request (path variables, query params, request body)
- Basic validation of input data
- Delegate actual business logic to service layer
- Handle errors and return appropriate HTTP status codes
- Return JSON responses to frontend

**How Controllers Work:**
1. Frontend sends HTTP request to a specific endpoint (e.g., POST /api/inspections)
2. Spring matches the request to the appropriate controller method based on @RequestMapping and @PostMapping annotations
3. Controller extracts all data from the request:
   - Path variables (e.g., {id} from `/api/inspections/{id}`)
   - Query parameters (e.g., ?status=completed)
   - Request body (JSON or form data)
4. Controller performs basic validation (e.g., checking if required fields are null)
5. If validation fails, controller returns error response (400 Bad Request)
6. If validation passes, controller calls the corresponding service method with the extracted data
7. Service processes the request and returns a result
8. Controller wraps the result in a ResponseEntity with appropriate HTTP status code
9. Response is sent back to frontend as JSON

#### Layer 3: Service Layer (Business Logic)
Services contain the core business logic and handle interactions with database, external APIs, and other services.

**Service Layer Responsibilities:**
- Receive processed data from controller
- Perform business logic operations
- Interact with database (Supabase)
- Call external services if needed (AWS Lambda, Batch)
- Process and transform data
- Return result to controller

**How Services Work:**
1. Controller calls service method with validated data
2. Service executes the core business logic step-by-step
3. Service may need to fetch data from database using Supabase REST API
4. Service may call external services like AWS Lambda or AWS Batch
5. Service processes responses and transforms data into required format
6. Service performs any calculations, validations, or conversions
7. Service returns result to controller
8. Controller wraps result and sends to frontend

#### Layer 4: External Services Integration
For advanced features, services call external APIs and microservices.

**AWS Lambda Integration (Phase 2 - Anomaly Detection):**
The service layer orchestrates the following workflow:
1. Service receives an inspection image upload from the controller
2. Service uploads the image file to Supabase Storage bucket and gets a public URL
3. Service reads the uploaded image file and converts it to Base64 encoded string
4. Service creates a JSON payload containing:
   - Base64-encoded image data
   - Detection threshold (default 0.1, can be customized)
   - IOU threshold (default 0.2 for filtering duplicate detections)
5. Service makes an HTTP POST request to AWS Lambda endpoint with this payload
6. AWS Lambda model processes the image and returns detected anomalies
7. Lambda response includes:
   - List of detections with confidence scores
   - Bounding box coordinates in [x1, y1, x2, y2] format (top-left and bottom-right)
   - Class names/types of detected anomalies
8. Service receives Lambda response and iterates through each detection
9. Service converts coordinates from [x1, y1, x2, y2] to [x_center, y_center, width, height] format
10. Service assigns unique ID to each detection and marks as "AI-generated"
11. Service creates anomaly log entries for audit trail
12. Service returns all detections and logs to controller
13. Controller sends complete inspection data (with detections) to frontend

**Supabase Integration:**
The service layer makes REST API calls to Supabase:
1. Service constructs Supabase REST URL with table name and filters
2. Service adds authentication headers (API key and Bearer token)
3. Service makes HTTP GET/POST/PATCH/DELETE request via RestTemplate
4. Supabase processes the request and returns JSON response
5. Service parses the JSON response and handles any errors
6. Service returns processed data to controller

### Complete Request-Response Example: Creating an Inspection (Phase 2)

**Step 1: Frontend Initiates Request**
- User fills out an inspection form in the React UI with:
  - Transformer number
  - Inspection date
  - Maintenance date
  - Status
  - Inspector name
  - Reference image (thermal image file)
- User clicks "Create Inspection" button
- Frontend uses Axios to construct a multipart/form-data POST request
- Request is sent to backend endpoint: `/api/inspections`
- Request headers include authentication tokens and content type

**Step 2: Backend Controller Receives Request**
- Spring Boot routes the request to InspectionController
- Controller method receives all form parameters:
  - transformerNumber from form data
  - inspectionNumber (optional) from form data
  - Various date and status fields
  - refImage as MultipartFile object
- Controller validates that transformerNumber is not null or empty
- If validation fails, controller returns 400 Bad Request error response
- If validation passes, controller calls inspectionService.createInspection() with all parameters

**Step 3: Service Layer Processes Request**
- Service receives the inspection data from controller
- Service first checks if inspectionNumber was provided by frontend
- If not provided, service generates a unique inspection number (e.g., "I-123456")
- Service checks if an image file was uploaded
- If image exists:
  - Service reads the image file bytes
  - Service uploads image to Supabase Storage bucket
  - Service receives a public URL pointing to stored image
  - Service converts image to Base64 encoding
  - Service creates JSON payload with Base64 image and threshold parameters
  - Service makes HTTP POST request to AWS Lambda endpoint
  - Service waits for Lambda response
  - Lambda returns detected anomalies with coordinates and confidence scores
  - Service converts coordinate format from [x1, y1, x2, y2] to [x_center, y_center, width, height]
  - Service assigns unique IDs to each detection
  - Service marks all detections as "AI-generated"
  - Service creates log entries for each detection in anomalies log
- Service prepares final inspection object with:
  - Generated inspection number
  - Transformer number
  - Inspection date and maintenance date
  - Image URL
  - Detected anomalies list
  - Anomalies log for audit trail
  - Status and inspector information
- Service makes HTTP POST request to Supabase with inspection data
- Supabase creates new record in inspections table and returns the created record
- Service returns the complete inspection record to controller

**Step 4: Frontend Receives Response**
- Frontend receives the response with created inspection data
- Frontend parses the JSON response
- Frontend displays success message to user
- Frontend extracts the new inspection ID and anomalies list
- Frontend renders the inspection details page
- Frontend displays the thermal image with detected anomalies visualized as bounding boxes
- Frontend updates the inspections list to include the newly created inspection
- User can now view anomalies and proceed to annotate them (Phase 3)

### Complete Request-Response Example: Updating Anomaly Annotation (Phase 3)

**Step 1: Frontend Sends Updated Anomaly**
- User views the inspection with detected anomalies displayed on the thermal image
- User hovers over or selects an anomaly to edit it
- User modifies the anomaly:
  - Adjusts the bounding box position and size
  - Updates comments (e.g., "User corrected hotspot location")
  - Changes severity score
- Frontend constructs the updated anomaly object with:
  - Anomaly ID (unchanged)
  - New bounding box coordinates in [x_center, y_center, width, height] format
  - User comments
  - Updated severity score
  - "madeBy" marked as "User" (not AI)
- Frontend uses Axios to send PUT request to `/api/inspections/{inspectionId}/anomalies/{anomalyId}`
- Request body contains the updated anomaly object as JSON

**Step 2: Controller Receives Update Request**
- Spring Boot routes the request to InspectionController
- Controller extracts:
  - Inspection ID (iid) from URL path
  - Anomaly ID from URL path
  - Updated anomaly data from request body (JSON)
- Controller validates that inspection ID and anomaly ID are valid
- Controller calls inspectionService.updateAnomaly() with all three parameters
- Controller catches any exceptions (like "anomaly not found") and returns appropriate error response

**Step 3: Service Updates Anomaly in Database**
- Service receives inspection ID and updated anomaly data from controller
- Service retrieves the current inspection record from Supabase database
- Service extracts the anomalies list and anomalies log from the inspection record
- Service searches through the anomalies list to find the anomaly with matching ID
- Service updates the found anomaly with new values:
  - New bounding box coordinates
  - New comments
  - Updated severity score
  - madeBy field set to "User"
- Service creates a new log entry documenting this change:
  - Anomaly ID being modified
  - New bounding box coordinates
  - Timestamp of change
  - "madeBy" marked as "User"
  - Action type marked as "edit"
  - User comments included
- Service adds this log entry to the anomaliesLog list
- Service prepares update payload with:
  - Updated anomalies list
  - Updated anomaliesLog with new entry
- Service makes HTTP PATCH request to Supabase to update the inspection record
- Supabase updates the record and returns the updated inspection
- Service returns the complete updated inspection record to controller

**Step 4: Frontend Receives Updated Anomaly**
- Frontend receives the response with updated inspection data
- Frontend parses the JSON response
- Frontend updates the local state with the new anomaly data
- Frontend refreshes the visualization of anomalies on the thermal image
- Anomaly display shows the new position/size if coordinates were changed
- Frontend displays success message to user
- User can now see the updated anomaly on the image
- Updated anomaly is now marked as "User" contribution in the anomaly log
- This information feeds back for model retraining (Phase 4)

### Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         FRONTEND (React/Next.js)                     │
│  - User interacts with UI (clicks, uploads, edits)                  │
│  - Constructs HTTP requests with Axios                              │
│  - Displays received data and handles errors                         │
└────────────────────────────┬────────────────────────────────────────┘
                             │ HTTP Request (JSON/FormData)
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│           BACKEND CONTROLLER (Spring Boot @RestController)           │
│  - Receives HTTP request                                             │
│  - Validates path variables & query parameters                       │
│  - Extracts request body                                             │
│  - Delegates to Service layer                                        │
│  - Returns HTTP Response                                             │
└────────────────────────────┬────────────────────────────────────────┘
                             │ Method call with processed data
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│            SERVICE LAYER (Business Logic @Service)                   │
│  - Implements core business logic                                    │
│  - Validates data integrity                                          │
│  - Calls database APIs (Supabase)                                    │
│  - Invokes external services (Lambda, Batch)                         │
│  - Transforms and processes data                                     │
│  - Returns results to Controller                                     │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                ┌────────────┴────────────┬──────────────┐
                ▼                         ▼              ▼
        ┌───────────────┐        ┌─────────────┐  ┌─────────────┐
        │   SUPABASE    │        │ AWS LAMBDA  │  │ AWS BATCH   │
        │  (Database &  │        │  (Anomaly   │  │ (Training)  │
        │   Storage)    │        │ Detection)  │  │             │
        └───────────────┘        └─────────────┘  └─────────────┘
```

### Error Handling & Response Codes

Controllers return appropriate HTTP status codes:
- **200 OK:** Request successful
- **201 Created:** Resource created successfully
- **400 Bad Request:** Invalid input data
- **404 Not Found:** Resource not found
- **500 Internal Server Error:** Server-side error
- **503 Service Unavailable:** External service (Lambda, Supabase) unavailable

---

## Technology Stack

- **Frontend:**
  - React 18+ with TypeScript
  - Next.js for server-side rendering and routing
  - Axios for API communication
  - Tailwind CSS for styling
  - shadcn/ui for component library

- **Backend:**
  - Java 17+
  - Spring Boot 3.x
  - Spring Data JPA for database operations
  - Spring Security for authentication/authorization
  - Maven for dependency management

- **Database & Storage:**
  - Supabase (Backend-as-a-Service)
  - PostgreSQL for relational data
  - S3-compatible storage for thermal images

- **Anomaly Detection:**
  - Python with PyTorch/TensorFlow
  - AWS Lambda or containerized service
  - ONNX for model inference

- **Additional Tools:**
  - Git & GitHub for version control
  - Docker for containerization
  - npm/pnpm for frontend package management

---

## Database Schema

The database schema evolves with each phase. Below is the current and planned structure.

### Phase 1: Core Tables (Implemented)

#### `transformers`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique transformer identifier |
| `location` | TEXT | Not Null | Physical address or coordinates |
| `capacity_kva` | INTEGER | Not Null | Power capacity in kVA |
| `created_at` | TIMESTAMPTZ | Default `now()` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Default `now()` | Last update timestamp |

#### `thermal_images`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique image identifier |
| `transformer_id` | UUID | Foreign Key | Links to transformer |
| `image_url` | TEXT | Not Null | Public URL in S3 storage |
| `image_type` | VARCHAR(20) | Not Null | 'BASELINE' or 'MAINTENANCE' |
| `env_condition` | VARCHAR(20) | Nullable | 'SUNNY', 'CLOUDY', 'RAINY' (BASELINE only) |
| `uploader_id` | TEXT | Not Null | Admin user ID/name |
| `uploaded_at` | TIMESTAMPTZ | Default `now()` | Upload timestamp |

### Phase 2-3: Anomaly Detection Tables (Implemented)

#### `anomaly_detections`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique detection identifier |
| `maintenance_image_id` | UUID | Foreign Key | References thermal_images |
| `baseline_image_id` | UUID | Foreign Key | References thermal_images |
| `detection_type` | VARCHAR(50) | Not Null | Type of anomaly (hotspot, asymmetry, etc.) |
| `coordinates` | JSON | Not Null | Bounding box or region coordinates |
| `severity_score` | FLOAT | Not Null | Anomaly severity (0-1 scale) |
| `confidence_score` | FLOAT | Not Null | Model confidence (0-1 scale) |
| `detected_at` | TIMESTAMPTZ | Default `now()` | Detection timestamp |

#### `annotations` (Phase 3 - Implemented)

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique annotation identifier |
| `detection_id` | UUID | Foreign Key | References anomaly_detections |
| `image_id` | UUID | Foreign Key | References thermal_images |
| `annotation_type` | VARCHAR(50) | Not Null | 'ADDED', 'EDITED', 'DELETED' |
| `coordinates` | JSON | Not Null | Annotated region coordinates |
| `comments` | TEXT | Nullable | User comments |
| `annotator_id` | TEXT | Not Null | User ID of annotator |
| `annotated_at` | TIMESTAMPTZ | Default `now()` | Annotation timestamp |

### Phase 4: Maintenance Records Tables (Implemented)

#### `maintenance_records`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique record identifier |
| `transformer_id` | UUID | Foreign Key | Links to transformer |
| `maintenance_image_id` | UUID | Foreign Key | References thermal_images |
| `inspection_date` | DATE | Not Null | Date of inspection |
| `inspector_name` | TEXT | Not Null | Name of inspector |
| `status` | VARCHAR(50) | Not Null | 'OK', 'NEEDS_MAINTENANCE', 'URGENT' |
| `electrical_readings` | JSON | Nullable | Voltage, current, etc. |
| `recommended_actions` | TEXT | Nullable | Engineer recommendations |
| `additional_remarks` | TEXT | Nullable | Extra notes/comments |
| `created_at` | TIMESTAMPTZ | Default `now()` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Default `now()` | Last update timestamp |

#### `record_anomalies` (Phase 4)

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | Primary Key | Unique association ID |
| `maintenance_record_id` | UUID | Foreign Key | References maintenance_records |
| `annotation_id` | UUID | Foreign Key | References annotations |
| `marked_in_record` | BOOLEAN | Not Null | Whether marked in final record |

---

## API Endpoints

The Spring Boot backend exposes RESTful endpoints organized by phase. Below are the actual implemented endpoints:

### Phase 1: Transformer & Image Management (Implemented)

| Method | Endpoint | Description | Request Body/Params |
|--------|----------|-------------|-------------|
| GET | `/api/transformers` | Fetch all transformers | - |
| GET | `/api/transformers/by-number/{transformerNumber}` | Get transformer by transformer number | - |
| POST | `/api/transformers` | Create new transformer record | `poleNumber`, `region`, `type`, `locationDetails` (optional), `capacity` (optional), `baselineImage` (optional) |
| GET | `/api/transformers/{id}` | Get transformer details with all inspections | - |
| POST | `/api/transformers/{id}/baselineImage` | Upload/update baseline image for transformer | `baselineImage` (file) |

### Phase 2 & 3: Inspections - Anomaly Detection & Annotations (Implemented)

| Method | Endpoint | Description | Request Body/Params |
|--------|----------|-------------|-------------|
| GET | `/api/inspections` | Fetch all inspections | - |
| GET | `/api/inspections/{iid}` | Get inspection with baseline image | - |
| GET | `/api/inspections/by-number/{inspectionNumber}` | Get inspection by inspection number | - |
| GET | `/api/inspections/by-transformer/{transformerNumber}` | Get all inspections for a transformer | - |
| POST | `/api/inspections` | Create new inspection with optional ref image | `transformerNumber` (required), `inspectionNumber` (optional), `inspectionDate`, `maintainanceDate`, `status`, `inspector` (optional), `refImage` (optional) |
| POST | `/api/inspections/{iid}/refImage` | Upload/update reference image for inspection | `refImage` (file), `threshold` (optional) |
| GET | `/api/inspections/{iid}/anomalies` | Get all anomalies detected in an inspection | - |
| POST | `/api/inspections/{iid}/anomalies` | Add a new anomaly annotation to inspection | JSON: `{coordinates, comments, severity, etc.}` |
| PUT | `/api/inspections/{iid}/anomalies/{anomalyId}` | Update existing anomaly annotation | JSON: Updated anomaly fields |
| DELETE | `/api/inspections/{iid}/anomalies/{anomalyId}` | Delete anomaly annotation | - |

### Phase 4: Maintenance Records (Implemented)

| Method | Endpoint | Description | Request Body |
|--------|----------|-------------|-------------|
| GET | `/api/maintenance` | Fetch all maintenance records | - |
| GET | `/api/maintenance/{mid}` | Get specific maintenance record by ID | - |
| GET | `/api/maintenance/inspection/{inspectionNumber}` | Get maintenance record by inspection number | - |
| POST | `/api/maintenance` | Create new maintenance record | `inspectionNumber`, `inspectorName`, `status`, `electricalReadings` (JSON), `recommendedActions`, `additionalRemarks` |
| PATCH | `/api/maintenance/{mid}` | Update maintenance record (partial) | `inspectorName`, `status`, `electricalReadings`, `recommendedActions`, `additionalRemarks` |
| PUT | `/api/maintenance/{mid}` | Replace maintenance record (full) | Same as PATCH |
| DELETE | `/api/maintenance/{mid}` | Delete maintenance record | - |

### Model Retraining

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/retrain` | Trigger model retraining based on user annotations/feedback | - |

---

## Project Structure

```
project-repo/
├── frontend/                          # React/Next.js application
│   ├── app/                          # Next.js app directory
│   │   ├── api/                      # API route handlers
│   │   ├── inspections/              # Inspections pages
│   │   ├── transformers/             # Transformer management pages
│   │   ├── maintenances/             # Maintenance records pages
│   │   └── ...other pages
│   ├── components/                   # Reusable React components
│   │   ├── ui/                       # UI component library
│   │   ├── inspections-page.tsx
│   │   ├── transformers-page.tsx
│   │   └── ...other components
│   ├── hooks/                        # Custom React hooks
│   ├── lib/                          # Utility functions & clients
│   │   ├── supabaseClient.ts
│   │   └── types.ts
│   └── package.json
│
├── backend/                           # Spring Boot application
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/.../        # Java source code
│   │   │   └── resources/            # Properties files
│   │   └── test/
│   ├── pom.xml                       # Maven configuration
│   └── mvnw                          # Maven wrapper
│
├── AnomalyDetection/                 # Anomaly detection service
│   ├── lambda/
│   │   ├── inference/                # Inference Lambda function
│   │   └── trigger_training/         # Training trigger Lambda
│   ├── batch/
│   │   └── batch-trainer/            # Batch training job
│   └── requirements.txt
│
└── README.md                         # This file
```

---

## Getting Started

Follow these instructions to get the project running on your local machine.

### Prerequisites

  * **Git:** For cloning the repository.
  * **Node.js & npm:** (v18 or higher) For running the React frontend.
  * **Java Development Kit (JDK):** (v17 or higher) For running the Spring Boot backend.
  * **Apache Maven:** (v3.8 or higher) For managing backend dependencies and building the project.
  * **Supabase Account:** A free account is sufficient to start.

### 1\. Supabase Setup

1.  Go to [Supabase](https://supabase.com/) and create a new project.
2.  **Database:** Once the project is ready, the database is automatically provisioned. No extra setup is needed for the tables; Spring Boot with JPA will handle table creation.
3.  **Storage:** In the Supabase dashboard, navigate to the **Storage** section and create a new **public bucket**. Let's name it `thermal-images`.
4.  **API Keys:** Navigate to **Project Settings \> API**. You will need the following information for your backend configuration:
      * Project URL
      * `service_role` secret key (this is a privileged key for backend use only)

### 2\. Backend Setup (Spring Boot)

```bash
# Clone the repository
git clone https://github.com/your-username/thermoscan.git
cd thermoscan/backend

# Create an application.properties file
# In `src/main/resources/`, create a file named `application.properties`
# and add the following configuration:
```

**`src/main/resources/application.properties`:**

```properties
# PostgreSQL Database Configuration (from Supabase)
spring.datasource.url=jdbc:postgresql://[YOUR_SUPABASE_HOST]:5432/[YOUR_DB_NAME]
spring.datasource.username=postgres
spring.datasource.password=[YOUR_SUPABASE_DB_PASSWORD]
spring.jpa.hibernate.ddl-auto=update

# Supabase Storage Configuration
supabase.url=[YOUR_SUPABASE_PROJECT_URL]
supabase.key=[YOUR_SUPABASE_SERVICE_ROLE_KEY]
supabase.bucket.name=thermal-images
```

```bash
# Install dependencies and run the application
mvn install
mvn spring-boot:run
```

The backend server will start, typically on `http://localhost:8080`.

### 3\. Frontend Setup (React)

```bash
# Navigate to the frontend directory
cd ../frontend

# Create a .env.local file in the root of the frontend directory
# Add the base URL for your backend API
```

**`frontend/.env.local`:**

```
REACT_APP_API_BASE_URL=http://localhost:8080
```

```bash
# Install dependencies and start the development server
npm install
npm start
```

The React application will open in your browser, usually at `http://localhost:3000`.

---

## Usage

Once both the frontend and backend are running:

1. Open your web browser and navigate to `http://localhost:3000`
2. Navigate to the **Transformers** section to manage transformer records
3. **Create a Transformer:**
   - Click "Add Transformer"
   - Enter transformer ID, location, and capacity
   - Submit to save

4. **Upload Baseline Image (Phase 1):**
   - Select a transformer from the list
   - Click "Upload Baseline Image"
   - Select image file and environmental condition (Sunny/Cloudy/Rainy)
   - Submit to upload

5. **Upload Maintenance Image (Phase 1):**
   - Select a transformer
   - Click "Upload Maintenance Image"
   - Select image file
   - Submit to upload

6. **View Images:**
   - Click on a transformer to view all associated images
   - See image details including upload date, type, and environmental condition

7. **Future Phases (Phase 2-4):**
   - Phase 2: Trigger anomaly detection on maintenance images
   - Phase 3: Annotate detected anomalies
   - Phase 4: Generate and manage maintenance records

---

## Contributing

Contributions are welcome! If you have suggestions or want to improve the code:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is part of the EN3350 Software Design Competition at the University of Moratuwa.

## Contact & Support

For issues, questions, or suggestions about this project, please open an issue in the GitHub repository.
