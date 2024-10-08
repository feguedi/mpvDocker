FROM debian:bookworm

LABEL maintainer="Ixe Fernández <feguedi@gmail.com>"
LABEL org.label-schema.description="Custom container for FFMPEG and MPV"
LABEL org.label-schema.name="mpv"
LABEL org.label-schema.schema-version="1.0"
LABEL org.label-schema.vcs-url="https://github.com/feguedi/docker-ffmpeg-mpv"

RUN apt update && apt upgrade

RUN apt install -y \
  autoconf \
  automake \
  build-essential \
  cmake \
  g++ \
  gcc \
  git-core \
  gpgrt-tools \
  libass-dev \
  libcrypt-gcrypt-perl \
  libcrypto++-dev \
  libcrypto++-doc \
  libcrypto++-utils \
  libfreetype6-dev \
  libgcrypt-mingw-w64-dev \
  libgcrypt20 \
  libgcrypt20-dev \
  libgnutls28-dev \
  libmp3lame-dev \
  libnuma-dev \
  libopus-dev \
  libsdl2-dev \
  libssl-dev \
  libtool \
  libva-dev \
  libvdpau-dev \
  libvorbis-dev \
  libvpx-dev \
  libx264-dev \
  libx265-dev \
  libxcb-shm0-dev \
  libxcb-xfixes0-dev \
  libxcb1-dev \
  mesa-va-drivers \
  ninja-build \
  openssl \
  pipx \
  pkg-config \
  pulseaudio \
  python3 \
  python3-pip \
  python3-setuptools \
  python3-venv \
  python3-wheel \
  tar \
  texinfo \
  unzip \
  wget \
  yasm \
  zlib1g-dev

RUN apt install -y xserver-xorg-video-amdgpu libdrm-amdgpu1 libgl1-mesa-dev libvulkan-dev libglu1-mesa-dev

RUN apt install -y \
  libaom-dev \
  libass-dev \
  libdav1d-dev \
  libfreetype6-dev \
  libgnutls28-dev \
  libmp3lame-dev \
  libnuma-dev \
  libopus-dev \
  libsdl2-dev \
  libsvtav1-dev \
  libsvtav1dec-dev \
  libsvtav1enc-dev \
  libtool \
  libunistring-dev \
  libva-dev \
  libvdpau-dev \
  libvorbis-dev \
  libvpx-dev \
  libx264-dev \
  libx265-dev \
  libxcb-shm0-dev \
  libxcb-xfixes0-dev \
  libxcb1-dev \
  nasm

RUN mkdir -p /ffmpeg_sources /bin /libs /ffmpeg_build

WORKDIR /ffmpeg_sources

RUN git clone https://github.com/mesonbuild/meson.git meson && \
  cd meson && \
  pip install --prefer-binary . --break-system-packages
  # PATH="$PATH:$HOME/.local/bin/meson"

RUN wget https://github.com/Netflix/vmaf/archive/v3.0.0.tar.gz && \
  tar xvf v3.0.0.tar.gz && \
  mkdir -p vmaf-3.0.0/libvmaf/build &&\
  cd vmaf-3.0.0/libvmaf/build && \
  meson setup -Denable_tests=false -Denable_docs=false --buildtype=release --default-library=static .. && \
  ninja && \
  ninja install

RUN cd /ffmpeg_sources && \
  git -C fdk-aac pull 2> /dev/null || git clone https://github.com/mstorsjo/fdk-aac && \
  cd fdk-aac && \
  autoreconf -fiv && \
  ./configure && \
  make -j6 && \
  make install

RUN cd /ffmpeg_sources && \
  git -C SVT-AV1 pull 2> /dev/null || git clone https://gitlab.com/AOMediaCodec/SVT-AV1.git && \
  mkdir -p SVT-AV1/build && \
  cd SVT-AV1/build && \
  PATH="/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Release -DBUILD_DEC=OFF -DBUILD_SHARED_LIBS=OFF .. && \
  PATH="/bin:$PATH" make -j6 && \
  make install

WORKDIR /ffmpeg_sources

# RUN wget -O ffmpeg-snapshot.tar.bz2 https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2 && \
#   tar xjvf ffmpeg-snapshot.tar.bz2
RUN wget https://github.com/FFmpeg/FFmpeg/archive/refs/tags/n7.1.zip && \
  unzip n7.1.zip && \
  mv FFmpeg-n7.1 ffmpeg

RUN cd ffmpeg && \
  PATH="/bin:$PATH" PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:/usr/lib/pkgconfig:/ffmpeg_build/lib/pkgconfig" ./configure \
    --pkg-config-flags="--static" \
    --extra-libs="-lpthread -lm" \
    --ld="g++" \
    --enable-debug=3 \
    --enable-gpl \
    --enable-libaom \
    --enable-libass \
    --enable-libfdk-aac \
    --enable-libfreetype \
    --enable-libmp3lame \
    --enable-libopus \
    --enable-libsvtav1 \
    --enable-libdav1d \
    --enable-libvorbis \
    --enable-libvpx \
    --enable-libx264 \
    --enable-libx265 \
    --enable-opengl \
    --enable-openssl \
    --enable-postproc \
    --enable-pthreads \
    --enable-shared \
    --enable-vaapi \
    --enable-version3 \
    --enable-nonfree && \
  make -j6 && \
  make install && \
  hash -r

RUN git clone --recursive https://code.videolan.org/videolan/libplacebo libplacebo && \
  cd libplacebo && \
  git submodule update --init && \
  DIR=./build && \
  mkdir build && \
  meson $DIR && \
  ninja -C$DIR && \
  ninja -Cbuild install

RUN cd / && git clone https://github.com/mpv-player/mpv-build.git

WORKDIR /mpv-build

RUN wget https://github.com/mpv-player/mpv/archive/refs/tags/v0.39.0.zip && unzip v0.39.0.zip && mv mpv-0.39.0 mpv

RUN printf "%s\n" --enable-libx264 >> ffmpeg_options && printf "%s\n" --enable-libmp3lame >> ffmpeg_options && printf "%s\n" --enable-libfdk-aac >> ffmpeg_options

RUN apt install -y vulkan-tools libvulkan-dev mesa-vulkan-drivers
# RUN echo "$(apt-cache search vulkan)" && apt install -y vulkan-amdgpu vulkan-tools libvulkan-dev vulkan-utils mesa-vulkan-drivers

RUN apt install -y libvulkan-dev vulkan-tools libgl1-mesa-dev libdrm-dev

RUN vulkaninfo | grep "Vulkan Instance Version"

RUN ./scripts/mpv-config

RUN rm ./mpv/meson.build

COPY meson.build /mpv-build/mpv/meson.build

RUN ./scripts/mpv-build -j4 

RUN ./install

WORKDIR /

RUN useradd -u 1000 mpv \
  && mkdir -p /home/mpv/media \
  && mkdir -p /home/mpv/.config/pulse \
  && echo "default-server = unix:/run/user/1000/pulse/native" > /home/mpv/.config/pulse/client.conf \
  && echo "autospawn = no" >> /home/mpv/.config/pulse/client.conf \
  && echo "daemon-binary = /bin/true" >> /home/mpv/.config/pulse/client.conf \
  && echo "enable-shm = false" >> /home/mpv/.config/pulse/client.conf

RUN mkdir -p /home/mpv/.config/mpv

COPY mpv /home/mpv/.config

RUN echo "UP add volume +2" > /home/mpv/.config/mpv/input.conf \
  && echo "DOWN add volume -2" >> /home/mpv/.config/mpv/input.conf

RUN chown -R mpv:mpv /home/mpv

USER mpv

WORKDIR /home/mpv/media

VOLUME ["/home/mpv/media"]

CMD ["ash"]
