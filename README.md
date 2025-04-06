# Ticket Management System API

A secure REST API for an internal ticket management system where employees can submit support tickets and IT staff can manage them.

## Features

- **User Roles**: Different access levels for IT Admins and regular employees
- **Ticket System**: Employees create tickets, IT Admins resolve them
- **Comments**: Discussions on tickets
- **Authentication**: Secure user authentication with Laravel Sanctum

## Setup Instructions

### Prerequisites

- PHP 8.2 or higher
- Laravel Installer
- MySQL

### Installation

1. Create a new Laravel project using the Laravel installer:
   ```bash
   laravel new ticket-system-api
   cd ticket-system-api
   ```

2. Generate application key:
   ```bash
   php artisan key:generate
   ```

3. Configure MySQL database in `.env`:
   ```
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=ticket_system
   DB_USERNAME=your_username
   DB_PASSWORD=your_password
   ```

4. Set the api
Run the following command to initialize or set up your API scaffolding:
`php artisan install:api`

5. Remove unnecessary frontend files (API-only project):
   ```bash
   # Remove frontend-related directories
   rm -rf resources/css/
   rm -rf resources/js/
   rm -rf resources/views/
   
   # Remove frontend config files
   rm vite.config.js
   rm package.json
   rm postcss.config.js
   rm tailwind.config.js
   
   # Remove test directories if not needed
   rm -rf tests/Feature/
   rm -rf tests/Unit/
   ```

6. Create migrations, models, and controllers:
   ```bash
   php artisan make:model Ticket -mcr --policy
   php artisan make:model Comment -mcr
   ```

7. Run migrations and seed the database:
   ```bash
   php artisan migrate --seed
   ```

8. Start the development server:
   ```bash
   php artisan serve
   ```

## Database Structure

### Users Table
| Column       | Type           | Description                     |
|-------------|--------------|---------------------------------|
| id          | bigIncrements | Unique ID                       |
| name        | string       | Full name                       |
| email       | string       | Unique, validated email         |
| password    | string       | Hashed password                 |
| role        | enum         | it_admin or employee            |
| created_at  | timestamp    | Auto-generated                  |
| updated_at  | timestamp    | Auto-generated                  |

### Tickets Table
| Column       | Type           | Description                     |
|-------------|--------------|---------------------------------|
| id          | bigIncrements | Unique ID                       |
| title       | string       | Short summary (max 100 chars)   |
| description | text         | Detailed issue                  |
| status      | string       | open, in_progress, resolved     |
| priority    | string       | low, medium, high, critical     |
| user_id     | foreignId    | Who created the ticket          |
| assigned_to | foreignId    | Who is handling it              |
| created_at  | timestamp    | Auto-generated                  |
| updated_at  | timestamp    | Auto-generated                  |

### Comments Table
| Column       | Type           | Description                     |
|-------------|--------------|---------------------------------|
| id          | bigIncrements | Unique ID                       |
| message     | text         | Comment text                    |
| user_id     | foreignId    | Who wrote the comment           |
| ticket_id   | foreignId    | Linked ticket                   |
| created_at  | timestamp    | Auto-generated                  |
| updated_at  | timestamp    | Auto-generated                  |

## API Endpoints

### Authentication
| Method | Endpoint       | Description                    | Access         |
|--------|--------------|-------------------------------|---------------|
| POST   | /api/login    | Log in with email + password   | Public         |
| POST   | /api/logout   | Log out current user           | Authenticated  |

### Users
| Method | Endpoint       | Description                    | Access         |
|--------|--------------|-------------------------------|---------------|
| GET    | /api/users    | List all users                 | IT Admin only  |
| POST   | /api/users    | Create a new user              | IT Admin only  |
| GET    | /api/users/{id} | View user profile             | Own profile or IT Admin |
| PUT    | /api/users/{id} | Update user details           | Own profile or IT Admin |

### Tickets
| Method | Endpoint       | Description                    | Access         |
|--------|--------------|-------------------------------|---------------|
| GET    | /api/tickets  | List tickets (filtered by role) | Authenticated  |
| POST   | /api/tickets  | Create a new ticket            | Authenticated  |
| GET    | /api/tickets/{id} | View ticket details         | Ticket owner or IT Admin |
| PUT    | /api/tickets/{id} | Update ticket status/details | IT Admin only  |
| DELETE | /api/tickets/{id} | Delete a ticket              | IT Admin only  |

### Comments
| Method | Endpoint       | Description                    | Access         |
|--------|--------------|-------------------------------|---------------|
| GET    | /api/tickets/{id}/comments | List comments on a ticket | Ticket owner or IT Admin |
| POST   | /api/tickets/{id}/comments | Add a comment to a ticket | Authenticated  |
| DELETE | /api/comments/{id} | Delete a comment             | Comment author or IT Admin |

## API Request/Response Examples

### Login
**Request:**
```json
POST /api/login
{
  "email": "admin@example.com",
  "password": "password"
}
```

**Response:**
```json
{
  "user": {
    "id": 1,
    "name": "Admin User",
    "email": "admin@example.com",
    "role": "it_admin"
  },
  "token": "1|laravel_sanctum_token_hash"
}
```

### Create Ticket
**Request:**
```json
POST /api/tickets
Authorization: Bearer {token}

{
  "title": "Network Connection Issue",
  "description": "Unable to connect to the corporate VPN from home office",
  "priority": "high"
}
```

**Response:**
```json
{
  "data": {
    "id": 1,
    "title": "Network Connection Issue",
    "description": "Unable to connect to the corporate VPN from home office",
    "status": "open",
    "priority": "high",
    "created_at": "2025-04-06T14:30:00.000000Z",
    "user": {
      "id": 5,
      "name": "John Employee"
    },
    "assigned_to": null
  },
  "message": "Ticket created successfully"
}
```

### List Tickets (IT Admin view)
**Request:**
```
GET /api/tickets
Authorization: Bearer {admin_token}
```

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "title": "Network Connection Issue",
      "status": "open",
      "priority": "high",
      "created_at": "2025-04-06T14:30:00.000000Z",
      "user": {
        "id": 5,
        "name": "John Employee"
      },
      "assigned_to": null
    },
    {
      "id": 2,
      "title": "Software License Expired",
      "status": "in_progress",
      "priority": "medium",
      "created_at": "2025-04-05T09:15:00.000000Z",
      "user": {
        "id": 6,
        "name": "Jane Employee"
      },
      "assigned_to": {
        "id": 1,
        "name": "Admin User"
      }
    }
  ]
}
```

## Error Handling

The API returns consistent error messages:

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "title": [
      "The title field is required."
    ],
    "priority": [
      "The selected priority is invalid."
    ]
  }
}
```

## Authentication

This API uses Laravel Sanctum for token-based authentication. Include the token in all authenticated requests:

```
Authorization: Bearer {your_token}
```

## License

[MIT](https://opensource.org/licenses/MIT)