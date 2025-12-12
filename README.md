# RustyRMM API Platform

A multi-tenant Remote Monitoring and Management (RMM) API platform built with Node.js and Express.

## Features

- **Multi-tenant Architecture**: Support for multiple tenants with branch management
- **Agent Management**: Register and manage agents across Windows, macOS, and Linux
- **Task System**: Create and manage tasks (reboot, restart agent, etc.) with scheduling support
- **Real-time Metrics**: Track agent uptime, RAM, CPU, and disk usage
- **Role-based Access**: Superuser, admin, and user roles with tenant-based access control
- **JWT Authentication**: Secure bearer token authentication with 24-hour expiration
- **API Documentation**: Complete Swagger/OpenAPI documentation

## Prerequisites

- Docker
- Docker Compose (v2+)

## Installation

### 1. Clone the repository
```bash
git clone <repo-url>
cd rustyrmm
```

### 2. Configure Environment Variables
Copy the example environment file and edit it:

```bash
cp docker-compose.env.example .env
nano .env
```

At a minimum, configure:
- `JWT_SECRET`
- `SUPER_ADMIN_PASSWORD`
- Database credentials (see below)

---

## Database Configuration (Important)

RustyRMM supports **two database modes** using Docker Compose profiles.

### Option A: Use Docker-managed MySQL (recommended for local/dev)

This will spin up a MySQL 8 container automatically.

```bash
docker compose --profile localdb up -d
```

In this mode:
- MySQL runs as a Docker container
- Data is stored in a Docker volume
- `DB_HOST` defaults to `db`
- Schema and initial data are automatically applied

No external database is required.

---

### Option B: Use an Existing MySQL Instance (recommended for production)

If you already have MySQL running (bare metal, VM, RDS, etc.), **do not enable the `localdb` profile**.

```bash
docker compose up -d
```

Update `.env` to point to your existing database:

```env
DB_HOST=your-mysql-hostname-or-ip
DB_PORT=3306
DB_USER=rustyrmm
DB_PASSWORD=strongpassword
DB_NAME=rustyrmm
```

Notes:
- Ensure the database already exists
- The configured user must have full schema permissions
- Database initialization and migrations are handled automatically by the API container

### MySQL on the Docker Host
If MySQL is running on the same machine as Docker:

- **macOS / Windows**
```env
DB_HOST=host.docker.internal
```

- **Linux**
```env
DB_HOST=172.17.0.1
```

---

## Service Access

- **API**: http://localhost:3000
- **Swagger Docs**: http://localhost:3000/api/api-docs

## Superuser Access

Superusers can:
- View all tenants and agents
- Switch between tenant contexts using `POST /tenants/switch`
- Access all resources regardless of tenant association

Regular users are restricted to their assigned tenants.

---

## Agent Registration Flow

1. **Create a Branch**
   - Generates a 32-character alphanumeric registration code

2. **Agent Registration**
   - Endpoint: `POST /agent/register`
   - Payload:
     - `hostname`
     - `OS` (e.g. `win 10 x64`)
     - `arch` (`win`, `osx`, `lin`)
     - `localIdentifier`
     - `registrationToken`

3. **Agent Check-in**
   - Endpoint: `POST /agent/checkin`
   - Payload:
     - `agent_id`
     - `local_id`
     - `uptime`, `ram`, `cpu`, `disk`

4. **Task Execution**
   - Pending tasks are returned during check-in
   - Results submitted to `POST /tasks/:task_id`

---

## Task Management

Tasks may be:
- **Immediate**: Executed on next agent check-in
- **Scheduled**: Executed after a specified time

Supported task types include:
- `reboot`
- `restart_agent`
- *(extensible)*

---

## Default Super Admin Credentials

On first startup, a super admin account is created:

- **Username**: `admin`
- **Email**: `admin@rustyrmm.com`
- **Password**: `N0R3@llyChangeMe123!`

All values can be overridden via `.env`.

**⚠️ IMPORTANT:** Change this password immediately after first login.

---

## Security Considerations

- Rotate `JWT_SECRET` regularly
- Use strong database passwords
- Enable HTTPS in production
- Add rate limiting and WAF protections
- Keep Docker images and dependencies up to date

---

## License

ISC

## Support

For issues or questions:
- Review the project documentation
- Open an issue in the repository
