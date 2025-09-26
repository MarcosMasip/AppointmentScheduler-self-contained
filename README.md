[![CircleCI](https://circleci.com/gh/circleci/circleci-docs.svg?style=shield)](https://circleci.com/gh/slabiak/AppointmentScheduler)

# Appointment scheduler

>This is a Spring Boot Web Application to manage and schedule appointments between providers and customers. It has many features such as automatic invoicing, email notifications, appointments cancelation, providers individual working plans with brakes etc.


<a href="https://github.com/slabiak/slabiak.github.io/blob/master/images/appointmentscheduler/calendar.png?raw=true"><img src="https://github.com/slabiak/slabiak.github.io/blob/master/images/appointmentscheduler/calendar.png?raw=true" width="600"></a>

## Demo

Improved and prod ready version of this app can be found at [SpotASlot.com](https://spotaslot.com/) 

The live demo of this repo (master branch) can be found [here](https://appscheduler.onrender.com/) 

You can use the following credentials with live demo:

| Account type | Username | Password 
| --- | --- | --- |
| `admin` | admin | qwerty123 |
| `provider` | provider |qwerty123 |
| `corporate customer` | customer_c |qwerty123 |
| `retail customer` | customer_r |qwerty123 |

## Blog

This application is being described in [devoxify.com](https://devoxify.com/) blog. If you are interested in how this project was created, what issues were encoutered and how they were solved I highly encourage you to visit this blog.

## Quickstart (cross‑platform) — run the app now

Below are the exact commands to get the app running after cloning, for macOS, Windows, and Linux. Choose Option A for the fastest path (Docker only), or Option B to run the backend locally with Dockerized MySQL.

Notes:
- You need Docker Desktop (Windows/macOS) or Docker Engine (Linux). If your setup uses the legacy CLI, replace `docker compose` with `docker-compose`.
- Default URL: http://localhost:8080
- Seeded credentials: username `admin`, password `qwerty123`
- If you’re on Apple silicon (M1/M2/M3) and see image/arch issues, set: `export DOCKER_DEFAULT_PLATFORM=linux/amd64`

### Option A — All‑Docker (recommended)

1) Clone and enter the repo
```bash
git clone https://github.com/MarcosMasip/AppointmentScheduler-self-contained.git
cd AppointmentScheduler-self-contained
```
Expected outcome: The repository is cloned locally and your shell is in its folder.

2) Start the stack (builds backend from this source)
```bash
docker compose up -d
```
Expected outcome: Compose builds the backend image from this repository, then two containers start in the background:
- MySQL 5.7 on port 3306 (executes `src/main/resources/appointmentscheduler.sql` on first boot)
- Backend on port 8080 (connects to the DB via docker network)

3) Check containers are healthy
```bash
docker compose ps
```
Expected outcome: Both `appointmentscheduler_db` and `backend` show “Up”, with ports `3306->3306` and `8080->8080`.

4) Watch backend logs until it says it’s ready (optional)
```bash
docker compose logs -f backend
```
Expected outcome: Lines including “Tomcat started on port(s): 8080” and “Started AppointmentSchedulerApplication”. Press Ctrl+C to stop following logs.

5) Open the app
- macOS: `open http://localhost:8080`
- Linux: `xdg-open http://localhost:8080`
- Windows (PowerShell): `start http://localhost:8080`
Expected outcome: Your browser shows the login page.

6) Log in
- Username: `admin`
- Password: `qwerty123`
Expected outcome: Successful login to the admin dashboard.

Stop when finished:
```bash
docker compose down
```
Expected outcome: Containers are stopped and removed (the named volume persists unless removed explicitly).

### Option B — Local backend + Dockerized MySQL

1) Clone and enter the repo
```bash
git clone https://github.com/MarcosMasip/AppointmentScheduler-self-contained.git
cd AppointmentScheduler-self-contained
```
Expected outcome: The repository is cloned locally and your shell is in its folder.

2) Start MySQL 5.7 in Docker and auto‑init the schema
```bash
docker run --name appointmentscheduler-db \
	-e MYSQL_DATABASE=appointmentscheduler \
	-e MYSQL_USER=user \
	-e MYSQL_PASSWORD=password \
	-e MYSQL_ROOT_PASSWORD=root_pass \
	-p 3306:3306 \
	-v "$PWD/src/main/resources:/docker-entrypoint-initdb.d" \
	-d mysql:5.7
```
Windows PowerShell: replace `$PWD` with `${PWD}` or the full path to the repo (e.g., `C:\path\to\repo`).

Expected outcome: A MySQL container starts on localhost:3306 and executes `appointmentscheduler.sql` on first run to create tables and seed users (including the admin account).

3) Wait for MySQL readiness (optional)
```bash
docker logs -f appointmentscheduler-db
```
Expected outcome: Log shows “ready for connections” and execution of the SQL from `/docker-entrypoint-initdb.d`. Press Ctrl+C to stop following logs.

4) Run the Spring Boot app locally (Maven wrapper)
```bash
./mvnw -DskipTests spring-boot:run
```
Windows (PowerShell): `./mvnw.cmd -DskipTests spring-boot:run`

Expected outcome: The backend downloads dependencies, then prints “Tomcat started on port(s): 8080” and “Started AppointmentSchedulerApplication”. Keep this terminal open.

5) Open the app
- macOS: `open http://localhost:8080`
- Linux: `xdg-open http://localhost:8080`
- Windows (PowerShell): `start http://localhost:8080`
Expected outcome: Your browser shows the login page.

6) Log in
- Username: `admin`
- Password: `qwerty123`
Expected outcome: Successful login to the admin dashboard.

Stop when finished:
- Stop Spring Boot: Ctrl+C in the app terminal
- Stop/remove MySQL container:
```bash
docker rm -f appointmentscheduler-db
```
Expected outcome: The container is removed. To keep data across runs, you can stop without removing (`docker stop appointmentscheduler-db`).

### Troubleshooting
- Port conflicts
	- If port 3306 is busy, stop your local MySQL or change the published port in the `docker run`/compose file.
	- If port 8080 is busy, you can run Spring Boot on a different port: `./mvnw -DskipTests -Dserver.port=8081 spring-boot:run`.
- Apple silicon (M1/M2/M3)
	- If you encounter image/arch errors, run: `export DOCKER_DEFAULT_PLATFORM=linux/amd64` then retry `docker compose up -d`.
- Login shows “Invalid username or password”
	- Ensure the backend can reach MySQL. With Docker Compose, the backend points to `appointmentscheduler_db` on the docker network. Wait until both services are Up and the backend logs show "Tomcat started".
	- Try an incognito window or clear cookies to avoid stale sessions.
	- Reset admin password to the seeded hash (bcrypt for `qwerty123`):
		```bash
		docker exec -i appointmentscheduler-self-contained-appointmentscheduler_db-1 \
			mysql -uuser -ppassword -e \
			"UPDATE appointmentscheduler.users SET password='\$2a\$10\$EqKcp1WFKVQISheBxkQJoOqFbsWDzGJXRz/tjkGq85IZKJJ1IipYi' WHERE username='admin';"
		```
- Emails
	- Email sending uses placeholders in `application.properties`. The app runs fine without a real SMTP server; actual email attempts will fail. To disable mailing locally, set `mailing.enabled=false`.

### Registration and login
- New registrations for both Retail and Corporate customers are supported at the login page links. After completing the form, you should be able to log in immediately with the chosen username and password.
- If you still can’t log in with a newly created account, verify that the `users_roles` table has the appropriate roles and the `users.password` column contains a bcrypt hash (starts with `$2a$`). Compose builds the backend from this source and uses BCrypt for password encoding.

### Credentials and URLs
- App URL: http://localhost:8080 (login page at `/login`)
- Seeded users:
	- admin / qwerty123
	- provider / qwerty123
	- customer_r / qwerty123
	- customer_c / qwerty123

## Steps to Setup (original upstream notes)

These are the original notes from the upstream project; the Quickstart above is the recommended, simplified path for this self-contained repository.

**1. Clone the application**

```bash
git clone https://github.com/slabiak/AppointmentScheduler.git
```

**2. Create MySQL database**

```bash
create database appointmentscheduler
```

- After that run MySQL script to create tables `src/main/resources/appointmentscheduler.sql`

**3. Configure enviroment variables**

+ open `src/main/resources/application.properties`
+ set env variables for JDBC `dbURL`, `dbUsername`, `dbPassword`
+ set env variables for mail server  `mailUsername`, `mailPassword`
+ set jwtSecret, encoded with Base64 `jwtSecret`

**4. Run the app using maven**

```bash
mvn spring-boot:run
```

The app will start running at <http://localhost:8080>

**5. Login to admin account**

+ username: `admin`
+ password: `qwerty123`


## Account types 

`admin` -  is created at database initialization. Admin can add new providers,  services and assign services to providers. Admin can see list of all: appointments, providers, customers, invoices. He can also issue invoices manually for all confirmed appointments.

`provider` - can by created by admin only. Provider can set his own working plan, add brakes to that working plan and change his available services. Provider sees only his own appointments.

`customer retail` - registration page is public and can be created by everyone. Customer can only book new appointments and manage them. This type of customer sees only services which targets retail customer.

`customer corporate` - almost the same as retail customer. The only difference is that this type of account needs to provide VAT number and Company Name and can see only services which targets corporrate customer.

## Booking process

To book a new appointment customer needs to click `New Appointment` button on all appointments page and then:

1. Choose desired work from available works list
2. Choose provider for selected work
3. Choose on of available date which is presented to him
4. Click book on confirmation page

Available hours are calculatated with getAvailableHours function from AppointmentService:

`List<TimePeroid> getAvailableHours(int providerId,int customerId, int workId, LocalDate date)`

This function works as follow:

1. gets selected provider working plan
2. gets working hours from working plan for selected day 
3. excludes all brakes from working hours
4. excludes all providers booked appointments for that day
5. excludes all customers booked appointments for that day
6. gets selected work duration and calculate available time peroids 
7. returns available hours

## Appointments lifecycle

**1. Every appointment has it's own status. Below you can find description for every possible status:**

| Status                | Set by   | When                                           | Condition                                                    |
| --------------------- | -------- | ---------------------------------------------- | ------------------------------------------------------------ |
| `scheudled`           | system   | New appointment is created                     | -                                                            |
| `finished`            | system   | Current date is after appointment end time     | current appointment status is `scheduled` and current date is after appointment end time |
| `confirmed`           | system   | Current date is 24h after appointment end time | current appointment status is `finished` and current date is more than 24h after appointment end time |
| `invoiced`            | system   | Invoice for appointment is created             | -                                                            |
| `canceled`            | customer | Customer clicks cancel button                  | current appointment status is `scheduled` and current date is not less than 24h before appointment start time and user total canceled appointments number for current month is not greater than 1 |
| `rejection requested` | customer | Customer clicks reject button                  | current appointment status is `finished` and current date is not more than 24h after appointment end time |
| `rejection accepted`  | provider | Provider clicks accept rejection button        | current appointment status is `rejection requested`          |

**2. Normal appointment lifecycle is:**

1. scheduled - after user creates new appointment
2. finished - after system time is after appointment end time
3. confirmed - after system time is more than 24h after appointment end time and user didn't request rejection
4. invoiced - after invoiced is issued automatically on the 1st day of next month

**3. Appointment rejection**

After appointment status is changed to finished system automatically sends email to customer with information that appointment is finished. In case that the appointment didn't take place there is also a link attached to that email that allows customer to reject that the appointment didn't take place. That link is valid for 24h after appointment finished time. If user will no click that link then appointment status will be automatically chaned to confirmed after 24h and invoiced at the 1st day of next month. If user will click that link an email is send to provider that his customer requested rejection. If provied will accept that rejection then appointment status will be changed to rejection accepted and appointment will be not invoiced.


**4. Apppointment cancellation**

Every appointment can be canceled by customer or provider. Customer is allowed to cancel 1 appointment in a month no less than 24h before appointment start date. Provider is allowed to cancel his appointments without any limit as long as the appointment status is `scheduled`. 

## Notifications

**1. An email notification is sent when:**

+ appointment is finished
+ appointment rejection is rquested
+ appointment rejection is accepted
+ new appointment is created
+ appointment is canceled
+ invoice is issued

Email templates can be found here: `src\main\resources\templates\email`


## Built With

* [Fullcalendar](https://fullcalendar.io/) - A JavaScript event calendar
* [FlyingSaucer](https://github.com/flyingsaucerproject/flyingsaucer) - Used to generate invoice PDF
* [jjwt](https://github.com/jwtk/jjwt) - Used to generate/validate JWT tokens

## Contribute

Let's together make AppointmentScheduler awesome!

If you have any suggestions/ideas please feel free to write about it. You are also welcome to fork this project and send pull request with your changes.


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
