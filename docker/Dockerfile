FROM debian:bookworm-slim

COPY celmet /home/celmet/celmet

RUN apt-get update && \
    apt-get install -y ca-certificates && \
    update-ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    chown -R www-data:www-data /home/celmet && chmod +x /home/celmet/celmet

USER www-data

WORKDIR /home/celmet

ENTRYPOINT ["/home/celmet/celmet"]

CMD [""]
