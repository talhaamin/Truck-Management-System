# Truck Management System

Truck Management System is an ASP.NET Core 8 MVC web application for fleet, driver, inventory, fuel, trip, maintenance, tyre, audit, RBAC, and reporting operations. It is built against the existing SQL Server database `TruckManagementDB` and follows a modular INTEGRA-style enterprise layout.

## Technology Stack

- ASP.NET Core 8 MVC
- Entity Framework Core 8
- SQL Server
- Cookie authentication
- Claims-based RBAC
- Serilog rolling file logs
- AdminLTE, Bootstrap 5, Bootstrap Icons
- jQuery, DataTables, Select2-style searchable dropdown behavior, SweetAlert2, Toastr, Chart.js
- xUnit tests

## Solution Structure

```text
TruckManagementSystem.sln
src/
  TMS.Domain/          Entities and domain contracts
  TMS.Application/     Interfaces, result models, service contracts
  TMS.Infrastructure/  EF Core DbContext, repositories, unit of work, auth, seeding
  TMS.Web/             MVC controllers, views, UI assets, middleware, helpers
tests/
  TMS.Tests/           Basic service and infrastructure tests
schema/                Database, lookup, audit, seed, report, and permission scripts
docs/                  Operator and setup documentation
screens/               Application screenshots for documentation
```

## Runtime behavior:

- Auth cookie name: `TMS.Auth`
- Login path: `/Account/Login`
- Access denied path: `/Account/AccessDenied`
- Cookie expiry: 10 hours with sliding expiration
- Data protection keys: `src/TMS.Web/DataProtectionKeys`
- Serilog files: `src/TMS.Web/logs/tms-YYYYMMDD.log`
- Driver document upload folder: `src/TMS.Web/wwwroot/uploads/driver-documents`

## Screenshots

 ![Truck Management System screenshot 1](./screens/1.PNG) 
 ![Truck Management System screenshot 2](./screens/2.PNG) 
 ![Truck Management System screenshot 2a](./screens/2a.PNG) 
 ![Truck Management System screenshot 3](./screens/3.PNG) 
 ![Truck Management System screenshot 3a](./screens/3a.PNG) 
 ![Truck Management System screenshot 3b](./screens/3b.PNG) 
 ![Truck Management System screenshot 4](./screens/4.PNG) 
 ![Truck Management System screenshot 5](./screens/5.PNG) 
 ![Truck Management System screenshot 6](./screens/6.PNG) 
 ![Truck Management System screenshot 7](./screens/7.PNG) 
 ![Truck Management System screenshot 8](./screens/8.PNG) 


## Modules

### Dashboard

- Active trucks and active drivers
- Total fuel and total KM
- Maintenance due and overdue counts
- License expiry alerts
- Low stock count
- Tyre installed, in-stock, and scrapped counts
- Current assignments
- Unassigned active trucks
- Drivers missing document uploads
- Trips and fuel for the current month
- Inventory in-hand grid with search
- Fuel and KM charts by truck
- Recent fuel and trip tables

### Fleet Management

- Trucks
  - Add, edit, view, activate/deactivate
  - Full truck profile
  - Profile tabs for fuel, trips, maintenance, tyres, and stock
  - Grid View and Detailed View in profile tabs
  - Tyre history modal by truck position
  - Create truck with AJAX tyre installation workflow

- Drivers
  - Add, edit, view, activate/deactivate
  - License expiry tracking
  - Aadhaar field
  - Profile with assignment, fuel/trip, and document history
  - License front/back and Aadhaar front/back upload

- Assignments
  - Assign driver to truck
  - Close previous current assignment
  - Prevent duplicate active driver/truck assignments

### Operations

- Fuel Entries
  - Truck, driver, fuel type, quantity, rate, station, odometer
  - Odometer validation
  - Truck odometer update

- Trip Logs
  - Route, old KM, new KM, total KM
  - Odometer validation
  - Truck odometer update

- Maintenance Logs
  - Maintenance type, workshop, cost, odometer, next due date/KM
  - Inventory consumption rows for non-tyre maintenance
  - Tyre Replacement workflow with tyre installation grid
  - Odometer validation
  - Maintenance due alerts

### Inventory

- Inventory Items
  - Item master, type, unit, reorder level
  - In-hand quantity shown in grid

- Stock Transactions
  - Opening
  - Purchase
  - Issue to truck
  - Return
  - Adjustment
  - Negative stock prevention
  - `STOCK_OVERRIDE` permission for exceptional negative stock with required remarks

### Tyre Management

- Tyre master
- Tyre installation
- Tyre removal
- Tyre scrap workflow
- Position validation
- Installed and removed KM tracking
- Complete truck-position tyre history

### Reports

- Truck Fuel Summary
- Truck KM Summary
- Stock Balance
- Fuel Consumption
- Trip Report
- Maintenance Report
- Driver Assignment Report
- Tyre Usage Report
- Low Stock Report
- License Expiry Report
- Maintenance Due Report
- Export to Excel/PDF/print through DataTables buttons

### Administration

- Users
- Roles
- Permissions
- Assign roles to users
- Role permission matrix
- Login history
- Audit logs with table/action/user/date/search filters
- Permission-based menus and buttons

### Setup Lookups

- Truck companies
- Truck models
- Inventory item types
- Units
- Stock transaction types
- Fuel types
- Fuel stations
- Locations
- Workshops
- Tyre brands
- Tyre sizes
- Tyre statuses
- Tyre positions

## Security and Business Rules

- Cookie authentication
- Claims-based permissions
- `[HasPermission]` authorization filter
- Permission-based sidebar rendering
- Anti-forgery tokens on forms
- Server-side validation
- Required field styling on data entry screens
- Regex validation for email, mobile, truck number, registration, username, Aadhaar, and license-like fields
- Password hashing with PBKDF2
- Login history recording
- Serilog logging
- Audit writer for create/edit/delete/upload-style events when audit migration is installed

Important controlled overrides:

- `STOCK_OVERRIDE`
  - Allows approved negative stock issue
  - Requires a meaningful reason in remarks
  - Assigned to Super Admin by default

- `ODOMETER_OVERRIDE`
  - Allows exceptional entries below latest known odometer
  - Assigned to Super Admin by default

## Logs and Troubleshooting

Application logs are written to:

```text
src/TMS.Web/logs/tms-YYYYMMDD.log
```

Common checks:

- If login fails, confirm `AppUsers`, `Roles`, `UserRoles`, `Permissions`, and `RolePermissions` are seeded.
- If lookup dropdowns are empty, run `schema/TMS-Lookup-Tables.sql`.
- If driver document upload fails, run `schema/TMS-Driver-Documents.sql` and verify write permission on `wwwroot/uploads/driver-documents`.
- If Audit Logs are empty, run `schema/TMS-Audit-Migration.sql` and perform a create/edit/delete action after login.
- If report procedure mode is required, run `schema/TMS-Report-Procedures.sql`.

## Testing

```powershell
dotnet test tests/TMS.Tests/TMS.Tests.csproj
```

Current test coverage is basic and focused on infrastructure/password services. Broader enterprise service tests can be added as the next hardening phase.

## Deployment

### IIS Deployment

1. Install .NET 8 Hosting Bundle on the server.
2. Create or restore `TruckManagementDB`.
3. Run database scripts in the sequence listed above.
4. Publish the app:

   ```powershell
   dotnet publish src/TMS.Web/TMS.Web.csproj -c Release -o .\publish\TMS.Web
   ```

5. Create an IIS site pointing to the publish folder.
6. Configure the app pool:
   - No Managed Code
   - Integrated pipeline
7. Set file permissions for:
   - `logs`
   - `DataProtectionKeys`
   - `wwwroot/uploads/driver-documents`
8. Set production connection string through `appsettings.Production.json`, environment variable, or IIS configuration.
9. Enable HTTPS.
10. Restart the site and sign in as `superadmin`.

### Production Checklist

- Change the default Super Admin password immediately.
- Use a least-privilege SQL login.
- Enable HTTPS only.
- Back up SQL Server regularly.
- Back up uploaded driver documents.
- Protect `DataProtectionKeys`.
- Configure log retention.
- Restrict `STOCK_OVERRIDE` and `ODOMETER_OVERRIDE`.
- Verify audit log visibility.
- Verify report exports.
- Verify document upload and download.

## Notes

- The application is built against the existing database and avoids forcing schema redesign.
- Optional scripts add audit, lookups, driver document metadata, report procedures, and demo data.
- SaaS tenant/site structure is intentionally not enabled because this installation is single-organization.
