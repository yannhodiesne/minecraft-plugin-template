########################################################
############## We use a java base image ################
########################################################
FROM azul/zulu-openjdk-alpine:17-jre AS build
RUN apk add curl jq
RUN apk add dos2unix --update-cache --repository http://dl-3.alpinelinux.org/alpine/edge/community/ --allow-untrusted

ARG version=1.19


########################################################
############## Download Paper with API #################
########################################################
WORKDIR /opt/minecraft
COPY ./getpaperserver.sh /
RUN dos2unix /getpaperserver.sh && chmod +x /getpaperserver.sh
RUN /getpaperserver.sh ${version}

# Run paperclip and obtain patched jar
RUN java -Dpaperclip.patchonly=true -jar /opt/minecraft/paperclip.jar; exit 0

COPY /docker-entrypoint.sh /
RUN dos2unix /docker-entrypoint.sh

########################################################
############## Running environment #####################
########################################################
FROM azul/zulu-openjdk-alpine:17-jre AS runtime

# Working directory
WORKDIR /data

# Obtain runable jar from build stage
COPY --from=build /opt/minecraft/paperclip.jar /opt/minecraft/paperspigot.jar

# Volumes for the external data (Server, World, Config...)
VOLUME "/data"

# Expose minecraft port
EXPOSE 25565/tcp
EXPOSE 25565/udp

# Set memory size
ARG memory_size=3G
ENV MEMORYSIZE=$memory_size

# Set Java Flags
ARG java_flags="-Dlog4j2.formatMsgNoLookups=true -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=mcflags.emc.gs -Dcom.mojang.eula.agree=true"
ENV JAVAFLAGS=$java_flags

WORKDIR /data

COPY --from=build /docker-entrypoint.sh /opt/minecraft
RUN chmod +x /opt/minecraft/docker-entrypoint.sh

# Install gosu
RUN set -eux; \
	apk update; \
	apk add --no-cache su-exec bash;

# Entrypoint
ENTRYPOINT ["bash", "/opt/minecraft/docker-entrypoint.sh"]
