FROM foundationubuntu2204

ARG HOVERSION
ARG LLVMMINGW_NAME=llvm-mingw-20260826-ucrt-ubuntu-22.04-x86_64
ENV PATH="/opt/$LLVMMINGW_NAME/bin:$PATH"

COPY wine /opt/wine
RUN mkdir -p /opt/wine64/amd64 && cp -a /opt/wine/. /opt/wine64/
WORKDIR /opt/wine64/amd64
RUN ../configure --enable-win64 && make -j$(nproc) __tooldeps__ && make -C nls
WORKDIR /opt/wine
RUN ./configure --prefix=/usr/local \
        --with-mingw=clang --enable-archs=arm64ec,aarch64,i386 \
        --enable-tools --disable-tests --host=aarch64-linux-gnu \
        host_alias=aarch64-linux-gnu build_alias=x86_64-linux-gnu \
        --with-wine-tools=../wine64/amd64 CC=aarch64-linux-gnu-gcc && \
    make -j$(nproc) && make install DESTDIR=/opt/staging

COPY fex /opt/fex
ENV PATH="/opt/bylaws-llvm-mingw-20240929-ucrt-ubuntu-20.04-x86_64/bin:$PATH"
WORKDIR /opt/fex
RUN sed -i "s/starting FEX based/starting FEX (Hangover ${HOVERSION}) based/g" \
        Source/Windows/ARM64EC/Module.cpp Source/Windows/WOW64/Module.cpp && \
    cmake -S . -B build_pe -DCMAKE_BUILD_TYPE=Release \
        -DENABLE_JEMALLOC_GLIBC_ALLOC=False -DENABLE_ASSERTIONS=False \
        -DCMAKE_TOOLCHAIN_FILE=Data/CMake/toolchain_mingw.cmake -DENABLE_LTO=False \
        -DMINGW_TRIPLE=aarch64-w64-mingw32 -DTUNE_CPU=none -DBUILD_TESTING=False && \
    cmake --build build_pe -j$(nproc) --target wow64fex && \
    aarch64-w64-mingw32-strip --strip-unneeded build_pe/Bin/libwow64fex.dll
RUN cmake -S . -B build_ec -DCMAKE_BUILD_TYPE=Release \
        -DENABLE_JEMALLOC_GLIBC_ALLOC=False -DENABLE_ASSERTIONS=False \
        -DCMAKE_TOOLCHAIN_FILE=Data/CMake/toolchain_mingw.cmake -DENABLE_LTO=False \
        -DMINGW_TRIPLE=arm64ec-w64-mingw32 -DTUNE_CPU=none -DBUILD_TESTING=False && \
    cmake --build build_ec -j$(nproc) --target arm64ecfex && \
    arm64ec-w64-mingw32-strip --strip-unneeded build_ec/Bin/libarm64ecfex.dll

COPY box64 /opt/box64
ENV PATH="/opt/$LLVMMINGW_NAME/bin:$PATH"
WORKDIR /opt/box64
RUN sed -i "s/GITREV/\"(Hangover ${HOVERSION})\"/g" src/build_info.h && \
    cmake -S . -B build -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_C_COMPILER=aarch64-linux-gnu-gcc -DARM_DYNAREC=ON -DWOW64=ON && \
    cmake --build build -j$(nproc) --target wowbox64 && \
    aarch64-w64-mingw32-strip --strip-unneeded build/wowbox64-prefix/src/wowbox64-build/wowbox64.dll

COPY dxvk /opt/dxvk
ENV PATH="/opt/bylaws-llvm-mingw-20240929-ucrt-ubuntu-20.04-x86_64/bin:$PATH"
WORKDIR /opt/dxvk
RUN mkdir -p /opt/dxvk/output && \
    ./package-release.sh "$(git describe --abbrev=0)" /opt/dxvk/output

WORKDIR /opt
RUN mkdir -p "/opt/bundle/hangover_${HOVERSION}_arm64" && \
    cp -a /opt/staging/usr/local/. "/opt/bundle/hangover_${HOVERSION}_arm64/" && \
    cd "/opt/bundle/hangover_${HOVERSION}_arm64" && \
    cp /opt/fex/build_pe/Bin/libwow64fex.dll /opt/fex/build_ec/Bin/libarm64ecfex.dll \
        /opt/box64/build/wowbox64-prefix/src/wowbox64-build/wowbox64.dll lib/wine/aarch64-windows/ && \
    mkdir -p share/dxvk share/doc/hangover && \
    tar -xzf /opt/dxvk/output/dxvk-*.tar.gz --strip-components=1 -C share/dxvk && \
    cp /opt/wine/COPYING.LIB share/doc/hangover/Wine-LICENSE && \
    cp /opt/fex/LICENSE share/doc/hangover/FEX-LICENSE && \
    cp /opt/box64/LICENSE share/doc/hangover/Box64-LICENSE && \
    cp /opt/dxvk/LICENSE share/doc/hangover/DXVK-LICENSE && \
    tar -cJf /opt/hangover.tar.xz -C /opt/bundle "hangover_${HOVERSION}_arm64"
