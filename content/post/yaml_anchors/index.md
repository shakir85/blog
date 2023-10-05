---
title: "YAML anchors in docker compose"
description: "Simplify your compose files for better readability"
date: 2023-10-04T21:22:40-07:00
categories: ["docker"]
tags: ["good to know"]
draft: true
---

In a docker compose file, you can use YAML anchors to define reusable sections of your configuration and then reference those anchors to set environment variables for multiple services. This can help simplify your docker compose file and avoid repetition. Here's an example of how to use YAML anchors for this purpose:

Suppose you have two services, webapp and database, and you want to set environment variables for both of them:

```yaml
version: '3'
services:
  webapp:
    image: my-webapp-image
    environment: &webapp_env
      - DB_HOST=db
      - DB_PORT=5432

  database:
    image: postgres:12
    environment: &db_env
      - POSTGRES_DB=mydatabase
```