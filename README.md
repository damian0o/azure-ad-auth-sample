# Integration with Azure AD for Backend Authentication

## Overview
This document outlines the steps to integrate Azure Active Directory (Azure AD) with the backend system for user authentication. After successful authentication, the backend will issue a JSON Web Token (JWT) to the frontend for subsequent authenticated requests. Additionally, user details will be inserted or updated in an MS SQL table.

## Objectives
1. Authenticate users via Azure AD.
2. Issue a JWT token to the frontend upon successful login.
3. Use the JWT token to authorize requests to the backend.
4. Insert or update user details in an MS SQL database.
5. Ensure secure communication and token management.

---

## Steps for Integration

### 1. **Set Up Azure AD Application**
1. Log in to the [Azure Portal](https://portal.azure.com/).
2. Navigate to **Azure Active Directory** > **App registrations** > **New registration**.
   - **Name:** `YourAppName`
   - **Supported account types:** Choose the appropriate option for your use case (e.g., "Accounts in this organizational directory only").
   - **Redirect URI:** Add the backend callback URL (e.g., `https://your-backend-url/api/auth/callback`).
3. Click **Register**.
4. Note down the following values:
   - **Application (client) ID**
   - **Directory (tenant) ID**
5. Navigate to **Certificates & secrets** and create a new client secret.
   - Note down the secret value.

### 2. **Backend Configuration**

#### a. Configuration File
Create a configuration file to store Azure AD details:

- **CLIENT_ID:** Application (client) ID from Azure AD.
- **CLIENT_SECRET:** The secret generated in the Certificates & secrets section.
- **TENANT_ID:** Directory (tenant) ID.
- **AUTHORITY:** Authority URL for Azure AD.
- **REDIRECT_URI:** Backend callback URL for Azure AD authentication.
- **JWT_SECRET:** Secret key used for signing JWT tokens.
- **JWT_ALGORITHM:** Algorithm for JWT signing (e.g., HS256).
- **DATABASE_URL:** Connection string for MS SQL database.

#### b. Authentication Flow
1. Implement the Azure AD authentication flow to handle OAuth 2.0 and OpenID Connect.
2. Define endpoints for login and callback to interact with Azure AD.
3. Extract user details (e.g., email, name) from the ID token provided by Azure AD.

### 3. **User Details Management**
1. Create a table in MS SQL to store user information:
   ```sql
   CREATE TABLE Users (
       UserID NVARCHAR(255) PRIMARY KEY,
       Email NVARCHAR(255) NOT NULL,
       Name NVARCHAR(255),
       LastLogin DATETIME NOT NULL
   );
   ```
2. Connect to the database and manage the `Users` table.
3. Implement a function to insert or update user details in the database during authentication.

### 4. **Frontend Integration**
1. After logging in, the backend returns a JWT token.
2. Store the token securely on the frontend (e.g., HTTP-only cookies or secure storage).
3. Use the token to authenticate subsequent API requests by including it in the `Authorization` header:
   ```http
   Authorization: Bearer <your-jwt-token>
   ```

### 5. **Secure the Endpoints**
Add JWT-based authorization to secure specific endpoints in your backend:
1. Decode and validate the JWT token in the request header.
2. Return an error if the token is invalid or expired.
3. Allow access to protected routes only for authenticated users.

### 6. **Token Expiry and Refresh**
- Azure AD issues tokens with an expiry time (e.g., 1 hour).
- Use refresh tokens from Azure AD to acquire new tokens without requiring the user to log in again.

### 7. **Testing**
1. Test the login flow by visiting `/auth/login`.
2. Verify that a JWT is returned after authentication.
3. Use the JWT to access protected routes.

---

## Security Considerations
1. **Store Secrets Securely:** Use environment variables or secure vaults to store Azure AD credentials and JWT signing secrets.
2. **HTTPS:** Ensure all communication between the frontend, backend, and Azure AD is encrypted.
3. **Token Expiry:** Regularly refresh tokens and handle token expiration gracefully.
4. **Scopes:** Limit the scopes requested from Azure AD to the minimum necessary for your application.

---

## Summary
This integration enables secure user authentication via Azure AD and leverages JWT tokens for authorizing requests between the frontend and backend. User details are inserted or updated in an MS SQL database to maintain a record of authenticated users. Ensure robust error handling and security practices to protect user data and application integrity.
