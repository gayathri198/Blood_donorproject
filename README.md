# Redline Blood Bank and Donor Management

A student-project implementation of the Blood Bank and Donor Management assignment using the MEAN stack: MongoDB, Express, Angular, and Node.js.

## Assignment Requirements Covered

- Register and browse donors with ABO/Rh blood group, contact details, status, and last donation date.
- Record donations and generate traceable blood-component units in inventory.
- Register hospitals and capture blood requests with component, quantity, priority, and status.
- Search inventory by recipient blood group, component, quantity, and availability status.
- Allocate compatible units to hospital requests without double-booking units.
- Monitor expiry dates and mark expired available units unavailable.
- Manage the six main collections: `Donors`, `Donations`, `BloodUnits`, `Hospitals`, `Requests`, and `Allocations`.

## Project Solution

The Angular operations dashboard presents donor, donation, inventory, hospital, and request views. Forms submit to an Express REST API. Mongoose validates and persists data in MongoDB. A completed donation creates red-cell, plasma, and platelet component records with separate shelf lives. Inventory search checks status and expiry, selects the earliest-expiring matching units, and reports both the number available and whether the requested quantity can be met.

For red-cell requests, compatibility is evaluated from the patient/recipient group to eligible donor groups (for example, an A+ recipient can receive A+, A-, O+, or O- red cells). Other components use an exact blood-group match in this instructional version. Allocation first claims a pending request, then atomically changes each selected unit from `available` to `allocated`. If there are too few units or a write fails, reserved units are released and the request returns to pending. This prevents two concurrent allocations from booking the same unit.

Expiry is checked whenever availability/inventory is loaded and through the dashboard's **Run expiry check** action. The implementation uses configurable component shelf-life rules in the API; validate all clinical rules with a qualified transfusion service before real use.

## Data Model

- `Donor`: donor identity, contact details, blood group, status, and last donation date.
- `Donation`: references one donor; records date, volume, and a donor/group snapshot.
- `BloodUnit`: references its source donation and donor; records unit code, blood group, component, volume, expiry, and availability status.
- `Hospital`: hospital identity, location, status, and request contact.
- `Request`: references a hospital; records the requested recipient group, component, quantity, priority, and fulfillment status.
- `Allocation`: references one request and hospital plus the set of reserved blood units.

## Requirements

- Node.js 22.12+ or 24.0+ and npm.
- MongoDB Community Server running locally, or a MongoDB Atlas connection string.
- A modern browser. Angular development server uses port 4200; the API uses port 3000.

## Run Locally

1. Install dependencies from the project root:

   ```powershell
   npm install
   npm install --prefix server
   npm install --prefix client
   ```

2. Configure the API. Copy `server/.env.example` to `server/.env`, then set `MONGODB_URI` to your local MongoDB URI or Atlas connection string. The example defaults to `mongodb://127.0.0.1:27017/redline_blood_bank`.

3. Start MongoDB, then load demonstration donors, hospitals, units, and requests:

   ```powershell
   npm run seed
   ```

   **Warning:** the seed script clears the six application collections before inserting its sample records. Do not run it against data you need to keep.

4. Start the API and Angular app together:

   ```powershell
   npm run dev
   ```

5. Open `http://localhost:4200`. The API health endpoint is `http://localhost:3000/api/health`.

## Useful Commands

- `npm test` runs the compatibility and availability logic tests.
- `npm run build` produces the Angular production build.
- `npm run seed` reloads the sample dataset (destructive to existing app records).
- `npm run dev --prefix server` starts only the API.
- `npm start --prefix client` starts only the Angular app.

## REST Endpoints

- `GET/POST /api/donors`
- `GET/POST /api/donations`
- `GET /api/units`, `GET /api/units/search`, `POST /api/units/expire`
- `GET/POST /api/hospitals`
- `GET/POST /api/requests`, `POST /api/requests/:id/allocate`
- `GET /api/allocations`, `GET /api/summary`, `GET /api/health`

Availability search accepts `bloodGroup`, `component`, `quantity`, and `status` query parameters, for example:

```text
/api/units/search?bloodGroup=A%2B&component=Red%20Cells&quantity=2&status=available
```

The response contains `units` (up to the requested quantity), `availableCount`, `requestedQuantity`, and `sufficient`.

## Production Considerations

This is an educational starting point, not a clinical or production-ready blood-bank system. Authentication, role-based access, audit trails, privacy controls, rate limiting, deployment configuration, and hardened security are not included. Add these before handling real donor or patient data, and have inventory compatibility, eligibility, storage, and expiry rules reviewed by qualified professionals.