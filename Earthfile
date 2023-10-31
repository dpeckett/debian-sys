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
    && apt install -y debootstrap \
    && debootstrap --print-debs bookworm "$(mktemp -d)" 'http://deb.debian.org/debian' | tr '\n' ' ' > packages.txt \
    && apt remove -y debootstrap \
    && apt autoremove -y \
    && xargs -a packages.txt apt install -y \
    && rm packages.txt \
    && apt install -y \
      lvm2 \
      sudo \
      systemd-resolved \
      linux-image-${TARGETARCH} \
      qemu-guest-agent \
    && rm -rf /var/lib/apt/lists/* \
    && rm -f /vmlinuz.old /initrd.img.old
  RUN systemctl enable systemd-networkd \
    && systemctl enable systemd-resolved
  RUN rm -rf /var/cache/* \
    && journalctl --vacuum-size=1K \
    && rm /etc/machine-id \
    && rm /var/lib/dbus/machine-id \
    && rm /etc/hostname
  # TODO: Remove this once we have cloud-init available.
  COPY ./autologin.conf /etc/systemd/system/serial-getty@ttyS0.service.d/autologin.conf
  RUN echo 'auth sufficient pam_listfile.so item=tty sense=allow file=/etc/securetty onerr=fail apply=root' | cat - /etc/pam.d/login > /etc/pam.d/login.new \
    && mv /etc/pam.d/login.new /etc/pam.d/login \
    && (umask 133; echo 'ttyS0' > /etc/securetty) \
    && systemctl enable serial-getty@ttyS0.service
  SAVE IMAGE --push ghcr.io/dpeckett/debian-sys:${VERSION}
  SAVE IMAGE --push ghcr.io/dpeckett/debian-sys:latest