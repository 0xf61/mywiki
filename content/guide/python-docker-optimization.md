---
title: Slim down Python Docker image size with Poetry and Pip
draft: false
date: 2025-06-04
tags:
  - python
  - docker
  - optimization
---

Python package management can be tricky because the default package manager (pip) doesn't track dependency versions like npm does for Node.js.

That's why you should use `poetry` to manage Python packages. It creates a lock file, so you can be sure that the versions will be the same every time you reinstall.

However, this can be a challenge when you want to create a Docker image with poetry. You need to do an extra `pip install poetry` (unless you include it in your base Python image). Also, using poetry to install packages can result in a larger Docker image size.

## Dockerfiles

Here are the Dockerfiles I use to compare using `poetry` and `pip`:

### Dockerfile.poetry

```Dockerfile
FROM python:3.12-slim

WORKDIR /app

# hadolint ignore=DL3013
RUN pip install --no-cache-dir poetry

COPY pyproject.toml .
COPY poetry.lock .

RUN poetry install --only main --no-root --no-directory
```

### Dockerfile.pip

```Dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt --no-cache-dir
```

## Installed packages

```toml
requests = "^2.31.0"
polars = "^0.20.18"
fastapi = "^0.110.1"
pydantic = "^2.6.4"
python-dotenv = "^1.0.1"
langchain = "^0.1.14"
psycopg2-binary = "^2.9.9"
```

## Result

The resulting image sizes are:

```log
benchmark_poetry      latest            23d3105ad0dd   11 seconds ago   520MB
benchmark_pip         latest            b7932a02a8d1   12 hours ago     388MB
```

As you can see, using poetry makes the image `132 MB` larger. If you deploy 12 times per month, that's an extra `1584 MB`.

Even though storage is cheap these days, reducing image sizes can still be helpful! 😎

[Reference](https://karnwong.me/posts/2024/04/slim-down-python-docker-image-size-with-poetry-and-pip/ )
