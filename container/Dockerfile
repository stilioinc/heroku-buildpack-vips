FROM heroku/heroku:18

ARG VIPS_VERSION

ARG DEBIAN_FRONTEND=noninteractive

# useful build tools ... we need gtk-doc to build orc, since they don't ship
# pre-baked tarballs
RUN apt-get update && apt-get install -y -qq \
	build-essential \
	autoconf \
	automake \
  cmake \
	libtool \
	intltool \
	gtk-doc-tools \
	unzip \
	wget \
	git \
	pkg-config

# heroku:18 includes some libraries, like tiff and jpeg, as part of the
# run-time platform, and we want to use those libs if we can
#
# see https://devcenter.heroku.com/articles/stack-packages
#
# libgsf needs libxml2
RUN apt-get update && apt-get install -y -qq \
	glib-2.0-dev \
	libexpat-dev \
	librsvg2-dev \
	libpng-dev \
	libjpeg-dev \
	libtiff5-dev \
	libexif-dev \
	liblcms2-dev \
	libxml2-dev \
	libfftw3-dev \
  libpoppler-glib-dev \
  libopenjp2-7-dev \
  libgdk-pixbuf2.0-dev

ARG GIFLIB_VERSION=5.1.4
ARG GIFLIB_URL=http://downloads.sourceforge.net/project/giflib

RUN cd /usr/src \
	&& wget ${GIFLIB_URL}/giflib-$GIFLIB_VERSION.tar.bz2 \
	&& tar xf giflib-${GIFLIB_VERSION}.tar.bz2 \
	&& cd giflib-${GIFLIB_VERSION} \
	&& ./configure --prefix=/usr/src/vips \
	&& make \
	&& make install

ARG ORC_VERSION=0.4.28
ARG ORC_URL=http://cgit.freedesktop.org/gstreamer/orc/snapshot

RUN cd /usr/src \
	&& wget ${ORC_URL}/orc-$ORC_VERSION.tar.gz \
	&& tar xf orc-${ORC_VERSION}.tar.gz \
	&& cd orc-${ORC_VERSION} \
	&& ./autogen.sh \
	&& ./configure --prefix=/usr/src/vips --disable-gtk-doc \
	&& make \
	&& make install

ARG GSF_VERSION=1.14.42
ARG GSF_URL=http://ftp.gnome.org/pub/GNOME/sources/libgsf

RUN cd /usr/src \
	&& wget ${GSF_URL}/${GSF_VERSION%.*}/libgsf-$GSF_VERSION.tar.xz \
	&& tar xf libgsf-${GSF_VERSION}.tar.xz \
	&& cd libgsf-${GSF_VERSION} \
	&& ./configure --prefix=/usr/src/vips --disable-gtk-doc \
	&& make \
	&& make install

ARG POPPLER_VERSION=0.78.0
ARG POPPLER_URL=https://poppler.freedesktop.org/poppler-0.78.0.tar.xz

RUN cd /usr/src \
	&& wget ${POPPLER_URL} \
	&& tar vxf poppler-${POPPLER_VERSION}.tar.xz \
	&& cd poppler-${POPPLER_VERSION} \
	&& export PKG_CONFIG_PATH=/usr/src/vips/lib/pkgconfig \
  && mkdir build \
  && cd build \
	&& cmake .. -DCMAKE_INSTALL_PREFIX=/usr/src/vips \
	&& make \
	&& make install

ARG VIPS_URL=https://github.com/libvips/libvips/releases/download

RUN cd /usr/src \
	&& wget ${VIPS_URL}/v${VIPS_VERSION}/vips-${VIPS_VERSION}.tar.gz \
	&& tar xzf vips-${VIPS_VERSION}.tar.gz \
	&& cd vips-${VIPS_VERSION} \
	&& export PKG_CONFIG_PATH=/usr/src/vips/lib/pkgconfig \
	&& ./configure --prefix=/usr/src/vips --disable-gtk-doc \
	&& make \
	&& make install

# clean the build area ready for packaging
RUN cd /usr/src/vips \
	&& rm bin/gif* bin/orc* bin/gsf* bin/batch_* bin/vips-* \
	&& rm bin/vipsprofile bin/light_correct bin/shrink_width \
	&& strip lib/*.a lib/lib*.so* \
	&& rm -rf share/gtk-doc \
	&& rm -rf share/man \
	&& rm -rf share/thumbnailers

RUN cd /usr/src/vips \
  && rm -rf build \
  && mkdir build \
  && tar czf \
    /usr/src/vips/build/libvips.tar.gz bin include lib
