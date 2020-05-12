FROM registry.access.redhat.com/ubi7/ubi
LABEL maintainer="michael.arnold@clairvoyantsoft.com"

ENV DIST redhat7
ENV PARCEL_DIST el7
ENV INSTALL_DIR /opt/cloudera/parcels

WORKDIR /BUILD

RUN yum -y -e1 -d1 install gcc-7.5.0 gcc-c++ kernel-devel make python3

RUN yum -y -e1 -d1 install readline-devel \
                           tk-devel \
                           gdbm-devel \
                           db4-devel \
                           libpcap-devel \
                           libffi-devel \
                           xz-devel \
                           expat-devel \
                           zlib-devel \
                           bzip2-devel \
                           openssl-devel \
                           ncurses-devel \
                           sqlite-devel \
                           cyrus-sasl-devel.x86_64 \
                           libpqxx-devel
RUN curl http://www.netgull.com/gcc/releases/gcc-8.2.0/gcc-8.2.0.tar.gz && \
	 tar -zxf gcc-8.2.0.tar.gz && \
	 mkdir gcc-8.2.0-build && \
	 cd gcc-8.2.0-build && \
	 ../gcc-8.2.0/configure --enable-languages=c,c++ --disable-multilib && \
	 make -j$(nproc) && \
	 make install

ARG REDIS_VERSION
ENV REDIS_VERSION ${REDIS_VERSION:-1.10.0}
ARG PARCEL_VERSION
ENV PARCEL_VERSION ${PARCEL_VERSION:-b0}
ENV PARCEL_NAME Redis-${REDIS_VERSION}_${PARCEL_VERSION}

RUN mkdir -p ${INSTALL_DIR}/${PARCEL_NAME}
COPY redis-${REDIS_VERSION}.tar.gz /BUILD/
RUN tar -xf /BUILD/redis-${REDIS_VERSION}.tar.gz -C ${INSTALL_DIR}/${PARCEL_NAME}
RUN echo $(gcc --version)

RUN cd  ${INSTALL_DIR}/${PARCEL_NAME}/redis-${REDIS_VERSION} && \
    make

COPY meta/ ${INSTALL_DIR}/${PARCEL_NAME}/meta/
RUN PYVER=$(echo $REDIS_VERSION | awk -F. '{print $1"."$2}') && \
    sed -e "/\"version\" :/s|{{ version }}|${REDIS_VERSION}_${PARCEL_VERSION}|" \
        -i ${INSTALL_DIR}/${PARCEL_NAME}/meta/parcel.json && \
    sed -e "/REDIS_DIRNAME/s|{{ version }}|${REDIS_VERSION}_${PARCEL_VERSION}|" \
        -i ${INSTALL_DIR}/${PARCEL_NAME}/meta/airflow_env.sh

RUN cd ${INSTALL_DIR} && \
    tar zcf /BUILD/${PARCEL_NAME}-${PARCEL_DIST}.parcel ${PARCEL_NAME} && \
    sha1sum /BUILD/${PARCEL_NAME}-${PARCEL_DIST}.parcel | awk '{print $1}' >/BUILD/${PARCEL_NAME}-${PARCEL_DIST}.parcel.sha
COPY make_manifest.py /BUILD/make_manifest.py
RUN PYVER=$(echo $REDIS_VERSION | awk -F. '{print $1"."$2}') && \
    ${INSTALL_DIR}/${PARCEL_NAME}/bin/python${PYVER} /BUILD/make_manifest.py /BUILD/

CMD /bin/bash

