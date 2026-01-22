# Transport Plan Enhancement - Multi-SCN/Container with Multiple Vehicle-Vendor Combinations

## Part 2: Transport Plan Dialog, TypeScript Changes, Validation, and Implementation

---

### 2. Transport Plan Dialog Enhancement

#### 2.1 Dialog Header Changes

**File**: `exim-import-road.component.html` (lines 506-510)

**Add Summary Section** (Following existing dialog content structure from `#transportOrder` template):

```html
<!-- Add after dialog title, before form - following existing dialog structure pattern -->
<div class="mb-10" style="padding: 12px; background-color: #f5f5f5; border-radius: 4px;" *ngIf="selectedTile === 'FCL Import'">
  <div fxLayout="row" fxLayoutAlign="space-between center">
    <span class="bold">Selected SCNs: {{getSelectedSCNCount()}}</span>
    <span class="bold">Selected Containers: {{getSelectedContainersCount()}}</span>
  </div>
</div>
<div class="mb-10" style="padding: 12px; background-color: #f5f5f5; border-radius: 4px;" *ngIf="selectedTile === 'LCL Import'">
  <div fxLayout="row" fxLayoutAlign="space-between center">
    <span class="bold">Selected SCNs: {{getSelectedSCNCount()}}</span>
  </div>
</div>
```

**Note**: Uses same styling pattern as existing summary sections (background color, padding, border-radius matching existing UI elements).

#### 2.2 Replace Single Form with Dynamic FormArray

**File**: `exim-import-road.component.html` (lines 514-609)

**Current Structure**: Single form with one set of fields

**New Structure**: FormArray with multiple rows, each row representing one combination

**UI/UX Consistency**: All form fields must use the same structure and classes as existing form fields in `#transportOrder` template:

- `mat-form-field` with `class="example-full-width w-300"` and `floatLabel="always"`
- Same layout pattern with `fxLayout="row" fxLayoutAlign="space-between center"`
- Same select pattern with search input inside (lines 562-570, 581-589)
- Same button styling and layout

**Implementation** (Following existing form structure from lines 514-610):

```html
<form class="example-form" [formGroup]="orderDetailsForm">
  <!-- Source/Destination (common for all combinations) -->
  <div fxLayout="row" fxLayoutAlign="space-between center">
    <mat-form-field id="source" class="example-full-width w-300" floatLabel="always">
      <mat-label>Source</mat-label>
      <input formControlName="source" readonly matInput type="text">
    </mat-form-field>
    <mat-form-field id="destination" class="example-full-width w-300" floatLabel="always">
      <mat-label>Destination</mat-label>
      <input formControlName="destination" readonly matInput type="text">
    </mat-form-field>
  </div>

  <!-- Service (common for all combinations) -->
  <div fxLayout="row" fxLayoutAlign="space-between center">
    <mat-form-field id="service" class="example-full-width w-300" floatLabel="always">
      <mat-label>Service</mat-label>
      <mat-select formControlName="service" (selectionChange)="onServiceSelected($event)">
        <mat-option *ngFor="let serv of serviceList" [value]="serv.ServiceProductID">
          {{serv.Description}} ({{serv.ServiceProductID}})
        </mat-option>
      </mat-select>
    </mat-form-field>
  </div>

  <!-- Mode of Transport (conditional, common) -->
  <div *ngIf="showModeOfTransportDropdown" fxLayout="row" fxLayoutAlign="space-between center">
    <mat-form-field id="modeofTransport" class="example-full-width w-300" floatLabel="always">
      <mat-label>Mode of Transport</mat-label>
      <mat-select formControlName="modeofTransport">
        <mat-option *ngFor="let mode of modeofTransportation" [value]="mode.ModeOfTransportID">
          {{mode.Name}}
        </mat-option>
      </mat-select>
    </mat-form-field>
  </div>

  <!-- Dynamic Vehicle-Vendor Combinations -->
  <div formArrayName="vehicleVendorCombinations" class="combinations-section">
      <div class="section-header" fxLayout="row" fxLayoutAlign="space-between center" style="margin-bottom: 15px; padding-bottom: 10px; border-bottom: 1px solid #e0e0e0;">
        <h3 style="font-size: 16px; color: #333; margin: 0;">Vehicle-Vendor Combinations</h3>
        <button type="button" (click)="addVehicleVendorCombination()" 
                class="round-border-primary-btn" style="height: 35px;">
          <span fxLayout="row" fxLayoutAlign="center center">
            <mat-icon class="mr-5" style="font-size: 18px;">add</mat-icon>
            <span>Add Combination</span>
          </span>
        </button>
      </div>

    <!-- Validation Message - Following existing error message patterns -->
    <div *ngIf="showVehicleCountValidation" 
         [ngStyle]="{'background-color': !isVehicleCountValid() ? '#ffebee' : '#e8f5e9', 
                     'color': !isVehicleCountValid() ? '#c62828' : '#2e7d32',
                     'border': !isVehicleCountValid() ? '1px solid #ef5350' : '1px solid #66bb6a'}"
         style="padding: 10px; margin: 10px 0; border-radius: 4px; display: flex; align-items: center; gap: 10px;">
      <mat-icon [style.color]="!isVehicleCountValid() ? '#c62828' : '#2e7d32'">warning</mat-icon>
      <span>Total Vehicles: {{getTotalVehiclesCount()}} / Selected Containers: {{getSelectedContainersCount()}}</span>
    </div>

    <!-- Combination Rows - Using existing card-like styling -->
    <div *ngFor="let combination of vehicleVendorCombinations.controls; let i = index" 
         [formGroupName]="i" 
         style="margin-bottom: 15px; padding: 15px; background-color: #f5f5f5; border-radius: 4px; border: 1px solid #e0e0e0;">
      <div class="row-header" fxLayout="row" fxLayoutAlign="space-between center" style="margin-bottom: 10px; padding-bottom: 10px; border-bottom: 1px solid #e0e0e0;">
        <span class="row-number" style="font-weight: 500; color: #333;">Combination {{i + 1}}</span>
        <button type="button" (click)="removeVehicleVendorCombination(i)" 
                *ngIf="vehicleVendorCombinations.length > 1"
                class="round-border-secondary-btn" style="padding: 5px 10px; min-width: auto;">
          <mat-icon style="font-size: 18px; color: #f44336;">delete</mat-icon>
        </button>
      </div>

      <div fxLayout="row" fxLayoutAlign="space-between center" fxLayoutGap="10px">
        <!-- Vehicle Type - Following existing pattern from line 560-571 -->
        <mat-form-field id="vehicletype" class="example-full-width w-300" floatLabel="always" fxFlex="30">
          <mat-label>Vehicle Type</mat-label>
          <mat-select formControlName="vehicleType" 
                      (selectionChange)="onVehicleTypeChange(i)">
            <input style="width: 88%; height: 20px; padding: 10px; margin: 10px;" 
                   type="text" [formControl]="searchVehicleTypeControls[i]"
                   (keydown)="$event.stopPropagation()" #search autocomplete="off"
                   placeholder="Search" aria-label="Search" />
            <mat-option *ngFor="let veh of vehicleTypeList | searchAll : searchVehicleTypeControls[i].value" 
                        [value]="veh">
              {{veh.VehicleType}} ({{veh.GVW_FromCapacity}} {{veh.VehicleCapacityUOM}} - 
              {{veh.GVW_ToCapacity}} {{veh.VehicleCapacityUOM}})
            </mat-option>
          </mat-select>
        </mat-form-field>

        <!-- Vendor - Following existing pattern from line 579-590 -->
        <mat-form-field id="vendor" class="example-full-width w-300" floatLabel="always" fxFlex="30">
          <mat-label>Vendor</mat-label>
          <mat-select formControlName="vendor" 
                      (selectionChange)="onVendorChange(i)">
            <input style="width: 88%; height: 20px; padding: 10px; margin: 10px;" 
                   type="text" [formControl]="searchVendorControls[i]"
                   (keydown)="$event.stopPropagation()" #search autocomplete="off"
                   placeholder="Search" aria-label="Search" />
            <mat-option *ngFor="let vendor of vendorList | searchAll : searchVendorControls[i].value" 
                        [value]="vendor.VendorID">
              {{vendor.Name}} ({{vendor.VendorID}})
            </mat-option>
          </mat-select>
        </mat-form-field>

        <!-- No of Vehicles - Following existing pattern from line 572-575 -->
        <mat-form-field id="vehicles" class="example-full-width w-300" floatLabel="always" fxFlex="20">
          <mat-label>No Of Vehicles</mat-label>
          <input formControlName="noOfVehicles" 
                 matInput type="number" 
                 min="1" 
                 [max]="getMaxVehiclesForCombination(i)"
                 (input)="onVehicleCountChange()">
          <mat-hint>Max: {{getMaxVehiclesForCombination(i)}}</mat-hint>
        </mat-form-field>

        <!-- Allocated Containers Display (readonly) - Following existing readonly input pattern -->
        <mat-form-field class="example-full-width w-300" floatLabel="always" fxFlex="15">
          <mat-label>Allocated</mat-label>
          <input formControlName="allocatedContainers" readonly matInput type="text">
        </mat-form-field>
      </div>
    </div>
  </div>
</form>
```

#### 2.3 Dialog Footer Changes

**File**: `exim-import-road.component.html` (lines 614-632)

**Add Validation Summary** (Following existing footer pattern from lines 614-632):

```html
<div class="footer-remark">
  <!-- Validation Summary - Following existing error display patterns -->
  <div *ngIf="!isVehicleCountValid()" style="padding: 10px; margin-bottom: 10px; background-color: #ffebee; color: #c62828; border-radius: 4px; display: flex; align-items: center; gap: 10px;">
    <mat-icon style="color: #c62828;">error</mat-icon>
    <span>Total vehicles ({{getTotalVehiclesCount()}}) cannot exceed selected containers ({{getSelectedContainersCount()}})</span>
  </div>

  <div fxLayout="row" fxLayoutAlign="end">
    <div style="width: 100%; gap: 10px;" fxLayoutAlign="center center">
      <button (click)="saveChanges()" 
              [ngStyle]="{'opacity': (orderDetailsForm.status != 'VALID' || !isVehicleCountValid()) ? 0.5 : 1}"
              type="button" 
              style="min-width: 60px;" 
              [disabled]="orderDetailsForm.status != 'VALID' || !isVehicleCountValid()"
              class="round-border-primary-btn">
        <span fxLayout="row" fxLayoutAlign="center center">
          <span class="bold">Create Transport Orders ({{vehicleVendorCombinations.length}})</span>
        </span>
      </button>
      <button (click)="closeDialog()" type="button" class="round-border-secondary-btn mr-5">
        <span fxLayout="row" fxLayoutAlign="center center">
          <span class="bold">Cancel</span>
        </span>
      </button>
    </div>
  </div>
</div>
```

**Note**: Footer structure matches existing pattern exactly - same classes, same layout, same button styling.

### 3. Component TypeScript Changes

#### 3.1 New Properties

**File**: `exim-import-road.component.ts`

```typescript
// Multi-selection tracking
selectedASNsForPlanning: any[] = []; // Array of selected ASN objects
selectedContainersForPlanning: any[] = []; // Array of selected container objects across all ASNs

// FormArray for combinations
vehicleVendorCombinations: FormArray;
searchVehicleTypeControls: FormControl[] = [];
searchVendorControls: FormControl[] = [];

// Validation
showVehicleCountValidation: boolean = false;
totalSelectedContainers: number = 0;
```

#### 3.2 Modified Methods

**`openTransportOrderPlan()` (line 789)**:

- Collect all selected ASNs (not just headerObj)
- Collect all selected containers across all selected ASNs
- Initialize FormArray with at least one combination
- Calculate total selected containers

**`initForm()` (line 624)**:

- Change from FormGroup to support FormArray
- Initialize `vehicleVendorCombinations` FormArray
- Add first combination row by default
- Initialize search controls for each row

**`onContainerClick()` (line 582)**:

- Modify to support checkbox-based multi-selection
- Update `isContainerSelectedForPlanning` instead of single selection
- Maintain backward compatibility for display selection

#### 3.3 New Methods

```typescript
// Selection Management
onASNSelectionChange(event: any, asnItem: any): void {
  // Toggle ASN selection
  // Update container list visibility
  // Track selected ASNs
}

onContainerSelectionChange(event: any, container: any): void {
  // Toggle container selection
  // Update total count
  // Validate vehicle counts
}

getSelectedSCNCount(): number {
  // Count selected ASNs
}

getSelectedContainersCount(): number {
  // Count all selected containers across all selected ASNs
}

// FormArray Management
addVehicleVendorCombination(): void {
  // Add new combination row to FormArray
  // Initialize with default values
  // Add search controls
}

removeVehicleVendorCombination(index: number): void {
  // Remove combination at index
  // Recalculate allocations
}

// Validation
onVehicleCountChange(): void {
  // Recalculate total vehicles
  // Validate against container count
  // Update max limits for each row
}

getTotalVehiclesCount(): number {
  // Sum all noOfVehicles from all combinations
}

isVehicleCountValid(): boolean {
  // Check if total vehicles <= total containers
}

getMaxVehiclesForCombination(index: number): number {
  // Calculate remaining available vehicles for this combination
  // Based on total containers and other combinations
}

// Container Allocation Logic
allocateContainersToCombinations(): void {
  // Distribute containers across combinations based on vehicle counts
  // Track which containers go to which combination
}

// Save Logic
saveChanges(): void {
  // Validate all combinations
  // For each combination:
  //   - Create payload with allocated containers
  //   - Call API to create Transport Order
  //   - Handle success/error for each
  // Show summary of created TOs
}
```

#### 3.4 Enhanced Save Method

**Current**: `saveChanges()` creates one TO

**New**: Create multiple TOs, one per combination

```typescript
saveChanges(): void {
  if (!this.isVehicleCountValid()) {
    this.scmExportService.showErrorMessage("Total vehicles exceed selected containers");
    return;
  }

  // Allocate containers to combinations
  const allocations = this.allocateContainersToCombinations();
  
  // Create TOs for each combination
  const toPromises = [];
  
  this.vehicleVendorCombinations.controls.forEach((combination, index) => {
    const payload = {
      AccountID: this.rolesFilters.roles.AccountId,
      Business: this.rolesFilters.Business,
      SubBusiness: this.rolesFilters.SubBusiness,
      Scenario: "IIRD",
      Source: this.orderDetailsForm.value.source,
      Destination: this.orderDetailsForm.value.destination,
      VehicleType: combination.value.vehicleType?.VehicleType,
      VehicleCapacity: combination.value.vehicleType?.VehicleCapacity,
      VehicleCapacityUOM: combination.value.vehicleType?.VehicleCapacityUOM,
      NoOfVehicle: combination.value.noOfVehicles,
      Vendor: combination.value.vendor,
      Service: this.orderDetailsForm.value.service,
      ModeID: this.orderDetailsForm.value.modeofTransport || "",
      asnDetails: this.buildASNDetailsForCombination(allocations[index])
    };

    toPromises.push(
      this.scmExportService.callASN_SCM("TOplan/TOCreateWrapper", payload)
    );
  });

  // Execute all TO creations
  Promise.all(toPromises).then((results) => {
    // Handle results
    const successCount = results.filter(r => r.success).length;
    const failCount = results.length - successCount;
    
    if (successCount > 0) {
      this.scmExportService.showSuccessMessage(
        `${successCount} Transport Order(s) created successfully`
      );
    }
    if (failCount > 0) {
      this.scmExportService.showErrorMessage(
        `${failCount} Transport Order(s) failed to create`
      );
    }
    
    this.dialog.closeAll();
    this.getOrderList();
  });
}

buildASNDetailsForCombination(allocatedContainers: any[]): any[] {
  // Build asnDetails array with only allocated containers for this combination
  // Group by ASN and include only relevant containers
}
```

### 4. Validation Logic

#### 4.1 Real-time Validation

- **Vehicle Count Validation**: Show error when total vehicles > selected containers
- **Per-row Max Validation**: Each row's max vehicles = remaining available containers
- **Required Field Validation**: All combinations must have Vehicle Type, Vendor, and No of Vehicles

#### 4.2 Disable Check Enhancement

**File**: `exim-import-road.component.ts` - `disableCheck()` (line 979)

**Current**: Checks if containers/orders are selected

**Enhanced**:

```typescript
disableCheck(): boolean {
  if (this.selectedTile === "FCL Import") {
    // Check if at least one ASN is selected for planning
    const selectedASNs = this.orderList.flatMap(order => 
      order.asnDetails.filter(asn => asn.isASNSelectedForPlanning)
    );
    
    if (selectedASNs.length === 0) {
      return true;
    }
    
    // Check if at least one container is selected across all selected ASNs
    const selectedContainers = selectedASNs.flatMap(asn => 
      asn.Container?.filter(cont => cont.isContainerSelectedForPlanning) || []
    );
    
    return selectedContainers.length === 0 || 
           selectedASNs.some(asn => !asn.BOEFlag);
  }
  
  // LCL Import - existing logic
  // ...
}
```

### 5. Data Structure Changes

#### 5.1 ASN Object Enhancement

```typescript
interface ASNDetail {
  // Existing properties...
  isASNSelectedForPlanning?: boolean; // New
  Container?: ContainerDetail[]; // Enhanced
}

interface ContainerDetail {
  // Existing properties...
  isContainerSelectedForPlanning?: boolean; // New
  allocatedToCombination?: number; // Track which combination uses this container
  parentASN?: ASNDetail; // Reference to parent ASN
}
```

#### 5.2 Form Structure

```typescript
orderDetailsForm: FormGroup = {
  source: FormControl,
  destination: FormControl,
  service: FormControl,
  modeofTransport: FormControl,
  vehicleVendorCombinations: FormArray<FormGroup> // New
}

// Each combination FormGroup:
{
  vehicleType: FormControl,
  vendor: FormControl,
  noOfVehicles: FormControl,
  allocatedContainers: FormControl (readonly, calculated)
}
```

### 6. UI/UX Enhancements

#### 6.1 Visual Indicators

- Highlight selected ASNs with different background color
- Show checkbox state clearly
- Display selected count badges
- Show allocation status for containers

#### 6.2 User Guidance

- Tooltip explaining multi-selection
- Help text for combination rows
- Clear error messages for validation
- Progress indicator during TO creation

#### 6.3 Responsive Design

- Ensure dialog is scrollable for many combinations
- Responsive layout for combination rows
- Mobile-friendly checkbox sizes

### 7. CSS Changes

**File**: `exim-import-road.component.css`

**IMPORTANT**: Minimize new CSS. Reuse existing classes and styles wherever possible. Only add new CSS if absolutely necessary for functionality that cannot be achieved with existing classes.

**Additional CSS for Search Dialog** (Only if needed - try to use existing classes first):

```css
/* Search Dialog Styles */
.search-dialog-content {
  min-height: 400px;
}

.search-type-section {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #f5f5f5;
  border-radius: 4px;
}

.search-type-section h3 {
  margin-bottom: 15px;
  font-size: 16px;
  color: #333;
}

.search-input-section {
  margin-bottom: 20px;
}

.hint-count {
  display: block;
  margin-top: 5px;
  font-size: 12px;
  color: #2e6bdc;
  font-weight: 500;
}

.parsed-values-section {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #e3f2fd;
  border-radius: 4px;
}

.parsed-values-section h4 {
  margin-bottom: 10px;
  font-size: 14px;
  color: #333;
}

.chips-container {
  max-height: 150px;
  overflow-y: auto;
}

.search-options-section {
  margin-bottom: 20px;
  display: flex;
  gap: 20px;
}

.search-results-preview {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #e8f5e9;
  border-radius: 4px;
  border: 1px solid #66bb6a;
}

.search-results-preview h4 {
  margin-bottom: 10px;
  color: #2e7d32;
}

.results-summary {
  display: flex;
  gap: 20px;
  font-size: 13px;
}

.summary-item {
  display: flex;
  align-items: center;
  gap: 5px;
}

.no-results-message {
  padding: 15px;
  background-color: #fff3e0;
  border-radius: 4px;
  border: 1px solid #ff9800;
  display: flex;
  align-items: center;
  gap: 10px;
  color: #e65100;
}

.active-search-indicator {
  padding: 10px 15px;
  background-color: #e3f2fd;
  border-left: 4px solid #2e6bdc;
  margin-bottom: 15px;
  border-radius: 4px;
}
```

**File**: `exim-import-road.component.css`

**Note**: Most styling can be achieved using inline styles or existing classes. Only add new CSS classes if absolutely necessary. The following are minimal additions:

```css
/* Only add if existing classes cannot be used */
/* Most styling should use existing menu-card classes and inline styles */

/* If needed for container checkbox positioning - but try inline styles first */
.container-checkbox-wrapper {
  position: absolute;
  top: 5px;
  right: 5px;
  z-index: 10;
}

/* Selection summary can use existing card patterns - no new CSS needed */
/* Combination rows can use inline styles with existing color variables */
/* Validation messages can use inline styles matching existing error patterns */
```

**Recommendation**: Use existing CSS classes and inline styles matching existing patterns. Avoid creating new CSS classes unless functionality cannot be achieved otherwise.

### 8. Testing Considerations

#### 8.1 Test Scenarios

**Search Functionality**:

1. Search by ASN with single value → filters correctly
2. Search by ASN with multiple values (comma-separated) → filters all matches
3. Search by ASN with multiple values (newline-separated) → filters all matches
4. Search by SCN with exact match → filters correctly
5. Search by SCN with partial match → filters correctly
6. Search by Container Number → filters and shows matching containers
7. Case-sensitive search → works correctly
8. Case-insensitive search → works correctly
9. Remove individual search filter → updates results correctly
10. Clear all search filters → restores full list
11. Search with no results → shows appropriate message
12. Apply search, then fetch new orders → search re-applied correctly

**Multi-Selection**:

13. Select single ASN, single container → should work as before
14. Select multiple ASNs, multiple containers → new functionality
15. Select ASNs after search filter → only filtered ASNs available

**Combinations**:

16. Add/remove combination rows
17. Vehicle count validation (exceed, equal, less than containers)
18. Create multiple TOs successfully
19. Handle partial failures in TO creation

**Integration**:

20. Search → Select → Plan → Create TOs (complete flow)
21. Multi-leg scenarios with new selection
22. LCL Import with multiple ASNs
23. Search filters persist during Transport Plan dialog

#### 8.2 Edge Cases

- No containers selected
- All containers selected
- Single combination with all vehicles
- Multiple combinations with equal distribution
- Uneven container distribution

## Implementation Sequence

1. **Phase 0**: Search functionality for multiple documents (ASN/SCN/Container)
2. **Phase 1**: Multi-selection UI (ASNs and Containers)
3. **Phase 2**: FormArray structure and combination rows
4. **Phase 3**: Validation logic
5. **Phase 4**: Save logic for multiple TOs
6. **Phase 5**: Testing and refinement

## User Flow with Search

### Complete Flow:

1. **User clicks "Search Documents" button**

   - Search dialog opens
   - User selects search type (ASN/SCN/Container)
   - User enters multiple values (comma/newline separated)
   - System parses and displays entered values as chips
   - User clicks "Apply Search"
   - System filters order list based on search criteria
   - Active search filters displayed as chips above order list

2. **User reviews filtered results**

   - Only matching ASNs/Containers are displayed
   - User can remove individual search filters
   - User can clear all filters to restore full list

3. **User proceeds with Transport Plan**

   - User selects multiple ASNs/Containers from filtered results
   - User clicks "Transport Order Plan"
   - System warns if search filters are active
   - Transport Plan dialog opens with selected items

4. **User configures Vehicle-Vendor combinations**

   - User adds multiple combinations
   - System validates vehicle count against containers
   - User saves to create multiple Transport Orders

## Backward Compatibility

- Maintain existing single-selection behavior as fallback
- Add feature flag if needed for gradual rollout
- Ensure existing API calls still work for single TO creation

---

**End of Part 2. Refer to Part 1 for Overview, Search Functionality, and Multi-Selection UI details.**

