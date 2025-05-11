# Requirement Specifications

This document outlines the technical and functional requirements for three key backend features of the Airbnb Clone project: User Authentication, Property Management, and Booking System.

## 1. User Authentication
### Functional Requirements
- Users can register as guests, hosts, or admins.
- Users can log in using email and password.
- Users can update their profile (e.g., name, phone number).
- Passwords must be securely hashed.

### Technical Requirements
- **API Endpoints**:
  - `POST /api/register`
    - **Input**: JSON `{ "first_name": string, "last_name": string, "email": string, "password": string, "phone_number": string, "role": "guest|host|admin" }`
    - **Output**: JSON `{ "user_id": UUID, "message": "Registration successful" }` or error `{ "error": string }`
    - **Validation**:
      - Email: Valid format, unique in User table.
      - Password: Minimum 8 characters, includes letters and numbers.
      - Role: Must be "guest", "host", or "admin".
    - **Status Codes**: 201 (Created), 400 (Bad Request), 409 (Conflict).
  - `POST /api/login`
    - **Input**: JSON `{ "email": string, "password": string }`
    - **Output**: JSON `{ "token": string, "user_id": UUID }` or error `{ "error": string }`
    - **Validation**: Email exists, password matches hash.
    - **Status Codes**: 200 (OK), 401 (Unauthorized).
  - `PUT /api/users/:user_id`
    - **Input**: JSON `{ "first_name": string, "last_name": string, "phone_number": string }`
    - **Output**: JSON `{ "message": "Profile updated" }` or error `{ "error": string }`
    - **Validation**: User_id exists, fields are valid.
    - **Status Codes**: 200 (OK), 400 (Bad Request), 404 (Not Found).
- **Performance Criteria**:
  - Response time: < 500ms for login/register.
  - Scalability: Handle 10,000 concurrent users.
  - Security: Use JWT for authentication, bcrypt for password hashing.

## 2. Property Management
### Functional Requirements
- Hosts can create, update, and delete property listings.
- Guests and Hosts can view property details.
- Properties include name, description, location, and price per night.

### Technical Requirements
- **API Endpoints**:
  - `POST /api/properties`
    - **Input**: JSON `{ "host_id": UUID, "name": string, "description": string, "location": string, "pricepernight": number }`
    - **Output**: JSON `{ "property_id": UUID, "message": "Property created" }` or error `{ "error": string }`
    - **Validation**:
      - Host_id: Valid user with role "host".
      - Pricepernight: Positive number.
      - Name, description, location: Non-empty strings.
    - **Status Codes**: 201 (Created), 400 (Bad Request), 403 (Forbidden).
  - `PUT /api/properties/:property_id`
    - **Input**: JSON `{ "name": string, "description": string, "location": string, "pricepernight": number }`
    - **Output**: JSON `{ "message": "Property updated" }` or error `{ "error": string }`
    - **Validation**: Property_id exists, host owns the property.
    - **Status Codes**: 200 (OK), 400 (Bad Request), 404 (Not Found).
  - `DELETE /api/properties/:property_id`
    - **Input**: None
    - **Output**: JSON `{ "message": "Property deleted" }` or error `{ "error": string }`
    - **Validation**: Property_id exists, host owns the property.
    - **Status Codes**: 200 (OK), 404 (Not Found).
  - `GET /api/properties/:property_id`
    - **Input**: None
    - **Output**: JSON `{ "property_id": UUID, "host_id": UUID, "name": string, "description": string, "location": string, "pricepernight": number }` or error `{ "error": string }`
    - **Status Codes**: 200 (OK), 404 (Not Found).
- **Performance Criteria**:
  - Response time: < 300ms for GET, < 500ms for POST/PUT/DELETE.
  - Scalability: Support 1,000 property listings per host.

## 3. Booking System
### Functional Requirements
- Guests can search for properties by location and dates.
- Guests can book a property for specific dates.
- System calculates total price based on price per night and stay duration.
- Guests can view booking status (pending, confirmed, canceled).

### Technical Requirements
- **API Endpoints**:
  - `GET /api/properties/search`
    - **Input**: Query params `?location=string&start_date=YYYY-MM-DD&end_date=YYYY-MM-DD`
    - **Output**: JSON `[{ "property_id": UUID, "name": string, "location": string, "pricepernight": number }]` or error `{ "error": string }`
    - **Validation**: Dates are valid, end_date > start_date.
    - **Status Codes**: 200 (OK), 400 (Bad Request).
  - `POST /api/bookings`
    - **Input**: JSON `{ "property_id": UUID, "user_id": UUID, "start_date": date, "end_date": date }`
    - **Output**: JSON `{ "booking_id": UUID, "total_price": number, "status": "pending" }` or error `{ "error": string }`
    - **Validation**:
      - Property_id exists, user_id is a guest.
      - Dates are available (no overlapping confirmed bookings).
      - Total_price calculated correctly.
    - **Status Codes**: 201 (Created), 400 (Bad Request), 409 (Conflict).
  - `GET /api/bookings/:booking_id`
    - **Input**: None
    - **Output**: JSON `{ "booking_id": UUID, "property_id": UUID, "user_id": UUID, "start_date": date, "end_date": date, "total_price": number, "status": string }` or error `{ "error": string }`
    - **Status Codes**: 200 ( CastingOK), 404 (Not Found).
- **Performance Criteria**:
  - Response time: < 500ms for search, < 700ms for booking creation.
  - Scalability: Handle 100,000 bookings per day.
  - Accuracy: Ensure no double bookings for the same property and dates.