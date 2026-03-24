FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean

RUN useradd -m -u 1000 appuser

USER appuser

CMD ["nginx", "-g", "daemon off;"]
