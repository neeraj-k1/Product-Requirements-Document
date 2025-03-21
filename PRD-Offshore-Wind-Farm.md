# Offshore Wind Farm - Product Requirements Document

## 1. Introduction

### 1.1 Purpose
To create a comprehensive offshore wind farm management system that enables engineers, operators, and maintenance teams to efficiently design, deploy, monitor, and maintain offshore wind energy installations while optimizing energy production and ensuring compliance with environmental and safety regulations.

### 1.2 Scope
This system will provide functionality for managing the complete lifecycle of offshore wind farms, including farm design and planning, turbine configuration and deployment, location analysis and monitoring, and maintenance scheduling and tracking. The system will support decision-making processes for optimal energy production while addressing the unique challenges of offshore operations.

## 2. Global API Specifications

### 2.1 Core Requirements:
- HTTP only (no HTTPS)
- No authentication required
- No pagination
- No concurrency handling
- No rate-limiting
- No performance requirements
- Resource IDs should be UUIDs
- When a resource ID is invalid or not found, always return 404 (no validation of UUID format)
- When creating an object using a POST request, the `created_at` and `updated_at` fields will have the same value. Therefore, it is sufficient to include only the `created_at` field in the response payload.
- Resources: wind farms, turbines, maintenance records
- Complex aspects: Manage wind farm design and planning, turbine configuration and deployment, and maintenance scheduling and tracking.

### 2.2 Base URL Configuration
- **Exact Base URL**: `/api/v1`
- **Versioning Strategy**: Explicit version in URL to support future API evolution
- **Global URL Pattern**: 
  - Parent Resources: `/{resource-type}`
  - Child/Nested Resources: `/{parent-resource}/{parent-id}/{child-resource}`
  - Special Operations: `/{resource-type}/{operation}`

### 2.3 Timestamp Management
- **Timestamp Type**: Server-managed
- **Timestamp Fields**:
  - `created_at`: Immutable creation timestamp
  - `updated_at`: Dynamically updated on any resource modification
  - `deleted_at`: Dynamically updated on any resource soft deletion
- **Format**: ISO 8601 Extended Format (e.g., "2024-02-03T15:30:45.123Z")
- **Time Zone**: Always stored in UTC
- **Precision**: Millisecond-level accuracy

### 2.4 Error Handling

#### 2.4.1. **`400 Bad Request` (Validation Errors)**
```json
{
    "error_id": "VALIDATION_ERROR",
    "errors": [
        {
            "id": "INVALID_FIELD_NAME",
            "field": "field_name",
            "message": "Descriptive error message explaining the validation issue."
        }
    ]
}
```
- **`error_id`**: Identifies the error category; use `"VALIDATION_ERROR"` for validation issues.
- **`errors`**: An array of objects, each detailing a specific validation error.
  - **`id`**: A unique error code (e.g., `"INVALID_FIELD_NAME"`) that categorizes the error.
  - **`field`**: The specific field that triggered the validation error.
  - **`message`**: A clear, descriptive explanation of the validation issue.

#### 2.4.2. **`404 Not Found`**
```json
{
    "error_id": "RESOURCE_NOT_FOUND",
    "message": "The requested resource was not found."
}
```
- **`error_id`**: Identifies the error type; use an identifier like `"ABC_NOT_FOUND"` for missing resources.
- **`message`**: Clearly states which resource was not found (e.g., `"Abc resource not found"`).

#### 2.4.3. **`409 Conflict`**
```json
{
    "error_id": "RESOURCE_CONFLICT",
    "errors": [
        {
            "id": "RESOURCE_STATE_CONFLICT",
            "field": "status",
            "message": "The resource cannot be modified due to its current state."
        }
    ]
}
```
- **`error_id`**: Indicates the conflict error type; use `"RESOURCE_CONFLICT"`.
- **`errors`**: An array of objects detailing each conflict.
  - **`id`**: A unique error code (e.g., `"EXISTING_RESOURCE"`) specifying the type of conflict.
  - **`field`**: Specifies the field or resource that is in conflict.
  - **`message`**: Explains the nature of the conflict.

#### 2.4.4. **`422 Unprocessable Entity`**
```json
{
    "error_id": "UNPROCESSABLE_ENTITY",
    "errors": [
        {
            "id": "INVALID_RELATIONSHIP",
            "field": "related_field",
            "message": "The proposed relationship violates system constraints."
        }
    ]
}
```
- **`error_id`**: Identifies the error type; use `"UNPROCESSABLE_ENTITY"` for processing issues.
- **`errors`**: An array of objects, each detailing a specific processing error.
  - **`id`**: A unique error code (e.g., `"INVALID_RELATIONSHIP"`) that categorizes the error.
  - **`field`**: Specifies the field that caused the error.
  - **`message`**: Provides a clear explanation of why the entity cannot be processed.

#### 2.4.5. **`500 Internal Server Error`**  
```json
{
    "error_id": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred while processing the request. Please try again later."
}
```
- **`error_id`**: Identifies the error category; use `"INTERNAL_SERVER_ERROR"` to indicate a server-side error.
- **`message`**: Provides a clear explanation that an unexpected error occurred and advises the client to try again later.

#### 2.4.6. Error Handling Principles
- Always return appropriate HTTP status codes
- Include unique error identifiers for log tracking
- Maintain consistency across all endpoints

### 2.5 Field Validation Principles
#### 2.5.1 String Fields:
  - Length constraints explicitly defined
  - Trimmed of leading/trailing whitespaces
  - Cannot be empty unless explicitly allowed

#### 2.5.2 Numeric Fields:
  - Precise range constraints
  - Non-negative unless explicitly specified

#### 2.5.3 Enum Fields:
  - Strict matching against predefined values
  - Case-sensitive matching
  - No default values unless specified

## 3. Comprehensive Resource Models

### 3.1 Wind Farm Model

#### 3.1.1 JSON Schema
```json
{
  "farm_id": "uuid",
  "name": "string",
  "status": "PLANNED|OPERATIONAL|DECOMMISSIONED",
  "capacity": "decimal",
  "turbine_ids": ["uuid"],
  "location_latitude": "decimal",
  "location_longitude": "decimal",
  "water_depth": "decimal",
  "distance_to_shore": "decimal",
  "commissioned_date": "date",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp"
  }
}
```

#### 3.1.2 Field Specifications

| Field Name | Type | Required | Mutable | Description | Constraints | Default |
|-------|------|----------|---------|-------------|------------|---------|
| `farm_id` | uuid | Yes | Never | Primary identifier for the wind farm that remains consistent throughout its lifecycle, used in all API operations and cross-references from turbines and maintenance records | Immutable unique identifier | None |
| `name` | string | Yes | Mutable | Official name of the wind farm used in documentation, regulatory filings, and grid connection agreements that will appear on all operational reports and compliance documents | Length: 1-100 | None |
| `status` | enum | Yes | Mutable | Current state of the wind farm that determines its development stage, operational capabilities, and regulatory requirements, affecting energy production forecasts and maintenance scheduling | [`PLANNED`, `OPERATIONAL`, `DECOMMISSIONED`] | `PLANNED` |
| `capacity` | decimal | Yes | Mutable | Total energy generation capacity of the wind farm in megawatts (MW), critical for production forecasting, grid integration planning, and revenue projections | Must be greater than 0 | None |
| `turbine_ids` | array of uuid | No | Mutable | List of turbine IDs associated with this wind farm, establishing direct relationships between the farm and its turbines for comprehensive management and monitoring. Relationship Type: One-to-Many (Wind Farm to Turbines) | Each ID must reference an existing turbine | Empty array |
| `location_latitude` | decimal | Yes | Mutable | Latitude coordinate of the farm's center point, essential for navigation, territorial jurisdiction determination, and weather forecasting | Range: -90 to 90 | None |
| `location_longitude` | decimal | Yes | Mutable | Longitude coordinate of the farm's center point, essential for navigation, territorial jurisdiction determination, and weather forecasting | Range: -180 to 180 | None |
| `water_depth` | decimal | Yes | Mutable | Average depth of water at the site in meters, determining foundation type, installation methods, and maintenance vessel requirements | Min: 0 | None |
| `distance_to_shore` | decimal | Yes | Mutable | Distance from site center to nearest shoreline in kilometers, affecting transmission costs, maintenance accessibility, and emergency response time | Min: 0 | None |
| `commissioned_date` | date | No | Immutable | Date when the wind farm became or is expected to become operational, important for regulatory compliance, warranty periods, and operational planning | Must be a valid date with format YYYY-MM-DD | None |
| `metadata.created_at` | timestamp | Yes | Never | Precise moment when the wind farm record was first created, used for audit trails and chronological tracking of farm development | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Current time |
| `metadata.updated_at` | timestamp | Yes | Mutable | Timestamp of the most recent modification to any wind farm field, used for tracking changes, synchronization, and version control | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Current time |
| `metadata.deleted_at` | timestamp | No | Mutable | When populated, indicates the wind farm has been soft-deleted and should be excluded from active operations while maintaining historical record for regulatory compliance | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Null |

### 3.2 Turbine Model

#### 3.2.1 JSON Schema
```json
{
  "turbine_id": "uuid",
  "farm_id": "uuid",
  "name": "string",
  "model": "string",
  "status": "INSTALLED|OPERATIONAL|MAINTENANCE|INACTIVE",
  "capacity": "decimal",
  "hub_height": "decimal",
  "rotor_diameter": "decimal",
  "installation_date": "date",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp"
  }
}
```

#### 3.2.2 Field Specifications
| Field Name | Type | Required | Mutable | Description | Constraints | Default |
|-------|------|----------|---------|-------------|------------|---------|
| `turbine_id` | uuid | Yes | Never | Primary identifier for the turbine that remains consistent throughout its lifecycle, used in all API operations, maintenance tracking, and performance analysis | Immutable unique identifier | None |
| `farm_id` | uuid | Yes | Never | Reference to the parent wind farm this turbine belongs to, establishing the hierarchical relationship between farm and turbine for operational management and reporting. Relationship Type: Many-to-One (Turbine to Wind Farm) | Must reference existing wind farm | None |
| `name` | string | Yes | Mutable | Identifying name or number of the turbine within the wind farm, used in technical documentation, maintenance records, and operational communications | Length: 1-100 | None |
| `model` | string | Yes | Mutable | Manufacturer and model designation of the turbine, determining its specifications, warranty terms, and maintenance requirements | Length: 1-100 | None |
| `status` | enum | Yes | Mutable | Current operational state of the turbine that determines its availability for energy production, maintenance needs, and inclusion in farm output calculations | [`INSTALLED`, `OPERATIONAL`, `MAINTENANCE`, `INACTIVE`] | `INSTALLED` |
| `capacity` | decimal | Yes | Mutable | Maximum power generation capacity of the turbine in megawatts (MW), essential for production planning, load balancing, and individual turbine performance assessment | Must be greater than 0 | None |
| `hub_height` | decimal | Yes | Mutable | Height of the turbine hub above sea level in meters, affecting wind capture efficiency, structural requirements, and navigation clearance considerations | Must be greater than 0 | None |
| `rotor_diameter` | decimal | Yes | Mutable | Diameter of the turbine rotor in meters, determining the swept area, potential energy capture, and minimum spacing requirements between turbines | Must be greater than 0 | None |
| `installation_date` | date | Yes | Immutable | Date when the turbine was installed at the offshore location, marking the start of its operational lifecycle, warranty period, and maintenance schedule | Must be a valid date with format YYYY-MM-DD | None |
| `metadata.created_at` | timestamp | Yes | Never | Precise moment when the turbine record was first created, used for audit trails, lifecycle tracking, and chronological analysis of farm development | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Current time |
| `metadata.updated_at` | timestamp | Yes | Mutable | Timestamp of the most recent modification to any turbine field, used for tracking changes, synchronization with maintenance systems, and configuration management | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Current time |
| `metadata.deleted_at` | timestamp | No | Mutable | When populated, indicates the turbine has been soft-deleted but is maintained for historical records, warranty claims, and lifecycle analysis | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Null |

### 3.3 Maintenance Model

#### 3.3.1 JSON Schema
```json
{
  "maintenance_id": "uuid",
  "turbine_id": "uuid",
  "status": "PLANNED|IN_PROGRESS|COMPLETED",
  "description": "string",
  "start_date": "date",
  "end_date": "date",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp"
  }
}
```

#### 3.3.2 Field Specifications
| Field Name | Type | Required | Mutable | Description | Constraints | Default |
|-------|------|----------|---------|-------------|------------|---------|
| `maintenance_id` | uuid | Yes | Never | Primary identifier for the maintenance record that remains consistent throughout its lifecycle, used for tracking, reporting, and historical analysis of turbine service history | Immutable unique identifier | None |
| `turbine_id` | uuid | Yes | Never | Reference to the specific turbine receiving maintenance, establishing relationship between maintenance activities and assets for comprehensive service history tracking. Relationship Type: Many-to-One (Maintenance to Turbine) | Must reference existing turbine | None |
| `status` | enum | Yes | Mutable | Current state of the maintenance activity in its workflow, tracking progress from planning through execution to completion for operational oversight | [`PLANNED`, `IN_PROGRESS`, `COMPLETED`] | `PLANNED` |
| `description` | string | No | Mutable | Detailed explanation of the maintenance activity, issues addressed, and work performed, providing context for operations teams and future maintenance planning | Length: 1-500 | None |
| `start_date` | date | Yes | Mutable | Planned or actual date when maintenance begins, used for scheduling maintenance vessels, technician teams, and calculating production impact during servicing | Must be a valid date with format YYYY-MM-DD | None |
| `end_date` | date | No | Mutable | Planned or actual date when maintenance concludes, used for tracking duration, coordinating return to service, and analyzing maintenance efficiency | Must be a valid date with format YYYY-MM-DD | None |
| `metadata.created_at` | timestamp | Yes | Never | Precise moment when the maintenance record was first created, used for audit trails, maintenance history chronology, and service interval tracking | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Current time |
| `metadata.updated_at` | timestamp | Yes | Mutable | Timestamp of the most recent modification to any maintenance field, tracking maintenance record evolution and service history updates | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Current time |
| `metadata.deleted_at` | timestamp | No | Mutable | When populated, indicates the maintenance record has been soft-deleted but maintained for historical reference, warranty documentation, and regulatory compliance | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Null |


## 4. Complex Business Rules

### 4.1 Conditional Field Requirements

#### 4.1.1 Wind Farm Model
- `commissioned_date` field becomes required when `status` transitions to "OPERATIONAL"
- `commissioned_date` cannot be modified once the Wind Farm status is "OPERATIONAL"
- A Wind Farm cannot be directly created with `status` "DECOMMISSIONED" or "OPERATIONAL" (must follow proper state transitions)
- `capacity` becomes required when `turbine_ids` array is not empty
- `distance_to_shore` field becomes required when status changes from "PLANNED" to "OPERATIONAL"

#### 4.1.2 Turbine Model
- `capacity` field cannot be modified when the Turbine status is "OPERATIONAL"

#### 4.1.3 Maintenance Model
- `end_date` field becomes required when `status` is "COMPLETED"
- `start_date` should be before `end_date`
- `description` should be provided when status is "COMPLETED".

### 4.2 State Machine Transitions

#### 4.2.1 Wind Farm Status Transitions
- From "PLANNED" to "OPERATIONAL"
- From "OPERATIONAL" to "DECOMMISSIONED"
- Once "DECOMMISSIONED", the status cannot be changed

#### 4.2.2 Turbine Status Transitions
- From "INSTALLED" to "OPERATIONAL"
- From "OPERATIONAL" to "MAINTENANCE"
- From "MAINTENANCE" to "OPERATIONAL"
- From "OPERATIONAL" to "INACTIVE"
- From "MAINTENANCE" to "INACTIVE"
- Once "INACTIVE", the status cannot be changed

#### 4.2.3 Maintenance Status Transitions
- From "PLANNED" to "IN_PROGRESS"
- From "IN_PROGRESS" to "COMPLETED"
- Once "COMPLETED", the status cannot be changed

### 4.3 Cross-Resource Validation Rules

#### 4.3.1 Wind Farm and Turbine Relationships
- Turbines can only reference Wind Farms that are not "DECOMMISSIONED"
- When a Wind Farm's status changes to "DECOMMISSIONED", all associated Turbines should have status "INACTIVE"
- Total `capacity` of all Turbines (referenced by `turbine_ids`) must not exceed the Wind Farm's `capacity`
- The `turbine_ids` array must contain only valid turbine IDs that exist in the system
- When a Turbine is created with a reference to a Wind Farm, that Turbine's ID must be automatically added to the Wind Farm's `turbine_ids` array

#### 4.3.2 Turbine and Maintenance Relationships
- Maintenance records can only reference Turbines that are not "INACTIVE"
- When a Maintenance record with status "IN_PROGRESS" exists for a Turbine, the Turbine's status must be "MAINTENANCE"
- When a Maintenance record changes to "COMPLETED", the associated Turbine's status should change to "OPERATIONAL" or "INACTIVE"

### 4.4 Multi-Step Operations

#### 4.4.1 Wind Farm Commissioning Process
1. **Initial Planning**
   - Create Wind Farm record with status "PLANNED"
   - Define basic attributes (name, location coordinates, water depth, distance to shore)

2. **Turbine Installation**
   - Add all INSTALLED turbines to the wind farm
   - Ensure all turbines have proper specifications
   - Verify total turbine capacity is less than or equal to the wind farm's capacity

3. **Operational Readiness**
   - Set commissioned_date to the current date
   - Change wind farm status to "OPERATIONAL"

**Rollback Scenarios:**
- If the total turbine capacity exceeds the wind farm's capacity, prevent the wind farm from transitioning to OPERATIONAL status. Rollback by updating the wind farm's capacity specification or removing excess turbines.

### 4.5 Deletion Behavior

#### 4.5.1 Wind Farm Model Deletion Rules
- Wind Farm deletion MUST be prevented if:
  - Wind Farm has status "OPERATIONAL" and has associated operational turbines

- When a Wind Farm is marked for deletion (metadata.deleted_at populated):
  - It should no longer appear in standard wind farm listings
  - It remains in the database for historical reference and cannot be modified
  - If the Wind Farm is deleted when in "PLANNED" status, associated turbines should be marked for deletion

#### 4.5.2 Turbine Model Deletion Rules
- Turbine deletion MUST be prevented if:
  - It has status "OPERATIONAL", "MAINTENANCE", "INSTALLED".

- When a Turbine is marked for deletion (metadata.deleted_at populated):
  - Turbine remains in database for historical reference
  - Turbine ID should be removed from the associated Wind Farm's `turbine_ids` array
  - Turbine is excluded from standard turbine listings

#### 4.5.3 Maintenance Model Deletion Rules
- Maintenance record deletion MUST be prevented if:
  - Maintenance has status "IN_PROGRESS"

- When a Maintenance record is marked for deletion (metadata.deleted_at populated):
  - Maintenance record remains available for historical reference
  - Maintenance record is excluded from standard maintenance listings

### 4.6 Deletion Audit Requirements

#### 4.6.1 Audit Trail:

- All deletion attempts (successful or prevented) must be logged with:
  - Timestamp of deletion attempt (ISO 8601 format)
  - User or system identifier that initiated the deletion
  - Resource ID and type

#### 4.6.2 Retention Period:

- Deletion audit logs must be retained for a minimum of 7 years
- Deletion of critical infrastructure components must be retained for 15 years
- Audit logs cannot be deleted or modified once created

#### 4.6.3 Regulatory Compliance:

- Deletion of certain resources may require regulatory notification
- Critical infrastructure deletions may require additional approval workflow


## 5. Comprehensive API Endpoints

### 5.1 Wind Farm Endpoints

#### 5.1.1 Create Wind Farm
**URL:** POST /api/v1/windfarms

**Request Body Schema:**
```json
{
  "name": "string", // Required - Official name of the wind farm
  "capacity": "decimal", // Required - Total energy generation capacity in megawatts
  "location_latitude": "decimal", // Required - Latitude coordinate of farm's center
  "location_longitude": "decimal", // Required - Longitude coordinate of farm's center
  "water_depth": "decimal", // Required - Average water depth in meters
  "distance_to_shore": "decimal", // Required - Distance to nearest shoreline in kilometers
  "status": "PLANNED" // Optional, Default: PLANNED
}
```

**Success Response (201 Created):**
```json
{
  "farm_id": "uuid",
  "name": "string",
  "status": "PLANNED",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_NAME_LENGTH",
      "field": "name",
      "message": "Name must be between 1 and 100 characters."
    },
    {
      "id": "INVALID_CAPACITY_VALUE",
      "field": "capacity",
      "message": "Capacity must be greater than 0."
    },
    {
      "id": "MISSING_LOCATION_LATITUDE",
      "field": "location_latitude",
      "message": "Location latitude is required."
    },
    {
      "id": "LATITUDE_OUT_OF_RANGE",
      "field": "location_latitude",
      "message": "Latitude must be between -90 and 90."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.1.2 Get Wind Farm
**URL:** GET /api/v1/windfarms/{farmId}

**Success Response (200 OK):**
```json
{
  "farm_id": "uuid",
  "name": "string",
  "status": "PLANNED|OPERATIONAL|DECOMMISSIONED",
  "capacity": "decimal",
  "turbine_ids": ["uuid"],
  "location_latitude": "decimal",
  "location_longitude": "decimal",
  "water_depth": "decimal",
  "distance_to_shore": "decimal",
  "commissioned_date": "date",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" | null
  }
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Wind Farm with ID {farmId} not found."
}
```

- **404 Resource Deleted:**
```json
{
  "error_id": "RESOURCE_DELETED",
  "message": "Wind Farm with ID {farmId} has been deleted. Use include_deleted=true query parameter to retrieve deleted resources."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.1.3 Update Wind Farm
**URL:** PATCH /api/v1/windfarms/{farmId}

**Request Body Schema:**
```json
{
  "name": "string", // Optional - Updated name of the wind farm
  "capacity": "decimal", // Optional - Updated capacity in megawatts
  "turbine_ids": ["uuid"], // Optional - Updated list of turbine IDs
  "location_latitude": "decimal", // Optional - Updated latitude coordinate
  "location_longitude": "decimal", // Optional - Updated longitude coordinate
  "water_depth": "decimal", // Optional - Updated water depth
  "distance_to_shore": "decimal", // Optional - Updated distance to shore
  "commissioned_date": "date", // Optional - Date of commissioning
  "status": "PLANNED|OPERATIONAL|DECOMMISSIONED" // Optional - Updated status
}
```

**Success Response (200 OK):**
```json
{
  "farm_id": "uuid",
  "name": "string",
  "status": "PLANNED|OPERATIONAL|DECOMMISSIONED",
  "capacity": "decimal",
  "turbine_ids": ["uuid"],
  "location_latitude": "decimal",
  "location_longitude": "decimal",
  "water_depth": "decimal",
  "distance_to_shore": "decimal",
  "commissioned_date": "date",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_CAPACITY_VALUE",
      "field": "capacity",
      "message": "Capacity must be greater than 0."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "WINDFARM_NOT_FOUND",
  "message": "Wind Farm with ID {farmId} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "TURBINE_LIST_MISMATCH",
      "field": "turbine_ids",
      "message": "The provided turbine IDs do not match the turbines associated with this wind farm."
    },
    {
      "id": "INVALID_STATUS_TRANSITION",
      "field": "status",
      "message": "Cannot transition from OPERATIONAL to PLANNED or from DECOMMISSIONED to any other status."
    }
  ]
}
```

- **422 Unprocessable Entity:**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "errors": [
    {
      "id": "IMMUTABLE_COMMISSIONED_DATE",
      "field": "commissioned_date",
      "message": "commissioned_date is immutable once the status is operational"
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.1.4 Delete Wind Farm
**URL:** DELETE /api/v1/windfarms/{farmId}

**Success Response (204 No Content)**

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "WINDFARM_NOT_FOUND",
  "message": "Wind Farm with ID {farmId} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "ACTIVE_WINDFARM_DELETION",
      "field": "status",
      "message": "Cannot delete wind farm with OPERATIONAL status and operational turbines."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.1.5 List Wind Farms
**URL:** GET /api/v1/windfarms

**Query Parameters:**
- status.eq: string (optional) - Filter by exact status match (PLANNED, OPERATIONAL, DECOMMISSIONED)
- include_deleted: boolean (optional, default: false) - Whether to include soft-deleted wind farms

**Success Response (200 OK):**
```json
{
  "windfarms": [
    {
      "farm_id": "uuid",
      "name": "string",
      "status": "PLANNED|OPERATIONAL|DECOMMISSIONED",
      "capacity": "decimal",
      "location_latitude": "decimal",
      "location_longitude": "decimal",
      "water_depth": "decimal",
      "distance_to_shore": "decimal",
      "commissioned_date": "date",
      "metadata": {
        "created_at": "timestamp",
        "updated_at": "timestamp",
        "deleted_at": "timestamp" | null
      }
    }
  ],
  "count": "integer"
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_QUERY_PARAMETER",
      "field": "status",
      "message": "Invalid query parameter value."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

### 5.2 Turbine Endpoints

#### 5.2.1 Create Turbine
**URL:** POST /api/v1/turbines

**Request Body Schema:**
```json
{
  "farm_id": "uuid", // Required - Reference to parent wind farm
  "name": "string", // Required - Identifying name/number of the turbine
  "model": "string", // Required - Manufacturer and model designation
  "capacity": "decimal", // Required - Maximum power generation capacity
  "hub_height": "decimal", // Required - Height of turbine hub above sea level
  "rotor_diameter": "decimal", // Required - Diameter of the turbine rotor
  "installation_date": "date" // Required - Date of installation (required since status is INSTALLED)
}
```

**Success Response (201 Created):**
```json
{
  "turbine_id": "uuid",
  "name": "string",
  "status": "INSTALLED",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_FARM_ID",
      "field": "farm_id",
      "message": "Farm ID is required."
    },
    {
      "id": "INVALID_CAPACITY_VALUE",
      "field": "capacity",
      "message": "Capacity must be greater than 0."
    },
    {
      "id": "MISSING_INSTALLATION_DATE",
      "field": "installation_date",
      "message": "Installation date is required when status is INSTALLED."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "WINDFARM_NOT_FOUND",
  "message": "Referenced wind farm with ID {farm_id} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "DECOMMISSIONED_WIND_FARM",
      "field": "farm_id",
      "message": "Cannot create turbine for a wind farm with DECOMMISSIONED status."
    },
    {
      "id": "CAPACITY_EXCEEDED",
      "field": "capacity",
      "message": "Adding this turbine would exceed the wind farm's total capacity."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.2.2 Get Turbine
**URL:** GET /api/v1/turbines/{turbineId}

**Success Response (200 OK):**
```json
{
  "turbine_id": "uuid",
  "farm_id": "uuid",
  "name": "string",
  "model": "string",
  "status": "INSTALLED|OPERATIONAL|MAINTENANCE|INACTIVE",
  "capacity": "decimal",
  "hub_height": "decimal",
  "rotor_diameter": "decimal",
  "installation_date": "date",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" | null
  }
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Turbine with ID {turbineId} not found."
}
```

- **404 Resource Deleted:**
```json
{
  "error_id": "RESOURCE_DELETED",
  "message": "Turbine with ID {turbineId} has been deleted. Use include_deleted=true query parameter to retrieve deleted resources."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.2.3 Update Turbine
**URL:** PATCH /api/v1/turbines/{turbineId}

**Request Body Schema:**
```json
{
  "name": "string", // Optional - Updated turbine name
  "model": "string", // Optional - Updated model designation
  "status": "INSTALLED|OPERATIONAL|MAINTENANCE|INACTIVE", // Optional - Updated status
  "capacity": "decimal", // Optional - Updated capacity
  "hub_height": "decimal", // Optional - Updated hub height
  "rotor_diameter": "decimal", // Optional - Updated rotor diameter
  "installation_date": "date", // Optional - Date of installation
}
```

**Success Response (200 OK):**
```json
{
  "turbine_id": "uuid",
  "farm_id": "uuid",
  "name": "string",
  "model": "string",
  "status": "INSTALLED|OPERATIONAL|MAINTENANCE|INACTIVE",
  "capacity": "decimal",
  "hub_height": "decimal",
  "rotor_diameter": "decimal",
  "installation_date": "date",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_STATUS_TRANSITION",
      "field": "status",
      "message": "Cannot transition from INACTIVE to any other status."
    },
    {
      "id": "MISSING_INSTALLATION_DATE",
      "field": "installation_date",
      "message": "Installation date is required when status is OPERATIONAL."
    },
    {
      "id": "IMMUTABLE_CAPACITY",
      "field": "capacity",
      "message": "Cannot modify capacity when turbine status is OPERATIONAL."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Turbine with ID {turbineId} not found."
}
```

- **404 Resource Deleted:**
```json
{
  "error_id": "RESOURCE_DELETED",
  "message": "Turbine with ID {turbineId} has been deleted. Use include_deleted=true query parameter to retrieve deleted resources."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "OPERATIONAL_STATUS_CONFLICT",
      "field": "status",
      "message": "Cannot set status to OPERATIONAL when parent wind farm is not OPERATIONAL."
    },
    {
      "id": "MAINTENANCE_STATUS_CONFLICT",
      "field": "status",
      "message": "Cannot change status from MAINTENANCE when there are maintenance records with IN_PROGRESS status."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.2.4 Delete Turbine
**URL:** DELETE /api/v1/turbines/{turbineId}

**Success Response (204 No Content)**

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Turbine with ID {turbineId} not found."
}
```

- **404 Resource Deleted:**
```json
{
  "error_id": "RESOURCE_DELETED",
  "message": "Turbine with ID {turbineId} has been deleted. Use include_deleted=true query parameter to retrieve deleted resources."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "ACTIVE_TURBINE_DELETION",
      "field": "status",
      "message": "Cannot delete turbine with status OPERATIONAL, MAINTENANCE, or INSTALLED."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.2.5 List Turbines by Wind Farm
**URL:** GET /api/v1/windfarms/{farmId}/turbines

**Query Parameters:**
- status.eq: string (optional) - Filter by exact status match (INSTALLED, OPERATIONAL, MAINTENANCE, INACTIVE)
- include_deleted: boolean (optional, default: false) - Whether to include soft-deleted turbines

**Success Response (200 OK):**
```json
{
  "turbines": [
    {
      "turbine_id": "uuid",
      "name": "string",
      "model": "string",
      "status": "INSTALLED|OPERATIONAL|MAINTENANCE|INACTIVE",
      "capacity": "decimal",
      "hub_height": "decimal",
      "rotor_diameter": "decimal",
      "installation_date": "date",
      "metadata": {
        "created_at": "timestamp",
        "updated_at": "timestamp",
        "deleted_at": "timestamp" | null
      }
    }
  ],
  "count": "integer"
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "WINDFARM_NOT_FOUND",
  "message": "Wind Farm with ID {farmId} not found."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

### 5.3 Maintenance Endpoints

#### 5.3.1 Create Maintenance
**URL:** POST /api/v1/maintenance

**Request Body Schema:**
```json
{
  "turbine_id": "uuid", // Required - Reference to turbine requiring maintenance
  "status": "PLANNED", // Optional, Default: PLANNED
  "description": "string", // Required - Detailed explanation of maintenance
  "start_date": "date" // Required
}
```

**Success Response (201 Created):**
```json
{
  "maintenance_id": "uuid",
  "turbine_id": "uuid",
  "status": "PLANNED",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_TURBINE_ID",
      "field": "turbine_id",
      "message": "Turbine ID is required."
    },
    {
      "id": "MISSING_DESCRIPTION",
      "field": "description",
      "message": "Description is required."
    },
    {
      "id": "MISSING_START_DATE",
      "field": "start_date",
      "message": "Start date is required."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "TURBINE_NOT_FOUND",
  "message": "Referenced turbine with ID {turbine_id} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "INACTIVE_TURBINE_MAINTENANCE",
      "field": "turbine_id",
      "message": "Cannot create maintenance for a turbine with INACTIVE status."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.3.2 Get Maintenance
**URL:** GET /api/v1/maintenance/{maintenanceId}

**Success Response (200 OK):**
```json
{
  "maintenance_id": "uuid",
  "turbine_id": "uuid",
  "status": "PLANNED|IN_PROGRESS|COMPLETED",
  "description": "string",
  "start_date": "date",
  "end_date": "date",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" | null
  }
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Maintenance record with ID {maintenanceId} not found."
}
```

- **404 Resource Deleted:**
```json
{
  "error_id": "RESOURCE_DELETED",
  "message": "Maintenance record with ID {maintenanceId} has been deleted. Use include_deleted=true query parameter to retrieve deleted resources."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.3.3 Update Maintenance
**URL:** PATCH /api/v1/maintenance/{maintenanceId}

**Request Body Schema:**
```json
{
  "status": "PLANNED|IN_PROGRESS|COMPLETED", // Optional - Updated status
  "description": "string", // Optional - Updated description
  "start_date": "date", // Optional - Updated start date
  "end_date": "date" // Optional - Completion date
}
```

**Success Response (200 OK):**
```json
{
  "maintenance_id": "uuid",
  "turbine_id": "uuid",
  "status": "PLANNED|IN_PROGRESS|COMPLETED",
  "description": "string",
  "start_date": "date",
  "end_date": "date",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_STATUS_TRANSITION",
      "field": "status",
      "message": "Cannot transition from COMPLETED to any other status."
    },
    {
      "id": "MISSING_END_DATE",
      "field": "end_date",
      "message": "End date is required when status is COMPLETED."
    },
    {
      "id": "INVALID_DATE_RANGE",
      "field": "end_date",
      "message": "End date must be on or after start date."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Maintenance record with ID {maintenanceId} not found."
}
```

- **404 Resource Deleted:**
```json
{
  "error_id": "RESOURCE_DELETED",
  "message": "Maintenance record with ID {maintenanceId} has been deleted. Use include_deleted=true query parameter to retrieve deleted resources."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "INACTIVE_TURBINE_MAINTENANCE",
      "field": "turbine_id",
      "message": "Cannot update maintenance when associated turbine is INACTIVE."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.3.4 Delete Maintenance
**URL:** DELETE /api/v1/maintenance/{maintenanceId}

**Success Response (204 No Content)**

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Maintenance record with ID {maintenanceId} not found."
}
```

- **404 Resource Deleted:**
```json
{
  "error_id": "RESOURCE_DELETED",
  "message": "Maintenance record with ID {maintenanceId} has been deleted. Use include_deleted=true query parameter to retrieve deleted resources."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "ACTIVE_MAINTENANCE_DELETION",
      "field": "status",
      "message": "Cannot delete maintenance with status IN_PROGRESS."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

#### 5.3.5 List Maintenance by Turbine
**URL:** GET /api/v1/turbines/{turbineId}/maintenance

**Query Parameters:**
- status.eq: string (optional) - Filter by exact status match (PLANNED, IN_PROGRESS, COMPLETED)
- include_deleted: boolean (optional, default: false) - Whether to include soft-deleted maintenance records

**Success Response (200 OK):**
```json
{
  "maintenance": [
    {
      "maintenance_id": "uuid",
      "status": "PLANNED|IN_PROGRESS|COMPLETED",
      "description": "string",
      "start_date": "date",
      "end_date": "date",
      "metadata": {
        "created_at": "timestamp",
        "updated_at": "timestamp",
        "deleted_at": "timestamp" | null
      }
    }
  ],
  "count": "integer"
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Turbine with ID {turbineId} not found."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request. Please try again later."
}
```

## 6. Example Workflows

### 6.1 Wind Farm Commissioning Process

This example illustrates the complete workflow for creating, configuring, and commissioning an offshore wind farm.

#### Step 1: Create Initial Wind Farm

**Request:**
```
POST /api/v1/windfarms
```
```json
{
  "name": "North Sea Winds",
  "capacity": 450.0,
  "location_latitude": 55.4,
  "location_longitude": 2.3,
  "water_depth": 35.0,
  "distance_to_shore": 28.5,
  "status": "PLANNED"
}
```

**Response (201 Created):**
```json
{
  "farm_id": "f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f",
  "name": "North Sea Winds",
  "status": "PLANNED",
  "metadata": {
    "created_at": "2023-09-15T08:30:22.456Z"
  }
}
```

#### Step 2: Add Turbines to Wind Farm

**Request:**
```
POST /api/v1/turbines
```
```json
{
  "farm_id": "f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f",
  "name": "NSW-T01",
  "model": "SeaWind X-10",
  "capacity": 10.0,
  "hub_height": 120.0,
  "rotor_diameter": 164.0,
  "installation_date": "2023-09-20"
}
```

**Response (201 Created):**
```json
{
  "turbine_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "name": "NSW-T01",
  "status": "INSTALLED",
  "metadata": {
    "created_at": "2023-09-20T14:15:33.789Z"
  }
}
```

**Request:**
```
POST /api/v1/turbines
```
```json
{
  "farm_id": "f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f",
  "name": "NSW-T02",
  "model": "SeaWind X-10",
  "capacity": 10.0,
  "hub_height": 120.0,
  "rotor_diameter": 164.0,
  "installation_date": "2023-09-20"
}
```

**Response (201 Created):**
```json
{
  "turbine_id": "b2c3d4e5-f6a7-8b9c-0d1e-2f3a4b5c6d7",
  "name": "NSW-T02",
  "status": "INSTALLED",
  "metadata": {
    "created_at": "2023-09-20T14:20:45.123Z"
  }
}
```

#### Step 3: Update Turbines to Operational Status

**Request:**
```
PATCH /api/v1/turbines/a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
```
```json
{
  "status": "OPERATIONAL"
}
```

**Response (200 OK):**
```json
{
  "turbine_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "farm_id": "f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f",
  "name": "NSW-T01",
  "model": "SeaWind X-10",
  "status": "OPERATIONAL",
  "capacity": 10.0,
  "hub_height": 120.0,
  "rotor_diameter": 164.0,
  "installation_date": "2023-09-20",
  "metadata": {
    "created_at": "2023-09-20T14:15:33.789Z",
    "updated_at": "2023-09-25T09:35:22.456Z"
  }
}
```

**Request:**
```
PATCH /api/v1/turbines/b2c3d4e5-f6a7-8b9c-0d1e-2f3a4b5c6d7
```
```json
{
  "status": "OPERATIONAL"
}
```

**Response (200 OK):**
```json
{
  "turbine_id": "b2c3d4e5-f6a7-8b9c-0d1e-2f3a4b5c6d7",
  "farm_id": "f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f",
  "name": "NSW-T02",
  "model": "SeaWind X-10",
  "status": "OPERATIONAL",
  "capacity": 10.0,
  "hub_height": 120.0,
  "rotor_diameter": 164.0,
  "installation_date": "2023-09-20",
  "metadata": {
    "created_at": "2023-09-20T14:20:45.123Z",
    "updated_at": "2023-09-25T09:40:18.789Z"
  }
}
```

#### Step 4: Commission the Wind Farm
- All turbines referenced in `turbine_ids` must be in OPERATIONAL status
- `capacity` is greater than the sum of the capacity of all turbines in the wind farm

**Request:**
```
PATCH /api/v1/windfarms/f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f
```
```json
{
  "status": "OPERATIONAL",
  "commissioned_date": "2023-09-30"
}
```

**Response (200 OK):**
```json
{
  "farm_id": "f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f",
  "name": "North Sea Winds",
  "status": "OPERATIONAL",
  "capacity": 450.0,
  "turbine_ids": ["a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d", "b2c3d4e5-f6a7-8b9c-0d1e-2f3a4b5c6d7"],
  "location_latitude": 55.4,
  "location_longitude": 2.3,
  "water_depth": 35.0,
  "distance_to_shore": 28.5,
  "commissioned_date": "2023-09-30",
  "metadata": {
    "created_at": "2023-09-15T08:30:22.456Z",
    "updated_at": "2023-09-30T12:00:45.789Z"
  }
}
```

#### Potential Error Scenarios and Rollbacks

### 6.3 Error Scenarios

#### Example 1: Attempting to commission a wind farm without all turbines operational

**Request:**
```
PATCH /api/v1/windfarms/f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f
```
```json
{
  "status": "OPERATIONAL",
  "commissioned_date": "2023-09-30"
}
```

**Response (409 Conflict):**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "TURBINE_STATUS_CONFLICT",
      "field": "status",
      "message": "Cannot change status to OPERATIONAL when turbines are not all in OPERATIONAL status."
    }
  ]
}
```

#### Example 2: Trying to change a turbine's capacity when it's operational

**Request:**
```
PATCH /api/v1/turbines/a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
```
```json
{
  "capacity": 12.0
}
```

**Response (400 Bad Request):**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "IMMUTABLE_CAPACITY",
      "field": "capacity",
      "message": "Cannot modify capacity when turbine status is OPERATIONAL."
    }
  ]
}
```

#### Example 3: Attempting to delete a wind farm with operational turbines

**Request:**
```
DELETE /api/v1/windfarms/f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f
```

**Response (409 Conflict):**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "ACTIVE_WINDFARM_DELETION",
      "field": "status",
      "message": "Cannot delete wind farm with OPERATIONAL status and operational turbines."
    }
  ]
}
```

#### Example 4: Attempting to complete maintenance without providing end date

**Request:**
```
PATCH /api/v1/maintenance/c5d6e7f8-a9b0-1c2d-3e4f-5a6b7c8d9e0f
```
```json
{
  "status": "COMPLETED"
}
```

**Response (400 Bad Request):**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_END_DATE",
      "field": "end_date",
      "message": "End date is required when status is COMPLETED."
    }
  ]
}
```

#### Example 5: Trying to modify commissioned_date after wind farm is operational

**Request:**
```
PATCH /api/v1/windfarms/f8e7d6c5-b4a3-2c1d-9e8f-7a6b5c4d3e2f
```
```json
{
  "commissioned_date": "2023-10-15"
}
```

**Response (422 Unprocessable Entity):**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "errors": [
    {
      "id": "IMMUTABLE_COMMISSIONED_DATE",
      "field": "commissioned_date",
      "message": "commissioned_date is immutable once the status is operational"
    }
  ]
}
```
