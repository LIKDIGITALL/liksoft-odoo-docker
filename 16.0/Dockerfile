FROM debian:bullseye-slim
MAINTAINER LIKSOFT Sarl U <info@odoo.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Retrieve the target architecture to install the correct wkhtmltopdf package
ARG TARGETARCH

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
        libssl-dev \
        node-less \
        npm \
        python3-magic \
        python3-num2words \
        python3-odf \
        python3-pdfminer \
        python3-pip \
        python3-phonenumbers \
        python3-pyldap \
        python3-qrcode \
        python3-renderpm \
        python3-setuptools \
        python3-slugify \
        python3-vobject \
        python3-watchdog \
        python3-xlrd \
        python3-xlwt \
        xz-utils \
        imagemagick \
        libpango1.0-0 \
        libgdk-pixbuf2.0-0 \
        libcairo2 \
        libpangoft2-1.0-0 \
        libmupdf-dev \
        libjpeg62-turbo-dev \
        libfreetype6 \
        libfontconfig1 \
        zlib1g-dev \
        libfreetype6-dev \
        liblcms2-dev \
        tk-dev \
        tcl-dev \
        libmagickwand-dev \
        poppler-utils \
        libblas-dev \
        liblapack-dev \
        gfortran \
        libc-ares-dev \
        libreoffice \
        && if [ -z "${TARGETARCH}" ]; then \
            TARGETARCH="$(dpkg --print-architecture)"; \
        fi; \
        WKHTMLTOPDF_ARCH=${TARGETARCH} && \
        case ${TARGETARCH} in \
        "amd64") WKHTMLTOPDF_ARCH=amd64 && WKHTMLTOPDF_SHA=9df8dd7b1e99782f1cfa19aca665969bbd9cc159  ;; \
        "arm64")  WKHTMLTOPDF_SHA=58c84db46b11ba0e14abb77a32324b1c257f1f22  ;; \
        "ppc64le" | "ppc64el") WKHTMLTOPDF_ARCH=ppc64el && WKHTMLTOPDF_SHA=7ed8f6dcedf5345a3dd4eeb58dc89704d862f9cd  ;; \
        esac \
        && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.bullseye_${WKHTMLTOPDF_ARCH}.deb \
        && echo ${WKHTMLTOPDF_SHA} wkhtmltox.deb | sha1sum -c - \
        && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ bullseye-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update  \
    && apt-get install --no-install-recommends -y postgresql-client \
    && rm -f /etc/apt/sources.list.d/pgdg.list \
    && rm -rf /var/lib/apt/lists/*

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss

# Install Odoo
ENV ODOO_VERSION 16.0
ARG ODOO_RELEASE=20240513
ARG ODOO_SHA=a1e0f0f72a7c09367bc0c09c4b75e801d15e11e6
RUN curl -o odoo.deb -sSL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
    && echo "${ODOO_SHA} odoo.deb" | sha1sum -c - \
    && apt-get update \
    && apt-get -y install --no-install-recommends ./odoo.deb \
    && rm -rf /var/lib/apt/lists/* odoo.deb

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Copy custom files
COPY liksoft/img/logo.png /usr/lib/python3/dist-packages/odoo/addons/web/static/img/logo.png
COPY liksoft/img/nologo.png /usr/lib/python3/dist-packages/odoo/addons/web/static/img/nologo.png
COPY liksoft/img/logo2.png /usr/lib/python3/dist-packages/odoo/addons/web/static/img/logo2.png
COPY liksoft/img/odoo_logo_tiny.png /usr/lib/python3/dist-packages/odoo/addons/web/static/img/odoo_logo_tiny.png
COPY liksoft/img/odoo_logo.svg /usr/lib/python3/dist-packages/odoo/addons/web/static/img/odoo_logo.svg
COPY liksoft/img/odoo_logo_dark.svg /usr/lib/python3/dist-packages/odoo/addons/web/static/img/odoo_logo_dark.svg
COPY liksoft/img/favicon.ico /usr/lib/python3/dist-packages/odoo/addons/web/static/img/favicon.ico

# Copy and install Python requirements
COPY liksoft/requirements.txt /tmp/requirements.txt
RUN pip3 install --no-cache-dir -r /tmp/requirements.txt

# Replace "Odoo" with "LikOS" in web files
RUN find /usr/lib/python3/dist-packages/odoo/addons/web/ -type f -exec sed -i 's/Odoo/LikOS/g' {} +

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
