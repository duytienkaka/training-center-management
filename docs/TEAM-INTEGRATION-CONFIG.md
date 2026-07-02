# Team Integration Configuration

This document records the agreed temporary integration settings for the three backend services over Radmin VPN.

## Service URLs

| Service | Owner | Radmin IP | Port | Base URL | Swagger |
| --- | --- | --- | --- | --- | --- |
| Service 1 - Course Schedule | TODO | `26.197.54.215` | `5201` | `http://26.197.54.215:5201` | `http://26.197.54.215:5201/swagger` |
| Service 2 - Student Attendance | TODO | `26.44.91.179` | `5202` | `http://26.44.91.179:5202` | `http://26.44.91.179:5202/swagger` |
| Service 3 - Payment Report/Auth | TODO | `26.222.155.205` | `5203` | `http://26.222.155.205:5203` | `http://26.222.155.205:5203/swagger` |
| Frontend | TODO | TODO | `5173` | TODO | N/A |

## API Prefixes

| Service | API Prefix |
| --- | --- |
| Service 1 - Course Schedule | `/api/course-schedule` |
| Service 2 - Student Attendance | `/api/student-attendance` |
| Service 3 - Payment Report/Auth | `/api/payment-report` |

## Run Commands

### Service 1 - Course Schedule

```powershell
dotnet run --project TODO --urls http://0.0.0.0:5201
```

### Service 2 - Student Attendance

```powershell
dotnet run --project .\StudentAttendance.Api\StudentAttendance.Api.csproj --urls http://0.0.0.0:5202
```

### Service 3 - Payment Report/Auth

```powershell
dotnet run --project TODO --urls http://0.0.0.0:5203
```

### Frontend

```powershell
npm run dev -- --host 0.0.0.0 --port 5173
```

## Service 2 External Configuration

```json
{
  "ExternalServices": {
    "CourseScheduleBaseUrl": "http://26.197.54.215:5201",
    "PaymentReportBaseUrl": "http://26.222.155.205:5203"
  }
}
```

## Authentication Agreement

Payment Report/Auth Service is responsible for login, user accounts, refresh tokens, and JWT issuing.

Service 1 and Service 2 only validate JWTs. They must use the same JWT settings as Service 3.

| Item | Agreed Value |
| --- | --- |
| JWT issuer | TODO |
| JWT audience | TODO |
| JWT secret/key | TODO |
| User id claim | `sub` and/or `userId` |
| Role claim | `role` |
| Admin role | `Admin` |
| Teacher role | `Teacher` |
| Student role | `Student` |
| Internal service role | `InternalService` |

Internal service-to-service requests should use a token with `role=InternalService`, unless the team agrees on a gateway or a different internal-auth mechanism.

## Firewall Checklist

| Machine | Required Port |
| --- | --- |
| Service 1 machine | `5201` |
| Service 2 machine | `5202` |
| Service 3 machine | `5203` |
| Frontend machine | `5173` |

## Quick Connectivity Tests

```powershell
Invoke-WebRequest http://26.197.54.215:5201/swagger
Invoke-WebRequest http://26.44.91.179:5202/swagger
Invoke-WebRequest http://26.222.155.205:5203/swagger
```

## Items To Confirm

- Owner of each service
- Final port for Service 1 and Service 3, if different from `5201` and `5203`
- Frontend Radmin IP and final port
- JWT issuer, audience, and secret from Service 3
- Internal-service token strategy
- Exact request/response contracts for APIs called between services
