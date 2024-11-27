# Azure AD Integration with Backend and JWT Authentication

This document outlines the integration of Azure AD for user authentication on the backend, using JWT tokens for the frontend to authenticate API requests. The document also includes project structures for the backend and details their key components.

## Backend Integration with Azure AD

### 1. Overview

The backend is responsible for authenticating users via Azure AD, generating a JWT token upon successful login, and managing user details in an MS SQL database. The backend communicates with Azure AD to retrieve user details and validates the authentication process.

### 2. Backend Project Structure

Below is the organized structure of the backend application. Each file and directory serves a specific role to maintain a modular and scalable codebase:

```
/backend
├── app/
│   ├── __init__.py
│   ├── main.py                # Application entry point
│   ├── auth/
│   │   ├── __init__.py
│   │   ├── azure_auth.py      # Azure AD integration logic
│   │   ├── jwt_handler.py     # JWT generation and validation
│   ├── db/
│   │   ├── __init__.py
│   │   ├── database.py        # Database connection setup
│   │   ├── user_crud.py       # User CRUD operations
│   ├── models/
│   │   ├── user_model.py      # User model definitions
│   ├── routes/
│   │   ├── auth_routes.py     # Authentication routes
│   │   ├── user_routes.py     # User-related routes
│   ├── schemas/
│   │   ├── user_schema.py     # Pydantic schemas for user data
│   ├── utils/
│       ├── config.py          # Configuration and environment variables
│       ├── logging.py         # Logging setup
└── tests/
    ├── test_auth.py           # Unit tests for authentication
    ├── test_user.py           # Unit tests for user routes
```

#### Configuration File

The `config.py` file is used to store application-wide configurations, such as Azure AD credentials, database settings, and JWT secret keys. Below is an example:

```python
# File: utils/config.py

import os

class Config:
    # Azure AD Configuration
    AZURE_CLIENT_ID = os.getenv('AZURE_CLIENT_ID')
    AZURE_CLIENT_SECRET = os.getenv('AZURE_CLIENT_SECRET')
    AZURE_TENANT_ID = os.getenv('AZURE_TENANT_ID')
    AZURE_REDIRECT_URI = os.getenv('AZURE_REDIRECT_URI')

    # Database Configuration
    DATABASE_URL = os.getenv('DATABASE_URL')

    # JWT Configuration
    JWT_SECRET_KEY = os.getenv('JWT_SECRET_KEY')
    JWT_ALGORITHM = "HS256"

config = Config()
```

### 3. Backend Implementation Details

#### Required Routes

- **GET /login**: Redirects the user to Azure AD for authentication.
- **GET /callback**: Processes the Azure AD authorization code, retrieves tokens, and finalizes the authentication.
- **GET /user**: Fetches user profile information for the frontend.
- **POST /logout**: Handles user logout by invalidating JWT tokens.

#### OAuth Flow Implementation Using Authlib

The entire OAuth flow is implemented using `authlib`. Below are the steps:

1. **Configure Authlib**: Setup the Azure AD endpoints for OAuth authentication.
2. **Login Endpoint**: Redirects the user to Azure AD for authentication.
3. **Callback Endpoint**: Exchanges the authorization code for tokens and retrieves user details.
4. **JWT Token Generation**: Generates a JWT token for the frontend.
5. **User Management**: Inserts or updates the user in the MS SQL database.

#### Callback Endpoint

The callback endpoint is an essential part of the Azure AD authentication flow. This endpoint handles the authorization code returned by Azure AD after successful login and uses it to fetch user details.

- **Purpose:**

  - Exchanges the authorization code for an access token.
  - Retrieves user information from Azure AD.
  - Updates or inserts user details in the database.
  - Generates a JWT token for the frontend.

- **Implementation:**
  Ensure that the callback endpoint is protected and securely processes the authorization code returned by Azure AD. Include validation checks to ensure no unauthorized access occurs.

#### File: `auth/jwt_handler.py`

Generates and validates JWT tokens:

```python
from datetime import datetime, timedelta
import jwt

SECRET_KEY = "your_secret_key"
ALGORITHM = "HS256"

# Generate a JWT token
def create_jwt_token(user_id: str):
    payload = {
        "sub": user_id,
        "exp": datetime.utcnow() + timedelta(hours=1),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
```

#### File: `db/user_crud.py`

Inserts or updates user details in the MS SQL database:

```python
from sqlalchemy.orm import Session
from models.user_model import User
from datetime import datetime

def upsert_user(session: Session, user_id: str, email: str, name: str):
    user = session.query(User).filter(User.user_id == user_id).first()
    if user:
        user.last_login = datetime.utcnow()
        user.email = email
        user.name = name
    else:
        user = User(user_id=user_id, email=email, name=name, last_login=datetime.utcnow())
        session.add(user)
    session.commit()
```

#### File: `routes/auth_routes.py`

Provides an endpoint for login and JWT token generation:

```python
from fastapi import APIRouter, Depends
from auth.azure_auth import authenticate_user
from auth.jwt_handler import create_jwt_token
from db.user_crud import upsert_user
from db.database import get_db

router = APIRouter()

@router.get("/login")
async def login():
    # Redirect user to Azure AD
    return {"message": "Redirect to Azure AD"}

@router.get("/callback")
async def callback(code: str, db: Session = Depends(get_db)):
    user_info = authenticate_user(code)  # Call Azure AD for user info
    upsert_user(db, user_info["id"], user_info["email"], user_info["name"])  # Insert/update user in DB
    token = create_jwt_token(user_info["id"])
    return {"token": token}
```

### Summary

This document provides a detailed explanation of the backend integration with Azure AD for authentication, including OAuth flow implementation, required endpoints, and user management using an MS SQL database. The configuration file example ensures secure and centralized management of sensitive data, while the project structure organizes the implementation into modular components.

### Security Considerations

- **Token Security**: Ensure that JWT tokens are signed with a strong secret key and use secure algorithms like HS256.
- **Authorization Code Validation**: Always validate the authorization code received from Azure AD to prevent unauthorized access.
- **Environment Variables**: Store sensitive configurations like client IDs, secrets, and database credentials in environment variables.
- **HTTPS**: Use HTTPS for all communication between the frontend, backend, and Azure AD to prevent data interception.
- **Token Expiry**: Implement token expiration and refresh mechanisms to reduce the risk of token misuse.

### Future Upgrades

- **Token Refresh**: Implement refresh tokens for seamless user sessions without requiring re-login.
- **Role-Based Access Control (RBAC)**: Expand user management to include roles and permissions.
- **Multi-Factor Authentication (MFA)**: Leverage Azure AD's MFA capabilities for enhanced security.
- **Logging and Monitoring**: Implement logging for authentication attempts and monitor unusual activities for proactive security.
- **Scalability**: Optimize the backend to handle a larger number of users and integrate with additional identity providers if needed.

