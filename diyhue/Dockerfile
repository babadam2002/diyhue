ARG BUILD_FROM=python:3.11-alpine
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Set the build architecture and convert it from hassio to diyhue style
# armhf, armv7, aarch64, amd64, i386
ARG BUILD_ARCH=aarch64
ENV BUILD_ARCHI = ${BUILD_ARCH}

# Other settings
ENV LANG C.UTF-8
ENV DIYHUE_VERSION=master
ENV WORKING_DIR=/opt/hue-emulator

# Create the needed folders
RUN mkdir diyhue config ${WORKING_DIR}

# Install all needed dependencies
RUN apk add -q -u python3 openssl nmap psmisc iproute2 alpine-sdk build-base
RUN apk add --no-cache bluez

# Download diyhue and unzip it
RUN curl -sL -o diyhue.zip https://github.com/diyhue/diyhue/archive/master.zip
RUN unzip -qo diyhue.zip
RUN rm diyhue.zip
RUN pip3 install -r /diyHue-master/requirements.txt --no-cache-dir

# Install diyHue
RUN cp -r /diyHue-master/BridgeEmulator/flaskUI /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/functions /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/lights /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/sensors /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/HueObjects /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/services /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/configManager /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/logManager /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/HueEmulator3.py /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/githubInstall.sh /opt/hue-emulator/
RUN cp -r /diyHue-master/BridgeEmulator/genCert.sh /diyHue-master/BridgeEmulator/openssl.conf	/opt/hue-emulator/


RUN curl -sL -o diyHueUI.zip https://www.github.com/diyhue/diyHueUI/releases/latest/download/DiyHueUI-release.zip
RUN mkdir /diyhueUI
RUN unzip -qo diyHueUI.zip -d /diyhueUI
RUN rm diyHueUI.zip
RUN cp -r /diyhueUI/dist/index.html			/opt/hue-emulator/flaskUI/templates/
RUN cp -r /diyhueUI/dist/assets			/opt/hue-emulator/flaskUI/
RUN rm -r /diyhueUI
RUN rm -r /diyHue-master

RUN apk --no-cache add --virtual build-deps gcc git make musl-dev \
	&& git clone https://github.com/wolfcw/libfaketime \
	&& cd libfaketime \
	&& git checkout v0.9.10 \
	&& make \
	&& mv src/*.so* /lib && mv src/faketime /bin \
	&& cd - \
	&& rm -rf libfaketime \
	&& apk del build-deps

# Set working directory to the Archive folder
WORKDIR /opt/hue-emulator

# Copy additional local build files
COPY rootfs ./

# Modify user rights of additional files
RUN chmod +x ./select.sh ./genCert.sh && chmod a+x ./run.sh

# Sleect the right binaries depending on the arch
RUN ./select.sh

# Build arguments
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION

## Document volume
VOLUME ["/config"]

# Labels
LABEL \
	io.hass.name="diyHue" \
	io.hass.description="Fully configurable diyHue Emulator" \
	io.hass.arch="${BUILD_ARCH}" \
	io.hass.type="addon" \
	io.hass.version=${BUILD_VERSION} \
	maintainer="diyHue <info@diyHue.org>" \
	org.opencontainers.image.title="diyHue" \
	org.opencontainers.image.description="Fully configurable diyHue Emulator" \
	org.opencontainers.image.vendor="diyHue" \
	org.opencontainers.image.authors="diyHue <info@diyHue.org>" \
	org.opencontainers.image.licenses="MIT" \
	org.opencontainers.image.url="diyHue.org" \
	org.opencontainers.image.source="https://github.com/diyhue/diyhue" \
	org.opencontainers.image.documentation="diyhue.readthedocs.io" \
	org.opencontainers.image.created=${BUILD_DATE} \
	org.opencontainers.image.revision=${BUILD_REF} \
	org.opencontainers.image.version=${BUILD_VERSION}

CMD [ "./run.sh" ]
