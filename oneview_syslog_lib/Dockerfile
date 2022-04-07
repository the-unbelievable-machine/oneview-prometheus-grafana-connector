ARG FROM=python:3.6-alpine
FROM ${FROM}

# Set proxy
ARG http_proxy
ENV HTTP_PROXY=$http_proxy
ENV HTTPS_PROXY=$http_proxy

#FIXME: workaround
RUN urlencodedproxy=$HTTP_PROXY \
    && urldecodedproxy=$(echo $HTTP_PROXY | sed 's/%23/#/g') \
    && export HTTP_PROXY=$urldecodedproxy \
    && export HTTPS_PROXY=$urldecodedproxy \
    && apk add --no-cache supervisor

#create local passwordless user with own group we run container under
RUN addgroup -g 2001 ovconnector \
    && adduser -u 2001 -D -G ovconnector ovconnector

# Copy requirements.txt file to install dependencies
COPY requirements.txt /plugin/requirements.txt
COPY packages /plugin/packages
COPY internal /plugin/internal
COPY ov_client /plugin/ov_client
COPY main.py /plugin/main.py

# Install Dependencies
RUN pip install --no-index --find-links=/plugin/packages -r /plugin/requirements.txt

#custom supervisord config for rootless setup
COPY supervisord.conf /etc/
COPY supervisord-log-generator.service /etc/supervisord.d/

#additional fs config needed for rootless setup
RUN mkdir -p /plugin/logs \
    && touch /plugin/logs/activity.log \
    && ln -s /plugin/logs /logs \
    && ln -s /plugin/logs/activity.log /activity.log \
    && chown -R ovconnector:ovconnector /plugin \
    && chown root:ovconnector /etc/supervisord.conf /etc/supervisord.d/supervisord-log-generator.service

USER ovconnector
ENTRYPOINT ["supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]
