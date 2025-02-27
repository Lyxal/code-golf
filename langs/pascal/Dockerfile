FROM alpine:3.21 AS builder

RUN apk add --no-cache build-base curl

ENV ARCH=x86_64-linux VERSION=3.2.2

RUN curl https://downloads.freepascal.org/fpc/dist/$VERSION/$ARCH/fpc-$VERSION.$ARCH.tar | tar x

# Workaround musl vs glibc entrypoint for fpcmkcfg.
RUN mkdir /lib64 \
 && ln -s /lib/ld-musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2

RUN cd fpc-$VERSION.$ARCH && echo -e '/usr\nN\nN\nN\n' | sh install.sh

# Remove some unneeded units.
RUN find /usr/lib/fpc/$VERSION/units/$ARCH \
    -type d -mindepth 1 -maxdepth 1        \
    -not -name fcl-base                    \
    -not -name rtl                         \
    -not -name rtl-console                 \
    -not -name rtl-objpas                  \
    -exec rm -r {} +

COPY pascal.c .

RUN gcc -Wall -Werror -Wextra -o pascal -s pascal.c

FROM codegolf/lang-base

COPY --from=0 /lib/ld-musl-x86_64.so.1  /lib/
COPY --from=0 /etc/fpc.cfg              /etc/
COPY --from=0 /pascal                   \
              /usr/bin/fpc              \
              /usr/bin/ld               /usr/bin/
COPY --from=0 /usr/lib/libbfd-2.43.1.so \
              /usr/lib/libctf.so.0      \
              /usr/lib/libjansson.so.4  \
              /usr/lib/libsframe.so.1   \
              /usr/lib/libz.so.1        \
              /usr/lib/libzstd.so.1     /usr/lib/
COPY --from=0 /usr/lib/fpc              /usr/lib/fpc

ENTRYPOINT ["pascal"]

CMD ["--version"]
