FROM wordpress:latest

RUN apt-get update && \
    apt-get install -y libxml2-dev

RUN docker-php-ext-install soap

# Switch to default user 1000:1000
USER 1000:1000
