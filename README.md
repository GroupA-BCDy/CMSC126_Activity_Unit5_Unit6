# UPV CRS 2.0

University of the Philippines Visayas Computerized Registration System 2.0 (UPV CRS 2.0) is an improved version of the university’s course registration platform designed to enhance usability, efficiency, and accessibility through a more user-friendly interface and a compact, readable layout. The system streamlines navigation and interaction, reduces visual clutter, and optimizes performance to ensure faster, smoother use. It is also built with scalability in mind, allowing for future enhancements while maintaining a reliable and intuitive user experience aligned with modern web standards.

### Team Members

- Blancaflor, Leona
- Dy, Nicole Ashley
- Sajorne, Chrystie Rae

## Tech Stack

### i. Frontend — React.js

React.js is the most widely adopted frontend library in the world, maintained by Meta and used in production by companies like Facebook, Instagram, Netflix, and Airbnb. It builds the user interface which is everything the student or admin sees and interacts with.

For CRS 2.0, React handles the dynamic parts of the system: the enrollment form that updates in real time, the schedule grid that highlights conflicts as you add subjects, and the subject search that filters results instantly without reloading the page. These interactions would be clunky and slow with plain HTML and JavaScript alone.

React is also component-based, meaning we build reusable pieces and reuse them across every screen. This keeps the codebase clean and easy to maintain long-term, even when developers rotate.

#### Reference: https://react.dev 

The official documentation is maintained by Meta. Stack Overflow's 2023 Developer Survey ranked React as the most used web framework for the 11th year in a row.

### ii. Backend — Django (Python)

Django is a high-level Python web framework that follows the "batteries-included" philosophy meaning it comes with authentication, an admin panel, database management, form validation, and security protections all built in. It is used in production by Instagram, Pinterest, Disqus, and Mozilla.

For CRS 2.0, Django serves as the engine behind the scenes. When a student submits an enrollment request, Django processes it via checks if the student is eligible, verifies there are still available slots, checks for schedule conflicts, and writes the result to the database. All of this happens in one request.
The built-in Django Admin panel is especially valuable here since the university registrar can manage subjects, sections, rooms, and academic calendars through a clean web interface without us needing to build a custom admin system from scratch, saving weeks of development time.

Django REST Framework (DRF) is installed on top of Django to expose clean API endpoints that the React frontend communicates with. For example, when a student searches for a subject, React sends a request and Django returns a list of matching subjects in JSON format.

Security-wise, Django has built-in protection against the most common web vulnerabilities like SQL injection, cross-site scripting (XSS), cross-site request forgery (CSRF), and clickjacking. For a university system that handles sensitive student records, this is a hard requirement.

#### Reference: https://www.djangoproject.com https://www.django-rest-framework.org 

Django has been in active development since 2005 and is one of the most well-documented frameworks available.


### iii. Database — PostgreSQL

PostgreSQL is the world's most advanced open-source relational database. It is free, battle-tested at massive scale, and the standard choice for any application that deals with structured, relational data. Reddit, Instagram, Twitch, and the U.S. government all run PostgreSQL in production.

CRS 2.0 data is inherently relational wherein a student belongs to a college, is enrolled in multiple subjects, each subject has multiple sections, each section has a schedule, and each schedule occupies a room at a specific time. All of these connections are foreign keys and joins, which is exactly what a relational database is designed to handle efficiently.

The most critical feature PostgreSQL provides for CRS specifically is transaction safety. During peak enrollment, hundreds of students may try to grab the last available slot in a section at the same time. PostgreSQL's transaction system guarantees that only one of them succeeds, thus, the slot is never double-booked. This is helpful because this is built into how PostgreSQL handles concurrent writes.

PostgreSQL integrates directly with Django through a single configuration line meaning no extra setup required.

#### Reference: https://www.postgresql.org 

PostgreSQL has been in continuous development since 1996 and consistently ranks as the most admired database in Stack Overflow's annual developer surveys.


### iv. Other Tools

#### Redis

Redis is an in-memory data store used as a caching layer. During peak enrollment periods, such as the first day of pre-enrollment, the CRS database would receive possibly hundreds of requests per minute for the same data such as the list of available subjects, the academic calendar, room assignments. Without caching, every single one of those requests hits the database, which slows the entire system down.

Redis solves this by storing that frequently-read data in memory. The first request fetches it from the database and stores a copy in Redis. Every subsequent request reads from Redis instead which is up to 100x faster than a database query. Redis is also used to store user session data, so students stay logged in across page refreshes.

#### Reference: https://redis.io 

Redis is used by Twitter, GitHub, Snapchat, and Stack Overflow.

#### Docker

Docker is a containerization tool that packages the entire application— which would be React frontend, Django backend, PostgreSQL database, and Redis— into isolated containers. Each container includes everything that the service needs to run which are the code, the runtime, the dependencies, and the configuration.
The practical benefit is that the system runs identically on a developer's laptop, on the staging server, and on the production server. There is no risk of "it worked in testing but broke in production" because the environment is exactly the same everywhere. It also makes it easy for ICTS staff to restart, update, or roll back any individual component without touching the others.

#### Reference: https://www.docker.com 

Docker is the industry standard for application deployment, used by virtually every major tech company.

## Hosting

CRS 2.0 will be hosted on physical servers owned by the university and managed by ICTS, rather than renting cloud space from a third party. This means student data such as grades, enrollment records, and personal information never leave the university's physical premises and remain fully under UPV's control, in compliance with the Data Privacy Act of 2012 (RA 10173).

When a student opens their browser and goes to crs.upv.edu.ph, the request travels through UPV's network and is received by Nginx, which acts as the front door of the system. Nginx decides what to do with each request like if the student is just loading the page, it serves the React files directly to the browser. If the student is doing something like logging in or submitting an enrollment, Nginx passes the request to Django, which handles all the logic such as checking eligibility, verifying available slots, and detecting schedule conflicts. Django runs through Gunicorn, a Python application server that allows hundreds of students to be served simultaneously without any one student having to wait for another. Once Django has what it needs, it reads from or writes to the PostgreSQL database, then sends the result back to the student's browser.

To keep the system fast during peak enrollment, Redis is used as a caching layer. Data that doesn't change often, like the list of subject offerings or the academic calendar, is stored in Redis so that Django doesn't have to query the database every single time a student loads that information. This alone significantly reduces the load on the system during the high-traffic enrollment window.

### Reliability and Safety Measures:

The university should operate at minimum two physical servers — one primary and one standby. If the primary server goes down for any reason (hardware failure, power issue), the standby server takes over automatically with no downtime for students. This is called a failover setup and is standard practice for any critical institutional system.
Automated database backups run every night using PostgreSQL's built-in pg_dump tool. These backups are stored on a separate physical drive and optionally copied to an offsite location, so even in a worst-case scenario like a fire or hardware failure, no student data is permanently lost.

All services run inside Docker containers managed by Docker Compose. This means the entire system — Nginx, Django, PostgreSQL, Redis — can be started, stopped, updated, or restarted with a single command. If a security patch needs to be applied or a bug needs to be fixed, ICTS staff can redeploy the updated container in minutes without taking the whole system offline.

HTTPS is enforced at the Nginx level using a certificate tied to the upv.edu.ph domain. All HTTP traffic is automatically redirected to HTTPS, so there is no scenario where a student's login credentials or enrollment data travels unencrypted over the network.

## Mockups

![image alt](https://github.com/GroupA-BCDy/CMSC126_Activity_Unit5_Unit6/blob/6666fa34d880d0f3cd5facd12e7ce10223671cc2/mockup1.png)
![image alt](https://github.com/GroupA-BCDy/CMSC126_Activity_Unit5_Unit6/blob/7647395cd6ce0d5640ddff708481d62b9de28e36/mockup2.png)
![image alt](https://github.com/GroupA-BCDy/CMSC126_Activity_Unit5_Unit6/blob/7647395cd6ce0d5640ddff708481d62b9de28e36/mockup3.png)
![image alt](https://github.com/GroupA-BCDy/CMSC126_Activity_Unit5_Unit6/blob/7647395cd6ce0d5640ddff708481d62b9de28e36/mockup4.png)
![image alt](https://github.com/GroupA-BCDy/CMSC126_Activity_Unit5_Unit6/blob/d5b69e275b890a7f77ffd38c7c0660f7c34cefe8/mockup5.png)
![image alt](https://github.com/GroupA-BCDy/CMSC126_Activity_Unit5_Unit6/blob/7647395cd6ce0d5640ddff708481d62b9de28e36/mockup6.png)
