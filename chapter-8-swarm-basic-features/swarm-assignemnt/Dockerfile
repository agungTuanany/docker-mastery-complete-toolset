# Create your custom drupal image here, based of official drupal

# Decide Drupal image tag
FROM drupal:8.8.2


# Update and install git
RUN apt-get update && apt-get install -y git \
        && rm -rf /var/lib/apt/lists/*

# Change working directory to clone bootstrap git files
WORKDIR /var/www/html/themes

# Clone bootstrap git files
RUN git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupal.org/project/bootstrap.git \
        && chown -R www-data:www-data bootstrap

# Change working directory back to it's default
WORKDIR /var/www/html
