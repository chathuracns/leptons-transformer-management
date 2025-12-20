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
  - Computer vision model (classical or deep learning-based) for image comparison
  - Detect thermal anomalies: temperature spikes, asymmetries, hotspot location changes
  - Thresholding mechanism (fixed or adaptive) to flag anomalies
  - Optimized model inference for responsive performance
  - Modular integration for model evolution over time
  - Metadata recording of all detection outputs

#### FR2.2: Side-by-Side Image Comparison View
- **Description:** Interactive side-by-side display of new and baseline images with comparison controls
- **Implemented Features:**
  - Side-by-side display of maintenance and baseline images
  - Interactive controls: zoom, pan (click and drag), reset
  - Visual highlighting of anomaly regions (bounding boxes, overlays, heatmaps)
  - Responsive image comparison interface

#### FR2.3: Automatic Anomaly Marking
- **Description:** Automatic annotation of images with detected anomalies and visual overlays
- **Implemented Features:**
  - Color-coded overlays or markers for detected anomalies
  - Anomaly metadata display: pixel coordinates, anomaly size, severity score
  - Confidence score or flag for uncertain detections
  - Visual representation of detection results for user understanding

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
  - Intuitive and user-friendly annotation interface

#### FR3.2: Metadata and Annotation Persistence
- **Description:** Storage and retrieval of all user annotations with complete metadata tracking
- **Implemented Features:**
  - Capture and save all annotation changes in the backend
  - Store comprehensive metadata:
    - User ID
    - Timestamp
    - Image ID
    - Transformer ID
    - Action taken
  - Automatic reloading of existing annotations when revisiting an image
  - Structured and queryable annotation storage (relational DB or NoSQL)

#### FR3.3: Feedback Integration for Model Improvement
- **Description:** Feedback log system for capturing user corrections and annotations for model improvement
- **Implemented Features:**
  - Feedback log including:
    - Original AI-generated detections
    - Final user-modified annotations
  - Training/validation data generation for model improvement
  - Use user-modified annotations to improve AI model accuracy
  - Exportable feedback log in JSON or CSV format with:
    - Image ID
    - Model-predicted anomalies
    - Final accepted annotations
    - Annotator metadata

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

1. **Frontend (Client-Side):** A responsive and interactive user interface built with **React**. It handles all user interactions and communicates with the backend via a REST API.

2. **Backend (Server-Side):** A robust RESTful API developed with **Java Spring Boot**. It manages all business logic, data processing, validation, and orchestration between frontend and database.

3. **Database & Storage:** We use **Supabase**, a Backend-as-a-Service platform.
   - **Database:** PostgreSQL relational database for structured data (transformers, images, annotations, maintenance records)
   - **Storage:** S3-compatible storage bucket for thermal image files

4. **Anomaly Detection Service:** Separate microservice (Lambda/containerized) for AI-based image analysis and anomaly detection.

### Data Flow Architecture

#### Image Upload Flow (Phase 1)
1. User uploads image in React UI
2. Frontend sends image + metadata to Spring Boot backend
3. Backend authenticates request
4. Image uploaded to Supabase S3, receives public URL
5. Image metadata + URL saved to PostgreSQL
6. Success response returned to frontend

#### Anomaly Detection Flow (Phase 2-3)
1. User triggers analysis on maintenance image
2. Backend retrieves baseline and maintenance images
3. Anomaly detection service processes image pair
4. Detections stored in database with confidence scores
5. User views detections in frontend UI
6. User provides feedback/annotations (Phase 3)
7. Annotations stored for model improvement

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
