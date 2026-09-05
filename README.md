# Rideshare — Flask Role-Based Ride Booking Application

A multi-role ride-booking web application built with **Flask**, **SQLAlchemy**, **Flask-Login**, **JWT-based invitations**, **Google Maps**, and **PayPal Sandbox**.

The project explores backend application architecture, authentication and authorization, ride-order workflows, external API integration, and role-specific user experiences for **customers**, **drivers**, and **administrators**.

---

## Overview

Rideshare was built as a practical full-stack Flask project around a simple VTC / ride-booking workflow.

The application separates functionality by role:

- **Users** can sign in, request rides, calculate prices, pay through PayPal Sandbox, and view ride status.
- **Drivers** can view pending rides and use dedicated driver routes to accept or decline requests.
- **Administrators** can manage users and drivers and send invitation-based registration links.

The backend uses Flask blueprints to keep role-specific routes separated and SQLAlchemy models for persistence.

---

## Main Features

### Authentication and authorization

- Session-based authentication with Flask-Login
- Password hashing with Werkzeug
- Role-based access control for `user`, `driver`, and `admin`
- Reusable `role_required()` authorization decorator
- Protected dashboards and application routes
- Ownership checks before displaying ride information
- Safe post-login redirect validation

### Invitation-based registration

Administrators can invite new users by email.

The application:

1. creates a JWT containing the invited email address and assigned role;
2. generates a registration link containing the token;
3. stores the invitation;
4. sends the link by email;
5. validates the invitation before creating the account.

This provides a controlled account-provisioning workflow rather than unrestricted public registration.

### Ride booking

Users can:

- enter departure and destination addresses;
- request a ride;
- receive a calculated price;
- store the ride request in the database;
- view order status and confirmation information.

Ride requests are represented through the `RideOrder` SQLAlchemy model.

### Distance-based pricing

The application integrates with the **Google Maps Distance Matrix API**.

The backend calculates driving distance between the departure and destination and applies a simple pricing model:

```text
price = base fare + (distance in km × per-km rate)
```

The prototype currently uses a base fare of `5.00` and a per-kilometre rate of `1.50`.

### PayPal Sandbox integration

The user payment workflow integrates with PayPal's sandbox API.

The application can:

- request an OAuth access token;
- create a PayPal payment;
- redirect the user to PayPal for approval;
- execute the payment after approval;
- display the result to the user.

This integration is intended for development and demonstration rather than production payment processing.

### Administration

Administrator routes support:

- sending invitation links;
- listing users, drivers, and administrators;
- deleting accounts;
- changing a user's role between user and driver.

### Driver workflow

Dedicated driver routes support:

- a driver dashboard;
- viewing pending ride requests;
- accepting rides;
- declining rides.

The current prototype includes the route logic for driver assignment, with some model/schema cleanup still required for the driver-assignment metadata.

---

## Technology Stack

| Technology | Purpose |
|---|---|
| Python | Backend application logic |
| Flask | Web application framework |
| Flask-SQLAlchemy | ORM and persistence |
| SQLite | Local development database |
| Flask-Login | Session authentication |
| Flask-WTF / WTForms | Form handling and validation |
| Flask-JWT-Extended | Invitation tokens |
| Werkzeug | Password hashing |
| Flask-Mail | Invitation email delivery |
| Flask-Smorest | Blueprint / route organization |
| Google Maps API | Distance calculation |
| PayPal Sandbox API | Payment workflow |
| Requests | External HTTP API calls |
| Jinja2 | Server-side templates |

---

## Application Architecture

```text
                        ┌─────────────────────┐
                        │      Flask App      │
                        └──────────┬──────────┘
                                   │
                 ┌─────────────────┼─────────────────┐
                 │                 │                 │
                 ▼                 ▼                 ▼
          User Blueprint     Driver Blueprint    Admin Blueprint
                 │                 │                 │
                 └─────────────────┼─────────────────┘
                                   │
                                   ▼
                        Authentication / RBAC
                                   │
                                   ▼
                         SQLAlchemy Models
                                   │
                     ┌─────────────┼─────────────┐
                     │             │             │
                     ▼             ▼             ▼
                   User        RideOrder       Vehicle
                                   │
                  ┌────────────────┼────────────────┐
                  │                                 │
                  ▼                                 ▼
          Google Maps API                    PayPal Sandbox
```

---

## Project Structure

```text
Rideshare/
├── admin_routes/
│   └── admin_controller.py
│
├── driver_routes/
│   └── driver_controller.py
│
├── user_routes/
│   ├── user_controller.py
│   └── user_service.py
│
├── decorators/
│   └── decorators.py
│
├── config/
│   ├── models.py
│   └── templates/
│
├── app.py
├── forms.py
├── requirements.txt
├── Project_app.ipynb
└── README.md
```

---

## Data Model

### User

Stores:

- username
- email
- hashed password
- first and last name
- role
- vehicle relationship

Supported roles:

```text
user
driver
admin
```

### RideOrder

Stores ride-request information including:

- customer name
- departure
- destination
- requested time
- status
- user relationship
- calculated price

### Vehicle

Stores basic driver vehicle information:

- make
- model
- year
- category
- driver relationship

### InvitationEmails

Stores invitation email addresses and generated invitation tokens.

---

## Security-Relevant Implementation

This project contains several security controls that were implemented directly in the application:

- passwords are stored using Werkzeug password hashes rather than plaintext;
- authenticated routes use Flask-Login;
- authorization is enforced through role checks;
- sensitive user ride records are checked against the authenticated user's ID;
- application secrets and external API credentials are loaded from environment variables;
- registration invitations use signed JWTs;
- login redirects are checked to avoid redirecting users to arbitrary external hosts.

The repository is an educational project and should not be treated as a production-hardened ride-booking or payment platform.

---

## Environment Variables

The application expects credentials and secrets through environment variables.

Example `.env` structure:

```env
SECRET_KEY=replace-with-a-random-secret
JWT_SECRET_KEY=replace-with-a-different-random-secret

MAIL_USERNAME=your-mail-account
MAIL_PASSWORD=your-mail-application-password

GOOGLE_MAPS_API_KEY=your-google-maps-key

PAYPAL_CLIENT_ID=your-paypal-sandbox-client-id
PAYPAL_CLIENT_SECRET=your-paypal-sandbox-secret
PAYPAL_API_BASE=https://api.sandbox.paypal.com
```

Do not commit real credentials to the repository.

---

## Running Locally

Clone the repository:

```bash
git clone https://github.com/Thomas170491/Rideshare.git
cd Rideshare
```

Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Create a `.env` file containing the required application credentials.

The current prototype uses a local SQLite configuration in `config/models.py`. Before running on another machine, ensure the configured database path points to a writable local location.

Start the application:

```bash
python app.py
```

The development server is then available locally through Flask.

---

## Current Project Status

This repository represents an educational full-stack application rather than a production service.

The principal application flows and integrations are present, but several areas would need additional work before production use:

- align the driver-assignment fields with the `RideOrder` database model;
- remove remaining development/debug output;
- make the SQLite database configuration fully portable;
- add automated unit and integration tests;
- add database migrations;
- improve API error handling and timeout handling;
- improve payment verification and failure handling;
- introduce stronger production configuration and deployment controls;
- expand input validation and logging.

---

## What This Project Demonstrates

The project demonstrates practical experience with:

- Python backend development
- Flask application architecture
- authentication and session management
- role-based access control
- password hashing
- JWT-based provisioning workflows
- relational data modeling with SQLAlchemy
- server-side form handling
- third-party API integration
- Google Maps distance calculations
- PayPal OAuth and payment APIs
- access-control checks on user-owned resources
- separation of application logic using Flask blueprints

---

## Portfolio Context

Rideshare is included in my portfolio as a supporting software-engineering project.

It demonstrates experience building an application where **identity, roles, authorization, user-owned data, external APIs, and payment workflows** all interact in the same system.

Those concepts complement my cybersecurity-focused work by providing practical experience with the application architectures and security controls that security teams are responsible for reviewing and protecting.