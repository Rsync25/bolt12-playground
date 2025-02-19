#################
# Builder image #
#################
FROM maven:3.8.6-openjdk-11-slim AS BUILD

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# References for eclair
ARG ECLAIR_REF=c4934930aa2447d66d1e364b252d6aafc5968cd5
ARG ECLAIR_PLUGINS_REF=cdc26dda96774fdc3b54075df078587574891fb7

WORKDIR /usr/src/eclair
RUN git clone https://github.com/ACINQ/eclair.git . \
    && git reset --hard ${ECLAIR_REF}
RUN mvn install -DskipTests

WORKDIR /usr/src/eclair-plugins
RUN git clone https://github.com/ACINQ/eclair-plugins.git . \
    && git reset --hard ${ECLAIR_PLUGINS_REF}
WORKDIR /usr/src/eclair-plugins/bolt12-tip-jar
RUN mvn package -DskipTests

# ###############
# # final image #
# ###############
FROM openjdk:11.0.4-jre-slim
WORKDIR /app

# Add utils
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    bash jq curl unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# copy and install eclair-cli executable
COPY --from=BUILD /usr/src/eclair/eclair-core/eclair-cli .
RUN chmod +x eclair-cli && mv eclair-cli /sbin/eclair-cli

# we only need the eclair-node.zip to run
COPY --from=BUILD /usr/src/eclair/eclair-node/target/eclair-node-*.zip ./eclair-node.zip
RUN unzip eclair-node.zip && mv eclair-node-* eclair-node && chmod +x eclair-node/bin/eclair-node.sh

# copy and install bolt12-tip-jar plugin
COPY --from=BUILD /usr/src/eclair-plugins/bolt12-tip-jar/target/bolt12-tip-jar-0.10.1-SNAPSHOT.jar .

ENV ECLAIR_DATADIR=/data
ENV JAVA_OPTS=

RUN mkdir -p "$ECLAIR_DATADIR"
VOLUME [ "/data" ]

# ENTRYPOINT JAVA_OPTS="${JAVA_OPTS}" eclair-node/bin/eclair-node.sh "-Declair.datadir=${ECLAIR_DATADIR}"
ENTRYPOINT JAVA_OPTS="${JAVA_OPTS}" eclair-node/bin/eclair-node.sh bolt12-tip-jar-0.10.1-SNAPSHOT.jar "-Declair.datadir=${ECLAIR_DATADIR}"
