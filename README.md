# Azure AD Integration with Backend and JWT Authentication

This document outlines the integration of Azure AD for user authentication on the backend, using JWT tokens for the frontend to authenticate API requests. The document also includes project structures for both backend and frontend and details their key components.

## Backend Integration with Azure AD

### 1. Overview
The backend is responsible for authenticating users via Azure AD, generating a JWT token upon successful login, and managing user details in an MS SQL database. The backend communicates with Azure AD to retrieve user details and validates the authentication process.

### 2. Backend Project Structure
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

### 3. Backend Implementation Details

#### File: `auth/azure_auth.py`
Handles Azure AD integration logic to authenticate users and retrieve their profile details.

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

@router.post("/login")
async def login(code: str, db: Session = Depends(get_db)):
    user_info = authenticate_user(code)  # Call Azure AD for user info
    upsert_user(db, user_info["id"], user_info["email"], user_info["name"])  # Insert/update user in DB
    token = create_jwt_token(user_info["id"])
    return {"token": token}
```

---

## Frontend Project Structure and JWT Authentication

### 1. Overview
The frontend interacts with the backend for authentication and user profile management. JWT tokens issued by the backend are used to authenticate API requests.

### 2. Frontend Project Structure
```
/frontend
├── src/
│   ├── app/
│   │   ├── auth/
│   │   │   ├── auth.service.ts          # Service for handling authentication
│   │   │   ├── jwt.interceptor.ts       # Interceptor for attaching JWT to requests
│   │   ├── user-profile/
│   │       ├── user-profile.component.ts # Displays user details
│   │       ├── user-profile.component.html
│   │       ├── user-profile.component.css
│   ├── assets/                          # Static assets
│   ├── environments/                    # Environment-specific configurations
└── tests/                               # End-to-end tests
```

### 3. Frontend Implementation Details

#### File: `auth/auth.service.ts`
Handles login and fetching user data from the backend:
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class AuthService {
  private apiUrl = 'http://localhost:3000/api'; // Backend API URL

  constructor(private http: HttpClient) {}

  login(code: string): Observable<any> {
    return this.http.post(`${this.apiUrl}/login`, { code });
  }

  getUserProfile(): Observable<any> {
    return this.http.get(`${this.apiUrl}/user`);
  }
}
```

#### File: `auth/jwt.interceptor.ts`
Attaches JWT tokens to outgoing HTTP requests:
```typescript
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable()
export class JwtInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = localStorage.getItem('token');
    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`,
        },
      });
    }
    return next.handle(req);
  }
}
```

#### File: `user-profile/user-profile.component.ts`
Displays user details fetched from the backend:
```typescript
import { Component, OnInit } from '@angular/core';
import { AuthService } from '../auth/auth.service';

@Component({
  selector: 'app-user-profile',
  templateUrl: './user-profile.component.html',
  styleUrls: ['./user-profile.component.css'],
})
export class UserProfileComponent implements OnInit {
  user: any;

  constructor(private authService: AuthService) {}

  ngOnInit(): void {
    this.authService.getUserProfile().subscribe((data) => {
      this.user = data;
    });
  }
}
```

---

## Integration Flow Diagrams

### Backend Architecture
```
Frontend <---> Backend
                 |
                 +--> Azure AD (Authentication)
                 |
                 +--> MS SQL (User Database)
                 |
                 +--> JWT (Token Management)
```

### Frontend JWT Authentication Flow
1. User logs in via frontend.
2. Frontend sends a login request to the backend.
3. Backend authenticates with Azure AD, updates user details in MS SQL, and returns a JWT.
4. Frontend uses the JWT for subsequent API requests.

---

### Summary
This document provides a comprehensive guide for integrating Azure AD on the backend with JWT-based authentication for the frontend. The project structure and key components for both backend and frontend are detailed to facilitate implementation.

