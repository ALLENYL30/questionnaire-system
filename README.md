# Questionnaire System

A web-based questionnaire management system that allows users to create, manage, and participate in questionnaires. Built with **Vue.js** on the frontend and **Spring Boot** on the backend.

## Features

- **User Authentication**: Supports user roles such as admin and participant.
- **Questionnaire Creation**: Admins can design custom questionnaires with multiple question types.
- **Real-Time Responses**: Collects user responses instantly.
- **Data Visualization**: Displays collected data through visual reports and graphs.

## Tech Stack

- **Frontend**: Vue.js
- **Backend**: Spring Boot (Java)
- **Database**: MySQL (or any relational database)
- **Languages**: Java, JavaScript, HTML, CSS

## Prerequisites

- Java 11 or newer
- Node.js
- MySQL (or other relational database)

## Setup

### Backend (Spring Boot)

1. Clone the repository:
   ```bash
   git clone https://github.com/ALLENYL30/questionnaire-system.git
   cd questionnaire-system
   ```

2. Navigate to the `springboot` directory:
   ```bash
   cd springboot
   ```

3. Set up the MySQL database:
   - Create a database named `questionnaire_db`.
   - Configure your `application.properties` file with the correct database credentials.

4. Build and run the Spring Boot application:
   ```bash
   mvn spring-boot:run
   ```

### Frontend (Vue.js)

1. Navigate to the `vue` directory:
   ```bash
   cd vue
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Run the development server:
   ```bash
   npm run serve
   ```

## Usage

Once both the frontend and backend servers are running, you can access the system in your browser at `http://localhost:8080`.

### Admin Features

- Login to the admin panel to create and manage questionnaires.
- View real-time responses from participants.

### User Features

- Participate in questionnaires.
- View personalized results after submitting responses.
