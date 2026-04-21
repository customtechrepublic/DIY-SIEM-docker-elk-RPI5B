FROM alpine:latest

RUN apk add --no-cache rsyslog rsyslog-relp

RUN mkdir -p /var/log/remote && \
    chown -R rsyslog:rsyslog /var/log/remote

COPY rsyslog.conf /etc/rsyslog.conf

RUN chmod 644 /etc/rsyslog.conf

USER rsyslog

EXPOSE 514/tcp 514/udp 5140/tcp 5140/udp

CMD ["rsyslogd", "-n"]