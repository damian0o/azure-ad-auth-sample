# Frontend Project Structure and JWT Authentication

## 1. Overview
The frontend interacts with the backend for authentication and user profile management. JWT tokens issued by the backend are used to authenticate API requests.

## 2. Frontend Project Structure
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

## 3. Frontend Implementation Details

### File: `auth/auth.service.ts`
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

### File: `auth/jwt.interceptor.ts`
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

### File: `user-profile/user-profile.component.ts`
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

## Integration Flow Diagrams

### Frontend JWT Authentication Flow
1. User logs in via frontend.
2. Frontend sends a login request to the backend.
3. Backend authenticates with Azure AD, updates user details in MS SQL, and returns a JWT.
4. Frontend uses the JWT for subsequent API requests.

## Summary
This document provides a detailed guide for implementing JWT-based authentication on the frontend, including project structure, essential files, and their responsibilities.

