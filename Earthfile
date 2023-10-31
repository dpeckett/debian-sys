VERSION 0.7
FROM golang:1.21-bookworm
WORKDIR /workspace

docker-all:
  BUILD --platform=linux/amd64 --platform=linux/arm64 +docker

docker:
  ARG TARGETARCH
  ARG VERSION
  FROM debian:bookworm
  # Disable swap.
  RUN mkdir -p /etc/initramfs-tools/conf.d/ \
    && echo "RESUME=none" > /etc/initramfs-tools/conf.d/resume
  RUN apt update \
    && apt install -y \
      lvm2 \
      sudo \
      systemd \
      systemd-resolved \
      systemd-sysv \
      systemd-timesyncd \
      qemu-guest-agent \
      linux-image-${TARGETARCH} \
    && rm -rf /var/lib/apt/lists/* \
    && rm -f /vmlinuz.old /initrd.img.old
  RUN systemctl enable systemd-networkd \
    && systemctl enable systemd-resolved
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