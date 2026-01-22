# Repositories to Clone for Transport Plan Enhancement

## Summary

This document lists the backend repositories that need to be cloned for implementing the Transport Plan enhancement functionality.

---

## Primary Repository (Most Changes Required)

### 1. ASN Platform Service (`asn-a`)
**Priority:** HIGH  
**Changes Required:** MAJOR

**Purpose:**
- Modify `TOplan/TOCreateWrapper` endpoint to support multiple Transport Order creation
- Add new `delivery/searchDocuments` endpoint for ASN/SCN/Container search
- Enhance `delivery/inbdeliverylist` endpoint with search capabilities

**Key Files to Modify:**
- `TOplan/TOCreateWrapper` controller/service
- `delivery/inbdeliverylist` controller/service
- Database models for Transport Order and Container allocation

**Repository Details:**
```
Repository Name: asn-a (or ASN Platform Service)
Service Path: /asn-a/api/v1/
Base URL Pattern: http://scmpfdev.ril.com/asn-a/api/v1/
```

**Clone Command:**
```bash
# Contact backend team for exact repository URL
git clone [ASN_SERVICE_REPO_URL]
cd asn-a
```

---

## Secondary Repositories (Review/Coordination)

### 2. Order Service (`order-a`)
**Priority:** MEDIUM  
**Changes Required:** REVIEW ONLY

**Purpose:**
- Review `order/createdeliveryto` and `order/createdeliverytoNew` endpoints
- Ensure consistency with ASN Service changes
- Coordinate if any shared logic needs updates

**Repository Details:**
```
Repository Name: order-a (or Order Service)
Service Path: /order-a/api/v1/
Base URL Pattern: http://scmpfdev.ril.com/order-a/api/v1/
```

**Clone Command:**
```bash
git clone [ORDER_SERVICE_REPO_URL]
cd order-a
```

---

### 3. Export/Execute TO Service (`execto`)
**Priority:** LOW  
**Changes Required:** REVIEW ONLY

**Purpose:**
- Review container allocation logic if shared
- Check if any changes needed for container include/exclude operations

**Repository Details:**
```
Repository Name: execto (or Export/Execute TO Service)
Service Path: /execto/api/v1/
Base URL Pattern: http://scmpfdev.ril.com/execto/api/v1/
```

**Clone Command:**
```bash
git clone [EXECTO_SERVICE_REPO_URL]
cd execto
```

---

## Reference Repositories (No Changes Expected)

### 4. Vehicle Service (`vehicleservice`)
**Priority:** LOW  
**Changes Required:** NONE

**Purpose:**
- Reference only for vehicle type data structure
- No changes expected

**Repository Details:**
```
Repository Name: vehicleservice
Service Path: /vehicleservice/api/v1/
```

---

### 5. Core Platform Service (`coreplatformservicev2`)
**Priority:** LOW  
**Changes Required:** NONE

**Purpose:**
- Reference only for service/product data structure
- No changes expected

**Repository Details:**
```
Repository Name: coreplatformservicev2
Service Path: /coreplatformservicev2/api/v1/
```

---

## Repository Access Information

### How to Get Repository URLs

1. **Contact Backend Team Lead** for:
   - Exact repository names and URLs
   - Access permissions
   - Branch naming conventions
   - Development environment setup

2. **Check Internal Documentation** for:
   - Service registry
   - Repository mapping
   - Service ownership

3. **Common Repository Patterns:**
   - GitLab: `https://gitlab.ril.com/[group]/[repo-name]`
   - GitHub Enterprise: `https://github.ril.com/[org]/[repo-name]`
   - Azure DevOps: `https://dev.azure.com/[org]/[project]/_git/[repo-name]`

---

## Development Environment Setup

### Prerequisites
```bash
# Node.js version (check with team)
node --version  # Should match service requirements

# Database access
# - Get database connection strings
# - Get schema access permissions

# Environment variables
# - Get .env.example or configuration template
# - Set up local development environment
```

### Setup Steps
```bash
# 1. Clone primary repository
git clone [ASN_SERVICE_REPO_URL]
cd asn-a

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.example .env
# Edit .env with local configuration

# 4. Set up database
# Follow database setup instructions

# 5. Run migrations (if applicable)
npm run migrate

# 6. Start development server
npm run dev
```

---

## Branch Strategy

### Recommended Branch Naming
```
feature/transport-plan-multi-to-creation
feature/asn-search-endpoint
feature/multi-combination-to-support
```

### Branch Workflow
1. Create feature branch from `develop` or `main`
2. Implement changes
3. Create pull request
4. Code review
5. Merge to `develop`
6. Deploy to dev environment
7. QA testing
8. Merge to `main` for production

---

## Key Contacts

### Backend Team
- **ASN Service Owner:** [Name] - [Email]
- **Order Service Owner:** [Name] - [Email]
- **Backend Team Lead:** [Name] - [Email]

### Infrastructure Team
- **DevOps:** [Name] - [Email]
- **Database Team:** [Name] - [Email]

---

## Next Steps

1. ✅ Identify repositories (this document)
2. ⏳ Get repository access from backend team
3. ⏳ Clone ASN Service repository
4. ⏳ Review existing code structure
5. ⏳ Set up local development environment
6. ⏳ Start implementation

---

**Document Version:** 1.0  
**Last Updated:** [Current Date]

