# CMPE 273 - Week 1 Lab 1: Your First Distributed System

A simple distributed system with two services that communicate over HTTP.

## Overview

- **Service A (Echo API)** - Port 8080: Receives messages and echoes them back
- **Service B (Client)** - Port 8081: Calls Service A and returns combined response

## How to Run Locally

### Prerequisites
- Python 3.10+
- Git

### Step 1: Start Service A (Terminal 1)
```bash
cd service-a
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python app.py
```
Service A will start on http://127.0.0.1:8080

### Step 2: Start Service B (Terminal 2)
```bash
cd service-b
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python app.py
```
Service B will start on http://127.0.0.1:8081

### Step 3: Test the System
```bash
curl "http://127.0.0.1:8081/call-echo?msg=hello"
```

---

## Proof of Success and Failure

### Success Case
*When both services are running:*

```bash
jamespham@Jamess-MacBook-Pro cmpe273-week1-lab1-starter % curl "http://127.0.0.1:8081/call-echo?msg=hello"
{"service_a":{"echo":"hello"},"service_b":"ok"}
```

### Failure Case
*When Service A is stopped (Ctrl+C) and Service B returns 503:*

```bash
jamespham@Jamess-MacBook-Pro cmpe273-week1-lab1-starter % curl "http://127.0.0.1:8081/call-echo?msg=hello"
{"error":"HTTPConnectionPool(host='127.0.0.1', port=8080): Max retries exceeded with url: /echo?msg=hello (Caused by NewConnectionError(\"HTTPConnection(host='127.0.0.1', port=8080): Failed to establish a new connection: [Errno 61] Connection refused\"))","service_a":"unavailable","service_b":"ok"}
```

---

## Questions and Answers

### 1. What makes this distributed?

This system is distributed because the two services are running independently (on two separate processes and ports). In addition, they communicate with each other over the network using HTTP. Since each service runs on separate processes, they occupy their own memory space. They can be started, stopped, and restarted without affecting the other's runtime. They communicate solely through network interfaces using APIs and cannot access each other's memory or state. All these facts make them distributed. If they were part of a monolithic software, stopping service A would immediately crash service B, but this wasn't the case. Instead, service B handles service A's unavailability without any trouble.

### 2. What happens on timeout?

When service B calls the unavailable service A, if it exceeds the allotted timeout of 1 second, a timeout exception occurs. Service B is coded to catch this exception and returns the error code 503 along with an error message to describe the timeout. The purpose of the timeout is the avoid the indefinite hanging of service B should service A is slow or unresponsive.

### 3. What happens if Service A is down?

Similar to the above, when service A is down, a timeout occurs and the requests library throws an exception. Service B catches this exception and returns HTTP 503 with the response `{"service_a":"unavailable","service_b":"ok"}`. The log also records `status=error` along with the specific error message. Nevertheless, service B can still run without crashing and can continue to handle other requests.

### 4. What do your logs show, and how would you debug?

The terminal logs display a timestamp followed by key-value pairs of service name, endpoint, status, and latency. For example:
```bash
2026-02-02 19:13:24,704 service=B endpoint=/call-echo status=ok latency_ms=9
2026-02-02 19:13:24,705 127.0.0.1 - - [02/Feb/2026 19:13:24] "GET /call-echo?msg=hello HTTP/1.1" 200 -
```
When an error occurs, an additional field would show the exception message. For example:
```bash
2026-02-02 19:14:49,147 service=B endpoint=/call-echo status=error error="HTTPConnectionPool(host='127.0.0.1', port=8080): Max retries exceeded with url: /echo?msg=hello (Caused by NewConnectionError("HTTPConnection(host='127.0.0.1', port=8080): Failed to establish a new connection: [Errno 61] Connection refused"))" latency_ms=1
```
For debugging, I would filter out `status=error` entries to identify the specific errors, check the error message to see whether its connection refusal or timeout. In this case, a connection timeout would indicate service A is down.

---

## API Endpoints

### Service A (Port 8080)
| Endpoint | Method | Description | Response |
|----------|--------|-------------|----------|
| `/health` | GET | Health check | `{"status": "ok"}` |
| `/echo?msg=<text>` | GET | Echo message | `{"echo": "<text>"}` |

### Service B (Port 8081)
| Endpoint | Method | Description | Response |
|----------|--------|-------------|----------|
| `/health` | GET | Health check | `{"status": "ok"}` |
| `/call-echo?msg=<text>` | GET | Call Service A | `{"service_b": "ok", "service_a": {"echo": "<text>"}}` |
