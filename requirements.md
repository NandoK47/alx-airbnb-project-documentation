# Backend Requirements Specification  
**Project:** ALX Airbnb Clone  
**Document:** requirements.md  
**Location:** Root directory of `alx-airbnb-project-documentation` repository  

---

## 1. User Authentication & Authorization

### Objective  
Provide secure registration, login, and role-based access to the platform.  

### Functional Requirements  
- Users can **register** with email, password, and full name.  
- Users can **log in** with email and password.  
- Passwords must be securely hashed before storage.  
- JWT (JSON Web Token) will be used for authentication.  
- Sessions expire after a set duration (e.g., 1 hour).  
- Unauthorized requests must return a `401 Unauthorized`.  

### API Endpoints  
- **POST /api/v1/auth/register**  
  - Input: `{ "email": "string", "password": "string", "full_name": "string" }`  
  - Output: `{ "id": "uuid", "email": "string", "token": "jwt_token" }`  
  - Validation:  
    - Email must be unique and valid format.  
    - Password must be ≥ 8 characters, contain at least one number & special character.  
    - Full name required.  

- **POST /api/v1/auth/login**  
  - Input: `{ "email": "string", "password": "string" }`  
  - Output: `{ "token": "jwt_token", "expires_in": 3600 }`  
  - Validation:  
    - Email must exist.  
    - Password must match hash.  

- **GET /api/v1/auth/profile** (Protected)  
  - Input: JWT token in header.  
  - Output: `{ "id": "uuid", "email": "string", "full_name": "string", "role": "user|admin" }`  

### Performance Criteria  
- Authentication requests should respond within **<200ms** under normal load.  
- Must support at least **1000 concurrent active sessions**.  

---

## 2. Property Management

### Objective  
Allow property owners to add, update, delete, and view property listings.  

### Functional Requirements  
- Owners can create property listings with details (title, description, price, location, images).  
- Only owners can update or delete their properties.  
- All users can view properties.  
- Properties should support filtering and searching by location, price, and availability.  

### API Endpoints  
- **POST /api/v1/properties** (Protected - Owner)  
  - Input:  
    ```json
    {
      "title": "string",
      "description": "string",
      "price_per_night": "decimal",
      "location": "string",
      "images": ["string"]
    }
    ```  
  - Output:  
    `{ "id": "uuid", "title": "string", "price_per_night": "decimal", "owner_id": "uuid" }`  
  - Validation:  
    - Title: min 5 characters.  
    - Price: must be > 0.  
    - Location: required.  
    - Images: optional but if provided must be valid URLs.  

- **GET /api/v1/properties**  
  - Input: optional query params: `?location=string&min_price=number&max_price=number`  
  - Output: Array of property objects.  

- **PUT /api/v1/properties/{id}** (Protected - Owner)  
  - Input: Partial or full property object.  
  - Output: Updated property object.  

- **DELETE /api/v1/properties/{id}** (Protected - Owner)  
  - Output: `{ "message": "Property deleted successfully" }`  

### Performance Criteria  
- Listing query must return results within **<500ms** with up to **10,000 properties**.  
- Support pagination (default: 10 per page).  

---

## 3. Booking System

### Objective  
Allow users to book available properties for specific dates.  

### Functional Requirements  
- Users can create bookings by selecting property and date range.  
- System must check for property availability (no double booking).  
- Owners can view bookings for their properties.  
- Users can cancel their bookings within allowed cancellation window (configurable).  

### API Endpoints  
- **POST /api/v1/bookings** (Protected - User)  
  - Input:  
    ```json
    {
      "property_id": "uuid",
      "check_in": "YYYY-MM-DD",
      "check_out": "YYYY-MM-DD"
    }
    ```  
  - Output:  
    `{ "id": "uuid", "property_id": "uuid", "user_id": "uuid", "status": "confirmed" }`  
  - Validation:  
    - Dates must be valid & `check_out` > `check_in`.  
    - Property must exist and be available.  

- **GET /api/v1/bookings/{id}** (Protected - User/Owner)  
  - Output: Booking details.  

- **DELETE /api/v1/bookings/{id}** (Protected - User/Owner)  
  - Output: `{ "message": "Booking cancelled successfully" }`  

### Performance Criteria  
- Booking confirmation must complete within **<300ms** under normal load.  
- System should prevent race conditions (use transactions/locking).  
- Must scale to handle at least **500 bookings per second** during peak load.  

---

## Notes  
- All endpoints return errors in format:  
  ```json
  {
    "error": true,
    "message": "Error description"
  }
