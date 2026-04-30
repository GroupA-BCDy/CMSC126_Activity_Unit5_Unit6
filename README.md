# CRS 2.0

A summary of the system.

## Tech Stack

- [Frontend](#frontend)
- [Backend](#backend)
- [Database](#database)
- [Other Tools](#othertools)

## Hosting

CRS 2.0 will be hosted on physical servers owned by the university and managed by ICTS, rather than renting cloud space from a third party. This means student data such as grades, enrollment records, and personal information never leave the university's physical premises and remain fully under UPV's control, in compliance with the Data Privacy Act of 2012 (RA 10173).

When a student opens their browser and goes to crs.upv.edu.ph, the request travels through UPV's network and is received by Nginx, which acts as the front door of the system. Nginx decides what to do with each request like if the student is just loading the page, it serves the React files directly to the browser. If the student is doing something like logging in or submitting an enrollment, Nginx passes the request to Django, which handles all the logic such as checking eligibility, verifying available slots, and detecting schedule conflicts. Django runs through Gunicorn, a Python application server that allows hundreds of students to be served simultaneously without any one student having to wait for another. Once Django has what it needs, it reads from or writes to the PostgreSQL database, then sends the result back to the student's browser.

To keep the system fast during peak enrollment, Redis is used as a caching layer. Data that doesn't change often, like the list of subject offerings or the academic calendar, is stored in Redis so that Django doesn't have to query the database every single time a student loads that information. This alone significantly reduces the load on the system during the high-traffic enrollment window.

### Step 1 — The student opens the browser. 

The student types crs.upv.edu.ph into their browser. The request travels through UPV's network to the university's server room.

### Step 2 — Nginx receives the request. 

Nginx is a web server that acts as the front door of the entire system. It is the first thing that receives every incoming request. Nginx is responsible for two things: serving the React frontend files (HTML, JavaScript, CSS) directly to the browser, and forwarding API requests (like "fetch available subjects") to Django. Nginx also handles HTTPS. It holds the SSL certificate issued under UPV's domain, so all traffic between the student's browser and the server is encrypted. No one can intercept or read the data in transit.

### Step 3 — Django processes the request. 

For requests that need server logic (logging in, submitting enrollment, checking conflicts), Nginx forwards them to Gunicorn, which is a Python application server that keeps Django running continuously. Gunicorn handles multiple requests simultaneously — if 200 students are enrolling at the same time, Gunicorn spawns multiple worker processes to handle them in parallel without any student having to wait for another to finish.

### Step 4 — Django reads from or writes to PostgreSQL. 

Django queries the PostgreSQL database for the data it needs — available slots, student records, schedule information — and returns the result back up the chain to the student's browser.

### Step 5 — Redis handles the load. 

For data that does not change frequently (subject listings, academic calendar, room assignments), Redis serves cached copies directly without hitting the database at all. This is what keeps the system fast and responsive during the high-traffic enrollment window.

### Reliability and Safety Measures:

The university should operate at minimum two physical servers — one primary and one standby. If the primary server goes down for any reason (hardware failure, power issue), the standby server takes over automatically with no downtime for students. This is called a failover setup and is standard practice for any critical institutional system.
Automated database backups run every night using PostgreSQL's built-in pg_dump tool. These backups are stored on a separate physical drive and optionally copied to an offsite location, so even in a worst-case scenario like a fire or hardware failure, no student data is permanently lost.

All services run inside Docker containers managed by Docker Compose. This means the entire system — Nginx, Django, PostgreSQL, Redis — can be started, stopped, updated, or restarted with a single command. If a security patch needs to be applied or a bug needs to be fixed, ICTS staff can redeploy the updated container in minutes without taking the whole system offline.

HTTPS is enforced at the Nginx level using a certificate tied to the upv.edu.ph domain. All HTTP traffic is automatically redirected to HTTPS, so there is no scenario where a student's login credentials or enrollment data travels unencrypted over the network.


## Mockups

