# Transport Plan Enhancement - Multi-SCN/Container with Multiple Vehicle-Vendor Combinations

## Part 1: Overview, Search Functionality, and Multi-Selection UI

---

## Overview

Enhance the Transport Plan function to allow:

1. **Search functionality** to find multiple documents by ASN, SCN, or Container Number before Transport Plan
2. Selection of multiple SCNs (ASNs) and Containers across different SCNs
3. Entry of multiple combinations of Vehicle Type + Vendor + No of Vehicles
4. Creation of one Transport Order per combination
5. Validation: Total vehicles ≤ Total selected containers

## Current State Analysis

### Current Selection Mechanism:

- **FCL Import**: Single ASN selection via card click, containers shown in right panel, single container selection
- **LCL Import**: Multiple ASN selection via checkbox (`isOrdersCheckboxSelected`)
- **Container Selection**: Single selection via `onContainerClick()` - only one container can be selected at a time

### Current Dialog:

- Single Vehicle Type dropdown
- Single Vendor dropdown  
- Single No of Vehicles input (readonly, default 1)
- Single Service dropdown
- Creates one Transport Order

### Current Search Mechanism:

- Basic text search via `searchText` and `searchAll` pipe (filters displayed items)
- No dedicated search dialog for ASN/SCN/Container Number
- Search is general text matching, not specific field-based search

## UI/UX Consistency Requirements

**CRITICAL**: All UI changes MUST maintain consistency with existing screens by:

1. **Using Existing Components**: Reuse Angular Material components (`mat-dialog`, `mat-form-field`, `mat-select`, `mat-checkbox`, `mat-chip`, etc.) exactly as they are used in existing dialogs
2. **Using Existing CSS Classes**: Reuse existing classes like `round-border-primary-btn`, `round-border-secondary-btn`, `menu-card`, `menu-card-primary`, `menu-card-normal`, `example-full-width`, `w-300`, etc.
3. **Following Existing Patterns**: Match dialog structure, form layout, button placement, and visual styling from existing `#transportOrder` dialog template
4. **Maintaining Visual Consistency**: Use same colors, spacing, borders, and visual indicators as existing screens
5. **Minimizing New CSS**: Only add new CSS classes if absolutely necessary; prefer inline styles or existing classes

### Reference Templates for UI Patterns:

- **Dialog Structure**: Follow `#transportOrder` template (lines 506-635 in `exim-import-road.component.html`)
- **Form Fields**: Follow form field pattern from lines 517-609 (same classes, same layout)
- **Buttons**: Follow button pattern from lines 47-60 and 614-632 (same classes, same structure)
- **Checkboxes**: Follow checkbox pattern from lines 182-187 (LCL Import) and 409-420 (table checkboxes)
- **Cards**: Follow card pattern from lines 188-248 (ASN cards) and 295-314 (container cards)
- **Search Input**: Follow search pattern from lines 99-109 (existing search wrapper)

## Required UI Changes

### Existing UI Components and Patterns to Follow:

1. **Dialog Structure**: Use `ng-template` with `mat-dialog-content`, `mat-dialog-title`, close icon pattern
2. **Form Fields**: Use `mat-form-field` with `class="example-full-width w-300"` and `floatLabel="always"`
3. **Buttons**: Use `round-border-primary-btn` and `round-border-secondary-btn` classes
4. **Cards**: Use `menu-card`, `menu-card-primary`, `menu-card-normal` classes for selection states
5. **Checkboxes**: Use `mat-checkbox` with existing styling patterns
6. **Layout**: Use FlexLayout directives (`fxLayout`, `fxLayoutAlign`, `fxFlex`)
7. **Icons**: Use `mat-icon` with existing icon names
8. **Chips**: Use `mat-chip-list` and `mat-chip` for removable chips
9. **Radio Buttons**: Use `mat-radio-group` and `mat-radio-button`
10. **Search Pattern**: Follow existing search input pattern with `searchAll` pipe

### 0. Search Dialog for Multiple Documents (NEW)

#### 0.1 Search Dialog Component

**Purpose**: Allow users to search and filter documents by ASN, SCN, or Container Number before proceeding with Transport Plan activity.

**File**: `exim-import-road.component.html` - New template section

**UI/UX Consistency**: This dialog must follow the exact same structure and styling as the existing `#transportOrder` dialog template (lines 506-635) to maintain consistency.

**Dialog Structure** (Following existing pattern from `#transportOrder` template):

```html
<ng-template #searchDocumentsDialog class="allocapp">
  <!-- Dialog Header - Following existing pattern from #transportOrder template (line 507-509) -->
  <div fxLayout="row" fxLayoutAlign="end" class="mt-5">
    <mat-icon class="cursor-pointer" style="font-size: 30px;" (click)="closeSearchDialog()">close</mat-icon>
  </div>
  <h1 matDialogTitle><span class="black-text">Search Documents</span></h1>
  <mat-dialog-content class="mb-10 mt-10">
    <div style="height: auto;">
      <div class="create-order-form mt-10">
        <form class="example-form" [formGroup]="searchForm">
          
          <!-- Search Type Selection - Using mat-radio-group similar to existing patterns -->
          <div fxLayout="row" fxLayoutAlign="space-between center" class="mb-10">
            <mat-form-field class="example-full-width w-300" floatLabel="always">
              <mat-label>Search By</mat-label>
              <mat-select formControlName="searchType" (selectionChange)="onSearchTypeChange()">
                <mat-option value="ASN">ASN Number</mat-option>
                <mat-option value="SCN">SCN Number</mat-option>
                <mat-option value="CONTAINER">Container Number</mat-option>
              </mat-select>
            </mat-form-field>
          </div>

          <!-- Multiple Search Input - Using existing mat-form-field pattern -->
          <div fxLayout="row" fxLayoutAlign="space-between center" class="mb-10">
            <mat-form-field class="example-full-width" floatLabel="always">
              <mat-label>Enter {{searchForm.get('searchType').value}} Numbers</mat-label>
              <textarea 
                matInput 
                formControlName="searchValues" 
                placeholder="Enter multiple values separated by comma or new line&#10;Example: ASN001, ASN002, ASN003"
                rows="4"
                (input)="onSearchInputChange()">
              </textarea>
              <mat-hint>
                <span>You can enter multiple values separated by comma (,) or new line</span>
                <span *ngIf="parsedSearchValues.length > 0" style="display: block; color: var(--color-primary); font-weight: 500; margin-top: 5px;">
                  {{parsedSearchValues.length}} value(s) entered
                </span>
              </mat-hint>
            </mat-form-field>
          </div>

          <!-- Parsed Values Display - Using mat-chip-list pattern -->
          <div *ngIf="parsedSearchValues.length > 0" class="mb-10" style="padding: 15px; background-color: #f5f5f5; border-radius: 4px;">
            <h4 style="margin-bottom: 10px; font-size: 14px; color: #333;">Search Values ({{parsedSearchValues.length}}):</h4>
            <mat-chip-list>
              <mat-chip 
                *ngFor="let value of parsedSearchValues; let i = index" 
                [removable]="true"
                (removed)="removeSearchValue(i)">
                {{value}}
                <mat-icon matChipRemove>cancel</mat-icon>
              </mat-chip>
            </mat-chip-list>
          </div>

          <!-- Search Options - Using mat-checkbox pattern similar to existing checkboxes -->
          <div fxLayout="row" fxLayoutAlign="space-between center" class="mb-10">
            <mat-checkbox [(ngModel)]="exactMatch" [ngModelOptions]="{standalone: true}">
              Exact Match Only
            </mat-checkbox>
            <mat-checkbox [(ngModel)]="caseSensitive" [ngModelOptions]="{standalone: true}">
              Case Sensitive
            </mat-checkbox>
          </div>

          <!-- Search Results Preview -->
          <div *ngIf="searchResults.length > 0" class="mb-10" style="padding: 15px; background-color: #e8f5e9; border-radius: 4px; border: 1px solid #66bb6a;">
            <h4 style="margin-bottom: 10px; color: #2e7d32; font-size: 14px;">Found {{searchResults.length}} matching document(s)</h4>
            <div fxLayout="row" fxLayoutAlign="space-between center" style="font-size: 13px;">
              <span>ASNs: {{getFoundASNCount()}}</span>
              <span *ngIf="selectedTile === 'FCL Import'">Containers: {{getFoundContainerCount()}}</span>
            </div>
          </div>

          <!-- No Results Message -->
          <div *ngIf="searchPerformed && searchResults.length === 0" class="mb-10" style="padding: 15px; background-color: #fff3e0; border-radius: 4px; border: 1px solid #ff9800;">
            <div fxLayout="row" fxLayoutAlign="start center" style="color: #e65100;">
              <mat-icon style="margin-right: 10px;">info</mat-icon>
              <span>No documents found matching the search criteria</span>
            </div>
          </div>

        </form>
      </div>

      <!-- Dialog Footer - Following existing pattern from #transportOrder template (lines 614-632) -->
      <div class="footer-remark">
        <div fxLayout="row" fxLayoutAlign="end">
          <div style="width: 100%; gap: 10px;" fxLayoutAlign="space-between center">
            <button (click)="clearSearch()" type="button" class="round-border-secondary-btn">
              <span fxLayout="row" fxLayoutAlign="center center">
                <span class="bold">Clear</span>
              </span>
            </button>
            <div fxLayout="row" fxLayoutGap="10px">
              <button (click)="closeSearchDialog()" type="button" class="round-border-secondary-btn mr-5">
                <span fxLayout="row" fxLayoutAlign="center center">
                  <span class="bold">Cancel</span>
                </span>
              </button>
              <button 
                (click)="applySearch()" 
                type="button" 
                class="round-border-primary-btn"
                [ngStyle]="{'opacity': (parsedSearchValues.length === 0 || searchResults.length === 0) ? 0.5 : 1}"
                [disabled]="parsedSearchValues.length === 0 || searchResults.length === 0">
                <span fxLayout="row" fxLayoutAlign="center center">
                  <span class="bold">Apply Search ({{searchResults.length}})</span>
                </span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </mat-dialog-content>
</ng-template>
```

**Note**: This dialog structure follows the exact same pattern as the existing `#transportOrder` dialog:

- Same header structure with close icon
- Same `mat-dialog-content` with `mb-10 mt-10` classes
- Same form structure with `create-order-form mt-10` class
- Same footer structure with `footer-remark` class
- Same button classes and layout patterns

#### 0.2 Search Integration in Main UI

**File**: `exim-import-road.component.html` (around line 43-62)

**Add Search Button** (Following existing button pattern from lines 44-61):

```html
<div *ngIf="selectedType == 1" class="absClspod">
  <!-- Add Search Button before Transport Plan button - Using same structure as existing buttons -->
  <div style="height: 35px;" fxLayout="row" fxLayoutAlign="center" fxLayoutGap="10px">
    <button 
      style="height: 45px;" 
      (click)="openSearchDocumentsDialog()"
      type="button"
      class="round-border-secondary-btn">
      <span fxLayout="row">
        <mat-icon class="mr-5">search</mat-icon> 
        <span class="mt-5">Search Documents</span>
      </span>
    </button>
    
    <!-- Existing Transport Plan button - Keep as is -->
    <button *ngIf="checkAccess('TO_PLN')" 
            style="height: 45px;" 
            (click)="openTransportOrderPlan()"
            [ngStyle]="{'opacity': disableCheck() ? '0.5' : '1' }" 
            [disabled]="disableCheck()" 
            type="button"
            class="round-border-primary-btn">
      <span fxLayout="row">
        <mat-icon class="mr-5">post_add</mat-icon> 
        <span class="mt-5">Transport Order Plan</span>
      </span>
    </button>
  </div>
</div>
```

**Note**: Button structure matches existing pattern exactly - same height (45px), same icon/text layout, same classes.

**Add Search Indicator** (Following existing card/indicator patterns):

```html
<!-- Show active search filters - Place after search_wrapper section (after line 110) -->
<div *ngIf="hasActiveSearch()" class="mb-10" style="padding: 10px 15px; background-color: #e3f2fd; border-left: 4px solid var(--color-primary); border-radius: 4px;">
  <div fxLayout="row" fxLayoutAlign="space-between center">
    <div fxLayout="row" fxLayoutAlign="center center" fxLayoutGap="10px" fxLayoutWrap="wrap">
      <mat-icon>filter_alt</mat-icon>
      <span style="font-weight: 500;">Active Search: {{getActiveSearchSummary()}}</span>
      <mat-chip-list>
        <mat-chip 
          *ngFor="let filter of activeSearchFilters" 
          [removable]="true" 
          (removed)="removeSearchFilter(filter)"
          style="margin: 2px;">
          {{filter.type}}: {{filter.value}}
          <mat-icon matChipRemove>cancel</mat-icon>
        </mat-chip>
      </mat-chip-list>
    </div>
    <button (click)="clearAllSearchFilters()" type="button" class="round-border-secondary-btn" style="padding: 5px 10px; font-size: 12px;">
      <mat-icon style="font-size: 18px; height: 18px; width: 18px;">clear_all</mat-icon>
      <span>Clear All</span>
    </button>
  </div>
</div>
```

**Note**: Uses existing chip pattern and button styling. Background color matches existing primary color theme.

#### 0.3 Component TypeScript Changes for Search

**File**: `exim-import-road.component.ts`

**New Properties**:

```typescript
// Search functionality
@ViewChild("searchDocumentsDialog", { static: false }) searchDocumentsDialog: any;
searchType: string = "ASN"; // "ASN" | "SCN" | "CONTAINER"
searchValues: string = "";
parsedSearchValues: string[] = [];
searchResults: any[] = [];
searchPerformed: boolean = false;
exactMatch: boolean = false;
caseSensitive: boolean = false;
activeSearchFilters: any[] = [];
orderListBackup: any[] = []; // Backup of original order list before search
searchForm: FormGroup;
```

**New Methods**:

```typescript
// Search Dialog Management
openSearchDocumentsDialog(): void {
  // Initialize search form
  this.initSearchForm();
  
  // Open dialog
  const dialogRef = this.dialog.open(this.searchDocumentsDialog, {
    width: "600px",
    height: "auto",
    panelClass: "modal-dialog-box-learning",
    data: "",
    disableClose: true,
  });
  
  dialogRef.afterClosed().subscribe((result) => {
    // Handle dialog close
  });
}

closeSearchDialog(): void {
  this.dialog.closeAll();
}

initSearchForm(): void {
  this.searchForm = this.fb.group({
    searchValues: ["", Validators.required],
    searchType: [this.searchType, Validators.required]
  });
}

// Search Input Processing
onSearchInputChange(): void {
  const inputValue = this.searchForm.get("searchValues").value;
  this.parseSearchValues(inputValue);
}

parseSearchValues(input: string): void {
  if (!input || input.trim() === "") {
    this.parsedSearchValues = [];
    return;
  }
  
  // Split by comma, newline, or semicolon
  const values = input
    .split(/[,\n;]/)
    .map(v => v.trim())
    .filter(v => v.length > 0);
  
  // Remove duplicates
  this.parsedSearchValues = [...new Set(values)];
}

removeSearchValue(index: number): void {
  this.parsedSearchValues.splice(index, 1);
  this.updateSearchFormValue();
}

updateSearchFormValue(): void {
  const newValue = this.parsedSearchValues.join(", ");
  this.searchForm.get("searchValues").setValue(newValue);
}

onSearchTypeChange(): void {
  // Clear parsed values when search type changes
  this.parsedSearchValues = [];
  this.searchForm.get("searchValues").setValue("");
  this.searchResults = [];
  this.searchPerformed = false;
}

// Search Execution
applySearch(): void {
  if (this.parsedSearchValues.length === 0) {
    this.scmExportService.showErrorMessage("Please enter at least one search value");
    return;
  }
  
  this.searchPerformed = true;
  this.performSearch();
  this.closeSearchDialog();
}

performSearch(): void {
  // Backup original order list if not already backed up
  if (this.orderListBackup.length === 0) {
    this.orderListBackup = JSON.parse(JSON.stringify(this.orderList));
  }
  
  // Filter order list based on search criteria
  const filteredResults: any[] = [];
  
  this.orderListBackup.forEach((order) => {
    const matchingASNs = order.asnDetails.filter((asn) => {
      return this.matchesSearchCriteria(asn);
    });
    
    if (matchingASNs.length > 0) {
      filteredResults.push({
        ...order,
        asnDetails: matchingASNs
      });
    }
  });
  
  this.orderList = filteredResults;
  this.searchResults = this.flattenSearchResults(filteredResults);
  
  // Update active search filters
  this.updateActiveSearchFilters();
  
  // Show results message
  if (this.searchResults.length > 0) {
    this.scmExportService.showSuccessMessage(
      `Found ${this.searchResults.length} matching document(s)`
    );
  } else {
    this.scmExportService.showErrorMessage("No documents found matching search criteria");
  }
  
  // Refresh UI
  this.cdr.detectChanges();
}

matchesSearchCriteria(asn: any): boolean {
  const searchField = this.getSearchField(asn);
  const searchValue = searchField?.toString() || "";
  
  return this.parsedSearchValues.some(searchTerm => {
    if (this.exactMatch) {
      if (this.caseSensitive) {
        return searchValue === searchTerm;
      } else {
        return searchValue.toLowerCase() === searchTerm.toLowerCase();
      }
    } else {
      if (this.caseSensitive) {
        return searchValue.includes(searchTerm);
      } else {
        return searchValue.toLowerCase().includes(searchTerm.toLowerCase());
      }
    }
  });
}

getSearchField(asn: any): string {
  switch (this.searchType) {
    case "ASN":
      return asn.ASNNo || "";
    case "SCN":
      return asn.SCNNo || "";
    case "CONTAINER":
      // For container search, check if any container matches
      if (asn.Container && asn.Container.length > 0) {
        const matchingContainers = asn.Container.filter(cont => 
          this.parsedSearchValues.some(searchTerm => {
            const containerNo = cont.containerNo || "";
            if (this.exactMatch) {
              return this.caseSensitive 
                ? containerNo === searchTerm 
                : containerNo.toLowerCase() === searchTerm.toLowerCase();
            } else {
              return this.caseSensitive 
                ? containerNo.includes(searchTerm)
                : containerNo.toLowerCase().includes(searchTerm.toLowerCase());
            }
          })
        );
        return matchingContainers.length > 0 ? "MATCH" : "";
      }
      return "";
    default:
      return "";
  }
}

flattenSearchResults(filteredOrders: any[]): any[] {
  const results: any[] = [];
  
  filteredOrders.forEach(order => {
    order.asnDetails.forEach(asn => {
      if (this.searchType === "CONTAINER" && asn.Container) {
        // For container search, include each matching container
        asn.Container.forEach(cont => {
          if (this.parsedSearchValues.some(term => {
            const containerNo = cont.containerNo || "";
            return this.exactMatch
              ? (this.caseSensitive ? containerNo === term : containerNo.toLowerCase() === term.toLowerCase())
              : (this.caseSensitive ? containerNo.includes(term) : containerNo.toLowerCase().includes(term.toLowerCase()));
          })) {
            results.push({
              type: "Container",
              asn: asn.ASNNo,
              scn: asn.SCNNo,
              containerNo: cont.containerNo,
              containerType: cont.ContainerType,
              asnDetail: asn,
              containerDetail: cont
            });
          }
        });
      } else {
        results.push({
          type: this.searchType,
          asn: asn.ASNNo,
          scn: asn.SCNNo,
          asnDetail: asn
        });
      }
    });
  });
  
  return results;
}

// Search Filter Management
updateActiveSearchFilters(): void {
  this.activeSearchFilters = this.parsedSearchValues.map(value => ({
    type: this.searchType,
    value: value,
    display: `${this.searchType}: ${value}`
  }));
}

removeSearchFilter(filter: any): void {
  const index = this.parsedSearchValues.findIndex(v => v === filter.value);
  if (index !== -1) {
    this.parsedSearchValues.splice(index, 1);
    if (this.parsedSearchValues.length === 0) {
      this.clearAllSearchFilters();
    } else {
      this.updateSearchFormValue();
      this.performSearch();
    }
  }
}

clearSearch(): void {
  this.parsedSearchValues = [];
  this.searchForm.get("searchValues").setValue("");
  this.searchResults = [];
  this.searchPerformed = false;
}

clearAllSearchFilters(): void {
  this.clearSearch();
  this.activeSearchFilters = [];
  
  // Restore original order list
  if (this.orderListBackup.length > 0) {
    this.orderList = JSON.parse(JSON.stringify(this.orderListBackup));
    this.orderListBackup = [];
  }
  
  this.cdr.detectChanges();
  this.scmExportService.showSuccessMessage("Search filters cleared");
}

hasActiveSearch(): boolean {
  return this.activeSearchFilters.length > 0;
}

getActiveSearchSummary(): string {
  if (this.activeSearchFilters.length === 0) return "";
  return `${this.activeSearchFilters.length} filter(s) active`;
}

getFoundASNCount(): number {
  return this.searchResults.filter(r => r.type === "ASN" || r.type === "SCN").length;
}

getFoundContainerCount(): number {
  return this.searchResults.filter(r => r.type === "Container").length;
}
```

#### 0.4 Modified Methods for Search Integration

**`getOrderList()` (line 671)**:

```typescript
getOrderList() {
  // ... existing code ...
  
  // After fetching order list, check if search is active
  if (this.hasActiveSearch()) {
    // Reapply search filters to newly fetched data
    this.orderListBackup = [...this.orderList];
    this.performSearch();
  } else {
    // Store backup for future searches
    this.orderListBackup = [...this.orderList];
  }
}
```

**`openTransportOrderPlan()` (line 789)**:

```typescript
openTransportOrderPlan() {
  // Check if search is active and warn user
  if (this.hasActiveSearch()) {
    const confirmed = confirm(
      "You have active search filters. Only matching documents will be included in Transport Plan. Continue?"
    );
    if (!confirmed) {
      return;
    }
  }
  
  // ... rest of existing code ...
}
```

### 1. Multi-SCN/Container Selection UI Changes

#### 1.1 FCL Import - Left Panel (ASN Cards)

**File**: `exim-import-road.component.html` (lines 180-252)

**Current Behavior**:

- Click on ASN card selects it and shows containers
- Only one ASN can be selected at a time (`isSelected`)

**Required Changes**:

- Add checkbox to each ASN card for multi-selection
- Add new property: `isASNSelectedForPlanning` (boolean)
- Allow multiple ASNs to be selected simultaneously
- Visual indicator for selected ASNs (similar to LCL checkbox style)

**Implementation** (Following existing checkbox pattern from LCL Import - lines 182-187):

```html
<!-- Add checkbox before ASN card for FCL Import when selectedType == 1 -->
<!-- Use same structure as existing LCL Import checkbox (line 182-187) -->
<div *ngIf="selectedType == 1 && selectedTile === 'FCL Import'" class="checbox_cards">
  <mat-checkbox 
    [disabled]="item.disable || item.checkboxDisabled"
    [(ngModel)]="item.isASNSelectedForPlanning" 
    [checked]="item.isASNSelectedForPlanning"
    (change)="onASNSelectionChange($event, item)"
    style="width: 100%; height: 100%;">
  </mat-checkbox>
</div>
```

**Note**: Uses exact same checkbox structure and classes as existing LCL Import checkbox (`checbox_cards` class, same disabled logic, same styling).

#### 1.2 FCL Import - Right Panel (Container List)

**File**: `exim-import-road.component.html` (lines 289-316)

**Current Behavior**:

- Single container selection via click
- `onContainerClick()` sets only one container as selected

**Required Changes**:

- Add checkbox to each container card
- Allow multiple container selection
- Add property: `isContainerSelectedForPlanning` (boolean)
- Show count of selected containers

**Implementation** (Following existing container card pattern from lines 295-314):

```html
<!-- Modify container card to support checkbox selection -->
<!-- Use existing menu-card classes and structure -->
<div class="scroll-data scrollable-content" style="width: 100%;">
  <div *ngFor="let item of containerList" class="menu-card" 
       [ngClass]="{'menu-card-primary': item.isSelected || item.isContainerSelectedForPlanning, 
                   'menu-card-normal': !item.isSelected && !item.isContainerSelectedForPlanning}"
       (click)="onContainerClick(item)">
    <!-- Add checkbox in top-right corner, similar to ASN checkbox pattern -->
    <div style="position: absolute; top: 5px; right: 5px; z-index: 10;" (click)="$event.stopPropagation()">
      <mat-checkbox 
        [(ngModel)]="item.isContainerSelectedForPlanning"
        [checked]="item.isContainerSelectedForPlanning"
        (change)="onContainerSelectionChange($event, item)"
        [disabled]="item.disable || item.checkboxDisabled">
      </mat-checkbox>
    </div>
    <!-- Existing container details - keep as is (lines 298-313) -->
    <div fxLayout="row" fxLayoutAlign="space-between center">
      <div>
        <span [ngClass]="{'bold_white': item.isSelected || item.isContainerSelectedForPlanning, 'bold_black': !item.isSelected && !item.isContainerSelectedForPlanning}">
          Container No: {{item.containerNo}}
        </span>
      </div>
    </div>
    <div fxLayout="row" fxLayoutAlign="space-between center">
      <div>
        <span class="font_11">Container Type {{item.ContainerType}}</span>
      </div>
    </div>
    <div fxLayout="row" fxLayoutAlign="space-between center">
      <div>
        <span class="font_11">Seal No : {{item.SealNo}}</span>
      </div>
    </div>
  </div>
  
  <!-- Add selected count display - following existing summary patterns -->
  <div *ngIf="getSelectedContainersCount() > 0" class="mt-10 mb-10" style="padding: 10px; background-color: #f5f5f5; border-radius: 4px; text-align: center;">
    <span class="bold">Selected Containers: {{getSelectedContainersCount()}}</span>
  </div>
</div>
```

**Note**:

- Uses existing `menu-card`, `menu-card-primary`, `menu-card-normal` classes
- Checkbox positioned absolutely to not interfere with card click
- Selected state uses same visual styling as existing `isSelected` state
- Count display follows existing summary card patterns

#### 1.3 LCL Import - Multi-ASN Selection

**Current State**: Already supports multi-selection via `isOrdersCheckboxSelected`

**Required Changes**:

- Ensure this selection is used for planning
- No changes needed to selection mechanism
- May need to enhance visual feedback

---

**Continue to Part 2 for Transport Plan Dialog Enhancement, Component TypeScript Changes, Validation, Data Structures, CSS, Testing, and Implementation details.**

