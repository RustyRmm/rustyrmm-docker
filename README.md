# RustyRMM API Platform

A multi-tenant Remote Monitoring and Management (RMM) API platform built with Node.js and Express.

## Features

- **Multi-tenant Architecture**: Support for multiple tenants with branch management
- **Agent Management**: Register and manage agents across Windows, Mac, and Linux
- **Task System**: Create and manage tasks (reboot, restart agent, etc.) with scheduling support
- **Real-time Metrics**: Track agent uptime, RAM, CPU, and disk usage
- **Role-based Access**: Superuser, admin, and user roles with tenant-based access control
- **JWT Authentication**: Secure bearer token authentication with 24-hour expiration
- **API Documentation**: Complete Swagger/OpenAPI documentation

## Prerequisites

- Docker / Compose

## Installation

1. **Clone the repository** (or navigate to the project directory)

2. **Configure Environment Variables**
```bash
cp docker-compose.env.example .env
nano .env
```
3. **Spinup your Environment**
```bash
docker compose up -d
```

The Service will be available at `http://localhost:3000`
Swagger documentation will be available at `http://localhost:3000/api/api-docs`

### Superuser Access

Superusers can:
- View all tenants and agents
- Switch between tenant contexts using `POST /tenants/switch`
- Access all resources regardless of tenant association

Regular users are associated with specific tenants and can only access resources within their assigned tenants.

## Agent Registration Flow

1. **Create a Branch**: A tenant owner creates a branch, which generates a 32-character alphanumeric registration code
2. **Agent Registration**: The agent calls `POST /agent/register` with:
   - `hostname`: Agent hostname
   - `OS`: Operating system string (e.g., "win 10 x64")
   - `arch`: Architecture (`win`, `osx`, or `lin`)
   - `localIdentifier`: Local UUID/GUID
   - `registrationToken`: The 32-character registration code
3. **Agent Check-in**: Agents periodically call `POST /agent/checkin` with:
   - `agent_id`: Server-assigned UUID
   - `local_id`: Local identifier
   - `uptime`, `ram`, `cpu`, `disk`: Current metrics
4. **Task Execution**: If a pending task exists, it's returned in the check-in response. The agent executes it and submits results via `POST /tasks/:task_id`

## Task Management

Tasks can be created as:
- **Immediate**: Available for pickup on next agent check-in
- **Scheduled**: Available only after the scheduled time

Supported task types:
- `reboot`: Reboot the agent system
- `restart_agent`: Restart the agent service
- etc..

## Default Super Admin Credentials

After running `npm run init-db`, the default super admin credentials are:

- **Username**: `admin` (or as set in `.env`)
- **Email**: `admin@rustyrmm.com` (or as set in `.env`)
- **Password**: `N0R3@llyChangeMe123!` (or as set in `.env`)

**⚠️ IMPORTANT**: Change the default password immediately after first login!

## Security Considerations

- Change default JWT secret in production
- Use strong passwords for database and super admin
- Implement rate limiting for production
- Use HTTPS in production
- Regularly rotate API tokens
- Keep dependencies up to date

## License

ISC

## Support

For issues and questions, please refer to the project documentation or create an issue in the repository.

