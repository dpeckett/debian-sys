VERSION 0.7
FROM golang:1.21-bookworm
WORKDIR /workspace

docker-all:
  BUILD --platform=linux/amd64 --platform=linux/arm64 +docker

docker:
  ARG TARGETARCH
  ARG VERSION
  FROM debian:bookworm
  RUN apt update
  RUN apt install -y \
    sudo \
    systemd \
    systemd-resolved \
    systemd-sysv \
    systemd-timesyncd \
    openssh-server \
    qemu-guest-agent \
    linux-image-${TARGETARCH}
  RUN systemctl enable systemd-networkd \
    && systemctl enable systemd-resolved \
    && systemctl enable ssh
  # Fixup sudo permissions
  RUN chown root:root /usr/bin/sudo && chmod 4755 /usr/bin/sudo
  # Add kernel symlink
  RUN ln -s /boot/vmlinuz-*-${TARGETARCH} /boot/vmlinuz
  RUN rm -rf /var/cache/* \
    && journalctl --vacuum-size=1K \
    && rm /etc/machine-id \
    && rm /var/lib/dbus/machine-id \
    && rm /etc/hostname
  SAVE IMAGE --push ghcr.io/dpeckett/debian-sys:${VERSION}
  SAVE IMAGE --push ghcr.io/dpeckett/debian-sys:latest

tools:
  ARG TARGETARCH
  RUN apt update && apt install -y ca-certificates curl jq
  RUN curl -fsSL https://get.docker.com | bash