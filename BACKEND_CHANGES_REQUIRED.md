# Backend/Node.js Changes Required for Transport Plan Enhancement

## Overview
This document identifies the backend changes needed to support the enhanced Transport Plan functionality that allows:
- Multi-SCN/Container selection
- Multiple Vehicle Type + Vendor + No of Vehicles combinations
- Multiple Transport Order creation (one per combination)
- Search functionality for ASN/SCN/Container Number

---

## 1. Backend Services Involved

Based on the frontend code analysis, the following backend services are involved:

### 1.1 ASN Service (`asn-a/api/v1/`)
**Base URL:** `/asn-a/api/v1/`
- **Current Endpoints:**
  - `TOplan/TOCreateWrapper` - Creates Transport Order
  - `delivery/inbdeliverylist` - Gets inbound delivery list
  - `delivery/plannedinblist` - Gets planned inbound delivery list
  - `TOplan/AddLegDetails` - Adds leg details to Transport Order

**Repository to Clone:** `asn-a` (ASN Platform Service)

### 1.2 Order Service (`order-a/api/v1/`)
**Base URL:** `/order-a/api/v1/`
- **Current Endpoints:**
  - `order/createdeliveryto` - Creates delivery/transport order
  - `order/createdeliverytoNew` - Creates delivery/transport order (new version)
  - `vendorallocation/get` - Gets vendor allocation list

**Repository to Clone:** `order-a` (Order Service)

### 1.3 Vehicle Service (`vehicleservice/api/v1/`)
**Base URL:** `/vehicleservice/api/v1/`
- **Current Endpoints:**
  - `vehicleType/get` - Gets vehicle type list

**Repository to Clone:** `vehicleservice` (Vehicle Service)

### 1.4 Core Platform Service (`coreplatformservicev2/api/v1/`)
**Base URL:** `/coreplatformservicev2/api/v1/`
- **Current Endpoints:**
  - `serviceProduct/get` - Gets service/product list

**Repository to Clone:** `coreplatformservicev2` (Core Platform Service)

### 1.5 Export Service (`execto/api/v1/`)
**Base URL:** `/execto/api/v1/`
- **Current Endpoints:**
  - `Delivery/includeDeliveryNew` - Includes delivery
  - `Delivery/excludeDeliveryNew` - Excludes delivery

**Repository to Clone:** `execto` (Export/Execute TO Service)

---

## 2. Required Backend Changes

### 2.1 ASN Service Changes

#### 2.1.1 Modify `TOplan/TOCreateWrapper` Endpoint

**Current Behavior:**
- Accepts single combination of Vehicle Type, Vendor, and No of Vehicles
- Creates single Transport Order

**Required Changes:**
- **Accept multiple combinations** in the request payload
- **Create multiple Transport Orders** (one per combination)
- **Validate** total vehicles count against total containers
- **Return summary** of created Transport Orders

**New Request Payload Structure:**
```json
{
  "AccountID": ["2200000005"],
  "Business": ["BUSINESS1"],
  "SubBusiness": ["SUBBUSINESS1"],
  "Scenario": "IIRD",
  "Source": "Mumbai Port",
  "Destination": "Delhi Plant",
  "asnDetails": [
    {
      "ASNNo": "ASN001234",
      "Containers": ["CNT001", "CNT002"]
    },
    {
      "ASNNo": "ASN001235",
      "Containers": ["CNT003", "CNT004"]
    }
  ],
  "combinations": [
    {
      "VehicleType": "TRUCK_20T",
      "VehicleCapacity": 20,
      "VehicleCapacityUOM": "TON",
      "Vendor": "VENDOR001",
      "NoOfVehicle": 2,
      "Service": "SERVICE001",
      "ModeID": "MODE001"
    },
    {
      "VehicleType": "TRUCK_10T",
      "VehicleCapacity": 10,
      "VehicleCapacityUOM": "TON",
      "Vendor": "VENDOR002",
      "NoOfVehicle": 1,
      "Service": "SERVICE002",
      "ModeID": "MODE001"
    }
  ]
}
```

**New Response Structure:**
```json
{
  "success": true,
  "message": "Successfully created 2 Transport Orders",
  "data": {
    "totalTOsCreated": 2,
    "transportOrders": [
      {
        "TONumber": "TO001234",
        "VehicleType": "TRUCK_20T",
        "Vendor": "VENDOR001",
        "NoOfVehicles": 2,
        "ContainersAllocated": 2
      },
      {
        "TONumber": "TO001235",
        "VehicleType": "TRUCK_10T",
        "Vendor": "VENDOR002",
        "NoOfVehicles": 1,
        "ContainersAllocated": 1
      }
    ],
    "totalContainers": 4,
    "totalVehicles": 3
  }
}
```

**Validation Rules:**
1. Total vehicles across all combinations ≤ Total selected containers
2. Each combination must have valid Vehicle Type, Vendor, and Service
3. No of Vehicles must be > 0 for each combination
4. All selected ASNs must be valid and in pending state

**Error Response:**
```json
{
  "success": false,
  "message": "Total vehicles (5) cannot exceed selected containers (4)",
  "errorCode": "VEHICLE_COUNT_EXCEEDED"
}
```

#### 2.1.2 Add Search Endpoint for ASN/SCN/Container

**New Endpoint:** `delivery/searchDocuments`

**Request Payload:**
```json
{
  "searchType": "ASN", // "ASN" | "SCN" | "CONTAINER"
  "searchValues": ["ASN001234", "ASN001235", "ASN001236"],
  "exactMatch": true,
  "caseSensitive": false,
  "filters": {
    "start_date": "2024-01-01",
    "end_date": "2024-01-31",
    "AccountID": ["2200000005"],
    "Business": ["BUSINESS1"],
    "SubBusiness": ["SUBBUSINESS1"]
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "totalFound": 2,
    "asns": [
      {
        "ASNNo": "ASN001234",
        "SCNNo": "SCN001234",
        "Date": "2024-01-15",
        "Containers": [
          {
            "ContainerNo": "CNT001",
            "Status": "Pending"
          },
          {
            "ContainerNo": "CNT002",
            "Status": "Pending"
          }
        ]
      }
    ],
    "totalContainers": 4
  }
}
```

#### 2.1.3 Enhance `delivery/inbdeliverylist` Endpoint

**Required Changes:**
- Add support for filtering by multiple ASN/SCN/Container numbers
- Add search parameters to existing endpoint

**New Query Parameters:**
- `searchType` - "ASN" | "SCN" | "CONTAINER"
- `searchValues` - Comma-separated list of search values
- `exactMatch` - Boolean
- `caseSensitive` - Boolean

---

### 2.2 Order Service Changes (if needed)

#### 2.2.1 Review `order/createdeliverytoNew` Endpoint

**Action Required:**
- Check if this endpoint needs modification to support multiple TO creation
- If ASN Service handles all TO creation, this may not need changes
- Coordinate with ASN Service team to ensure consistency

---

### 2.3 Database Schema Changes (if applicable)

#### 2.3.1 Transport Order Table
- Ensure table supports multiple TOs with same ASN/Container references
- Check foreign key constraints
- Verify indexing for search performance

#### 2.3.2 Container Allocation Table
- Track which containers are allocated to which Transport Order
- Ensure proper allocation tracking for multiple TOs

---

## 3. Implementation Details

### 3.1 Multiple TO Creation Logic

**Pseudocode:**
```
1. Validate total vehicles ≤ total containers
2. For each combination:
   a. Calculate containers to allocate (based on NoOfVehicle)
   b. Create Transport Order with combination details
   c. Allocate containers to this TO
   d. Update container status
   e. Generate TO Number
3. Return summary of all created TOs
```

### 3.2 Container Allocation Logic

**Rules:**
- Allocate containers sequentially across combinations
- First combination gets first N containers (where N = NoOfVehicle)
- Second combination gets next M containers (where M = NoOfVehicle)
- Continue until all containers are allocated

**Example:**
- Total Containers: 4
- Combination 1: 2 vehicles → Allocates Container 1, 2
- Combination 2: 1 vehicle → Allocates Container 3
- Combination 3: 1 vehicle → Allocates Container 4

### 3.3 Search Implementation

**Search Logic:**
```
1. Parse search values (handle comma/newline separated)
2. Trim whitespace and validate format
3. Based on searchType:
   - ASN: Search in ASN table
   - SCN: Search in SCN table
   - Container: Search in Container table
4. Apply exactMatch and caseSensitive filters
5. Return matching documents with container details
```

---

## 4. API Endpoint Summary

### 4.1 Modified Endpoints

| Endpoint | Service | Method | Changes |
|----------|---------|--------|---------|
| `TOplan/TOCreateWrapper` | ASN Service | POST | Accept multiple combinations, create multiple TOs |
| `delivery/inbdeliverylist` | ASN Service | POST | Add search parameters |

### 4.2 New Endpoints

| Endpoint | Service | Method | Purpose |
|----------|---------|--------|---------|
| `delivery/searchDocuments` | ASN Service | POST | Search by ASN/SCN/Container |

---

## 5. Repositories to Clone

### 5.1 Primary Repository (Most Changes)
```
Repository: asn-a (ASN Platform Service)
Branch: [Check with team for appropriate branch]
Path: [Service path for TOplan and delivery endpoints]
```

### 5.2 Secondary Repositories (Review/Coordination)
```
Repository: order-a (Order Service)
Purpose: Review if any changes needed for consistency

Repository: execto (Export/Execute TO Service)
Purpose: Review container allocation logic if shared
```

### 5.3 Supporting Repositories (No Changes Expected)
```
Repository: vehicleservice (Vehicle Service)
Purpose: Reference only - no changes expected

Repository: coreplatformservicev2 (Core Platform Service)
Purpose: Reference only - no changes expected
```

---

## 6. Testing Requirements

### 6.1 Unit Tests
- Test multiple combination creation
- Test vehicle count validation
- Test container allocation logic
- Test search functionality with various inputs

### 6.2 Integration Tests
- Test end-to-end flow: Search → Select → Create Multiple TOs
- Test error scenarios (vehicle count exceeded, invalid combinations)
- Test with large number of combinations

### 6.3 Performance Tests
- Test search performance with large datasets
- Test multiple TO creation performance
- Test concurrent requests

---

## 7. Error Handling

### 7.1 Validation Errors
- Vehicle count exceeded
- Invalid combination data
- Missing required fields
- Invalid ASN/Container references

### 7.2 Business Logic Errors
- Containers already allocated
- ASN not in pending state
- Invalid vendor/vehicle type combination

### 7.3 System Errors
- Database connection failures
- Transaction rollback scenarios
- Partial TO creation failures

---

## 8. Migration Considerations

### 8.1 Backward Compatibility
- Ensure existing single TO creation still works
- Maintain backward compatibility with current payload structure
- Add new fields as optional initially

### 8.2 Data Migration
- No data migration expected
- Ensure existing TOs are not affected

### 8.3 Deployment Strategy
- Deploy ASN Service changes first
- Coordinate with frontend deployment
- Monitor for any issues post-deployment

---

## 9. Configuration Changes

### 9.1 Environment Variables
- Check if any new configuration needed for search
- Verify TO creation limits/configurations

### 9.2 Database Indexes
- Add indexes on ASN, SCN, Container Number columns for search performance
- Review existing indexes for optimization

---

## 10. Documentation Updates

### 10.1 API Documentation
- Update API docs for modified endpoints
- Document new search endpoint
- Update request/response examples

### 10.2 Developer Documentation
- Document new payload structure
- Document validation rules
- Document error codes

---

## 11. Coordination Points

### 11.1 Frontend Team
- Coordinate payload structure changes
- Align on error handling
- Test integration together

### 11.2 Database Team
- Review schema changes if any
- Optimize queries for search
- Review indexing strategy

### 11.3 QA Team
- Share test scenarios
- Coordinate test data setup
- Plan regression testing

---

## 12. Estimated Effort

### 12.1 Development
- ASN Service modifications: 5-7 days
- Search endpoint implementation: 2-3 days
- Testing and bug fixes: 3-4 days
- **Total: 10-14 days**

### 12.2 Code Review & Deployment
- Code review: 2 days
- Deployment and monitoring: 1 day
- **Total: 3 days**

---

## 13. Risk Assessment

### 13.1 High Risk
- Multiple TO creation logic complexity
- Container allocation edge cases
- Performance impact of search on large datasets

### 13.2 Medium Risk
- Backward compatibility issues
- Integration with existing TO creation flow

### 13.3 Low Risk
- Search functionality (standard implementation)
- UI integration (frontend handles most logic)

---

## 14. Next Steps

1. **Clone ASN Service repository** (`asn-a`)
2. **Review existing `TOplan/TOCreateWrapper` implementation**
3. **Design new payload structure** (coordinate with frontend)
4. **Implement multiple TO creation logic**
5. **Implement search endpoint**
6. **Write unit tests**
7. **Coordinate integration testing with frontend**
8. **Deploy to dev environment**
9. **QA testing**
10. **Production deployment**

---

## 15. Questions to Clarify

1. What is the exact repository name and location for ASN Service?
2. Are there any existing patterns for multiple TO creation in other modules?
3. What is the current container allocation logic?
4. Are there any business rules for container-to-vehicle allocation?
5. What is the expected search performance requirement?
6. Are there any rate limiting considerations for multiple TO creation?
7. What is the rollback strategy if partial TO creation fails?

---

## 16. Contact Information

**Backend Team Contacts:**
- ASN Service Owner: [To be filled]
- Order Service Owner: [To be filled]
- Database Team: [To be filled]

---

**Document Version:** 1.0  
**Last Updated:** [Current Date]  
**Author:** [Your Name]

