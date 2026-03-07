# Splunk Deployment Server & Universal Forwarder Lab (Docker)

## Overview

This repository provides a Docker-based Splunk lab environment designed to simulate **Deployment Server (DS) and Universal Forwarder (UF) deployment**.

It supports two deployment modes:

1. [**Base Environment (Unconfigured)**](https://github.com/michaelsayala/splunk-ds-uf-docker-lab/blob/main/docker-compose-unconfigure.yml)
2. [**Preconfigured DS with UF Clients**](https://github.com/michaelsayala/splunk-ds-uf-docker-lab/blob/main/docker-compose-ds.yml)

****
This lab is intended for **learning, testing, troubleshooting, and demonstrating Deployment Server and Universal Forwarder deployment in a controlled environment**.

---

## Architecture

### Base Environment

- Single Splunk Enterprise container acting as a Deployment Server  
- Universal Forwarders not configured by default  
- Suitable for practicing deployment server configuration, app deployment, and client registration  

### Preconfigured DS with UF Clients

- 1 Deployment Server container  
- 1 Universal Forwarder containers configured as clients  
- Deployment server apps preloaded  
- Universal Forwarders automatically poll DS and pull apps  
- Forwarding optional (can forward to a preconfigured Indexer or HF) 

---

## Components

| Component       | Hostname | Web UI Port | Management Port | Role           |
|-----------------|----------|------------|----------------|----------------|
| Deployment Server    | hf1      | 8001       | 8091           | Forwarder      |
| Indexer              | idx1     | 8002       | 8092           | Member         |
| Universal Forwarder  | uf1      | N/A        | 8093           | Cluster Master |

---

## Prerequisites

- Docker  
- Docker Compose  
- External Docker network named `skynet`

Create the required Docker network before deployment:


```bash
docker network create skynet
```

---

## Repository Structure

```
.
├── docker-compose-ds-uf.yml
├── docker-compose-unconfigure.yml
├── .env.example
└── README.md
```

---

## Environment Variables

Create a `.env` file in the root directory. Do not commit this file to GitHub.

Example `.env`:

```
SPLUNK_PASSWORD=splunkpassword
```

Add `.env` to your `.gitignore`.

---

## Deployment Instructions

### Deploy Base Environment (Unconfigured)

This deploys a standalone DS and UF clients without app deployment or forwarding configured.

```bash
docker compose -f docker-compose_unconfigure.yml up -d
```

Access the Deployment Server:

- DS1: http://localhost:8001  

Use this mode if you want to manually configure:

- Deployment Server apps
- UF client registration (`splunk set deploy-poll`)
- Optional forwarding to Indexers/HF

---

### Deploy Preconfigured DS with UF Clients

This deployment automatically configures:

- Deployment Server with preloaded apps  
- Universal Forwarders polling the DS  
- Optional forwarding to a configured Indexer or HF  

```bash
docker compose -f docker-compose-ds-uf.yml up -d
```

Access the Deployment Server:

- DS1: http://localhost:8006

Verify UF client registration:

```bash
docker exec -it uf1 /opt/splunkforwarder/bin/splunk show deploy-poll
```

**Expected Output:**  
```
Client is configured to pull apps from DS: ds1:8089
Apps deployed successfully
```
