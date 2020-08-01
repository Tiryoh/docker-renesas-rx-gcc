FROM tiryoh/ubuntu:bionic-dev as builder
USER root
RUN apt-get update && apt-get install -y unzip gcc-8 g++-8 autogen m4 texinfo libmpfr-dev libmpc-dev zlib1g-dev && \
	update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 7 && \
	update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 7 && \
	update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 8 && \
	update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 8 && \
	rm -rf /var/lib/apt/lists/*

USER ubuntu
RUN mkdir -p /home/ubuntu/Downloads && \
	mkdir -p /home/ubuntu/rxgcc/src && \
	mkdir -p /home/ubuntu/rxgcc/build
WORKDIR /home/ubuntu/Downloads
RUN wget https://gcc-renesas.com/downloads/d.php?f=rx/gcc/8.3.0.202002-gnurx/gcc_rx_8.3.0_2020q2.zip -O gcc_rx_8.3.0_2020q2.zip && \
	wget https://gcc-renesas.com/downloads/d.php?f=rx/newlib/8.3.0.202002-gnurx/newlib_rx_3.1.0_2020q2.zip -O newlib_rx_3.1.0_2020q2.zip && \
	wget https://gcc-renesas.com/downloads/d.php?f=rx/binutils/8.3.0.202002-gnurx/binutils_rx_2.24_2020q2.zip -O binutils_rx_2.24_2020q2.zip && \
	wget https://gcc-renesas.com/downloads/d.php?f=rx/gdb/8.3.0.202002-gnurx/gdb_rx_7.8.2_2020q2.zip -O gdb_rx_7.8.2_2020q2.zip && \
	unzip /home/ubuntu/Downloads/gcc_rx_8.3.0_2020q2.zip -d /home/ubuntu/rxgcc/src && \
	unzip /home/ubuntu/Downloads/newlib_rx_3.1.0_2020q2.zip -d /home/ubuntu/rxgcc/src && \
	unzip /home/ubuntu/Downloads/binutils_rx_2.24_2020q2.zip -d /home/ubuntu/rxgcc/src && \
	unzip /home/ubuntu/Downloads/gdb_rx_7.8.2_2020q2.zip -d /home/ubuntu/rxgcc/src && \
	rm -rf /home/ubuntu/Downloads/*.zip

# install autoconf 2.64
WORKDIR /home/ubuntu/Downloads
RUN wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.64.tar.gz && \
	cd /home/ubuntu/rxgcc/src && \
	tar xvf /home/ubuntu/Downloads/autoconf-2.64.tar.gz && \
	cd autoconf-2.64 && \
	./configure --prefix=/usr && \
	make && \
	sudo make install && \
	rm -rf /home/ubuntu/Downloads/*.zip && \
	rm -rf /home/ubuntu/rxgcc/src/autoconf-2.64

# build and install binutils
RUN mkdir -p /home/ubuntu/rxgcc/build/binutils && \
	chmod +x /home/ubuntu/rxgcc/src/binutils/configure && \
	chmod +x /home/ubuntu/rxgcc/src/binutils/missing && \
	chmod +x /home/ubuntu/rxgcc/src/binutils/mkinstalldirs
WORKDIR /home/ubuntu/rxgcc/build/binutils
RUN /home/ubuntu/rxgcc/src/binutils/configure --target=rx-elf --prefix=/usr/local --disable-werror && \
	make && \
	sudo make install

# build and install gcc
RUN mkdir -p /home/ubuntu/rxgcc/build/gcc && \
	chmod +x /home/ubuntu/rxgcc/src/gcc/configure
WORKDIR /home/ubuntu/rxgcc/build/gcc
RUN /home/ubuntu/rxgcc/src/gcc/configure --target=rx-elf --prefix=/usr/local \
	--enable-languages=c,c++ --disable-shared --with-newlib --enable-lto --disable-libstdcxx-pch CC=gcc \
	--disable-muiltlib --disable-nls --disable-libmudflap --with-system-zlib --disable-libssp --disable-libgomp \
	--with-gmp=/usr/local --with-mpfr=/usr/local --with-mpc=/usr/local && \
	chmod +x /home/ubuntu/rxgcc/src/gcc/install-sh && \
	chmod +x /home/ubuntu/rxgcc/src/gcc/move-if-change && \
	chmod +x /home/ubuntu/rxgcc/src/gcc/libiconv/build-aux/install-sh && \
	make all-gcc && \
	sudo make install-gcc

# build and install newlib
RUN mkdir -p /home/ubuntu/rxgcc/build/newlib && \
	chmod +x /home/ubuntu/rxgcc/src/newlib/configure
WORKDIR /home/ubuntu/rxgcc/build/newlib
RUN /home/ubuntu/rxgcc/src/newlib/configure --target=rx-elf --prefix=/usr/local CC=gcc && \
	chmod +x /home/ubuntu/rxgcc/src/newlib/install-sh && \
	make && \
	sudo make install

# build and install all binaries related to gcc
WORKDIR /home/ubuntu/rxgcc/build/gcc
RUN chmod +x /home/ubuntu/rxgcc/src/gcc/libgcc/mkheader.sh && \
	make && \
	sudo make install


# main
FROM tiryoh/ubuntu:bionic-dev
LABEL maintainer="Tiryoh <tiryoh@gmail.com>"

USER root
RUN apt-get update && apt-get install -y unzip gcc-8 g++-8 autogen m4 texinfo libmpfr-dev libmpc-dev zlib1g-dev && \
	update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 7 && \
	update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 7 && \
	update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 8 && \
	update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 8 && \
	rm -rf /var/lib/apt/lists/*

USER ubuntu
WORKDIR /home/ubuntu
COPY --from=builder /usr/local /usr/local

