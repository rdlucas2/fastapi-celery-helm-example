# pull official base image
FROM python:3.9.5-slim-buster as base

# set work directory
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# copy project
COPY . .

FROM base as web
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--reload"]

FROM base as worker
CMD ["celery", "worker", "--app=worker.celery", "--loglevel=info", "--logfile=logs/celery.log"]

FROM base as dashboard
CMD ["flower", "--app=worker.celery", "--port=5555", "--broker=redis://redis:6379/0"]
