# php-fpm/Dockerfile

# Define el argumento para la versión de PHP, usada en la imagen base
ARG PHP_VERSION
FROM php:${PHP_VERSION}-fpm-alpine

# Instalar dependencias del sistema necesarias para las extensiones PHP y Composer.
# build-base: Incluye gcc, make, g++, que son esenciales para compilar muchas extensiones.
# linux-headers: Necesario para extensiones de bajo nivel como 'sockets'.
# autoconf, libtool, re2c: Herramientas de compilación.
# libzip-dev, libpng-dev, libjpeg-turbo-dev, freetype-dev, libwebp-dev, zlib-dev: Para la extensión 'gd' y otras de imagen.
# icu-dev: Para la extensión 'intl'.
# postgresql-dev: Aunque usamos MariaDB, a veces se requiere si alguna dependencia lo espera.
# libxml2-dev, openssl-dev, libxslt-dev: Para las extensiones 'soap' y 'xsl'.
# oniguruma-dev: Para la extensión 'mbstring' (aunque no la listamos explícitamente, es buena práctica tenerla).
# gmp-dev: Para la extensión 'gmp'.
# bzip2-dev: Para la extensión 'zip'.
# sqlite-dev: A veces es una dependencia implícita para algunas herramientas o el propio PHP.
# nodejs, npm: Para la compilación de assets de Magento (ej. temas LESS/Sass).
RUN apk add --no-cache \
    git \
    curl \
    libzip-dev \
    libpng-dev \
    libjpeg-turbo-dev \
    freetype-dev \
    icu-dev \
    postgresql-dev \
    libxml2-dev \
    oniguruma-dev \
    gmp-dev \
    libwebp-dev \
    zlib-dev \
    openssl-dev \
    libxslt-dev \
    nodejs \
    npm \
    build-base \
    autoconf \
    libtool \
    bzip2-dev \
    re2c \
    sqlite-dev \
    linux-headers \
    ; # Importante: el último ; y sin espacio final aquí.

# --- INICIO DE INSTALACIÓN DE EXTENSIONES PHP INDIVIDUALMENTE ---
# Cada extensión se instala en su propia capa RUN para facilitar la depuración,
# y para asegurar que los errores sean atómicos si uno falla.

RUN docker-php-ext-install -j$(nproc) pdo_mysql;
RUN docker-php-ext-install -j$(nproc) opcache;
RUN docker-php-ext-install -j$(nproc) zip;
RUN docker-php-ext-install -j$(nproc) bcmath;
RUN docker-php-ext-install -j$(nproc) gmp;
RUN docker-php-ext-install -j$(nproc) sockets;
RUN docker-php-ext-install -j$(nproc) gd;
RUN docker-php-ext-install -j$(nproc) xsl;
RUN docker-php-ext-install -j$(nproc) intl;
RUN docker-php-ext-install -j$(nproc) soap;

# --- FIN DE INSTALACIÓN DE EXTENSIONES PHP ---

# Instalar Composer globalmente en el contenedor.
# Se copia desde una imagen oficial de Composer, lo que es una buena práctica.
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

# Configurar el usuario y grupo 'nginx' con un GID/UID específico (1000)
# para evitar conflictos si el GID 82 ya está en uso en Alpine base.
# Esto asegura que el PHP-FPM y Nginx tengan los mismos permisos sobre los archivos de Magento.
RUN addgroup -g 1000 -S nginx && adduser -u 1000 -D -S -G nginx nginx;

# Establecer límites de memoria y tiempo de ejecución para PHP
# esenciales para las operaciones de Magento (ej. instalación, compilación).
RUN echo "memory_limit = 2G" > /usr/local/etc/php/conf.d/magento-memory.ini \
    && echo "max_execution_time = 1800" >> /usr/local/etc/php/conf.d/magento-memory.ini \
    && echo "opcache.enable_cli = 1" >> /usr/local/etc/php/conf.d/magento-memory.ini;

# Establecer el directorio de trabajo predeterminado para el contenedor.
WORKDIR /var/www/html

# Establecer el usuario bajo el cual se ejecutarán los procesos en este contenedor.
USER nginx