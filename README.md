# Reminder Service

> A lightweight Node.js microservice for creating and processing email reminder tickets for the Airline Management system.

This service accepts reminder requests, stores them in MySQL with Sequelize, and integrates with RabbitMQ so reminder-related events can be consumed asynchronously. It is designed as a small backend building block that can sit behind a booking, notification, or scheduler workflow.

## Highlights

- Create reminder tickets through a simple REST endpoint
- Persist notification jobs with Sequelize and MySQL
- Consume reminder events from RabbitMQ
- Send emails with Nodemailer
- Keep the codebase organized with controller, service, repository, and utility layers

## Tech Stack

- Node.js
- Express
- Sequelize ORM
- MySQL
- RabbitMQ
- Nodemailer
- node-cron

## How It Works

1. A client or another service sends reminder data to the API or through a broker event.
2. The service stores that reminder as a `NotificationTicket`.
3. Pending reminders can later be processed and sent as emails.
4. Ticket status is updated as the reminder moves through the flow.

## API

### Create Reminder Ticket

`POST /api/v1/tickets`

Sample request body:

```json
{
  "subject": "Flight check-in reminder",
  "content": "Your flight check-in opens in 2 hours.",
  "recepientEmail": "traveler@example.com",
  "status": "PENDING",
  "notificationTime": "2026-05-12T18:30:00.000Z"
}
```

Sample success response:

```json
{
  "success": true,
  "data": {
    "id": 1,
    "subject": "Flight check-in reminder",
    "content": "Your flight check-in opens in 2 hours.",
    "recepientEmail": "traveler@example.com",
    "status": "PENDING",
    "notificationTime": "2026-05-12T18:30:00.000Z"
  },
  "err": {},
  "message": "Successfully registered an email reminder"
}
```

## Environment Variables

Create a `.env` file in the project root:

```env
PORT=3005
EMAIL_ID=your-gmail-address
EMAIL_PASS=your-app-password
MESSAGE_BROKER_URL=amqp://localhost
EXCHANGE_NAME=reminder_exchange
REMINDER_BINDING_KEY=reminder_key
```

Notes:

- `EMAIL_PASS` should be an app password if you use Gmail.
- The current source loads environment variables with `dotenv.config()`, so the default expected location is the project root `.env`.

## Database Setup

The project already contains Sequelize models and migrations inside `src/`.

Update [`src/config/config.json`](/c:/myLearnings/Wev-devlopment/X-BackEnd_Dev/Y-Projects/AirLine%20Managment/Reminder-Service/src/config/config.json) with your MySQL credentials and database names, then run:

```powershell
npx sequelize db:migrate --config src/config/config.json --migrations-path src/migrations --models-path src/models
```

## Run Locally

Install dependencies:

```powershell
npm install
```

Start the service:

```powershell
npm start
```

By default, the server runs on the port defined in `.env`.

## Message Queue Integration

The service connects to RabbitMQ on startup and subscribes to the configured binding key.

Current event handling supports:

- `CREATE_TICKET`
- `SEND_BASIC_MAIL`

These are processed through the email service layer in [`src/services/email-service.js`](/c:/myLearnings/Wev-devlopment/X-BackEnd_Dev/Y-Projects/AirLine%20Managment/Reminder-Service/src/services/email-service.js).

## Project Structure

```text
src/
|-- config/        # environment, mail, and sequelize config
|-- controllers/   # request handlers
|-- migrations/    # sequelize migrations
|-- models/        # sequelize models
|-- repository/    # database access layer
|-- services/      # business logic
|-- utils/         # queue and job helpers
`-- index.js       # app entry point
```

## Current Model

The primary Sequelize model is `NotificationTicket` with these fields:

- `subject`
- `content`
- `recepientEmail`
- `status`
- `notificationTime`

## Development Notes

- The current API/model uses the field name `recepientEmail`. If you want the corrected spelling `recipientEmail`, update both the model and migration or create a rename migration.
- A cron-based job utility exists in [`src/utils/job.js`](/c:/myLearnings/Wev-devlopment/X-BackEnd_Dev/Y-Projects/AirLine%20Managment/Reminder-Service/src/utils/job.js), but the active startup flow currently centers on RabbitMQ subscription in [`src/index.js`](/c:/myLearnings/Wev-devlopment/X-BackEnd_Dev/Y-Projects/AirLine%20Managment/Reminder-Service/src/index.js).

## Future Improvements

- Add request validation for ticket creation
- Improve error handling and logging
- Add health-check and readiness endpoints
- Add automated tests for controller, service, and repository layers
- Move secrets to a secure configuration provider for production

## License

ISC
