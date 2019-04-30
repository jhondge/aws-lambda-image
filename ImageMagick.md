# **Support extra file format for ImageMagick in AMI**

You know the amazon lambda service does not allow create own customize environment for lambda server, so we need build the `ImageMagick`library
for ourself and set the node js env path `LD_LIBRARY_PATH` as our library path in the zip    

Support file formats: `heic`, `webp`, `jp2`

## Requirements
* Docker environment
* Source Code
    * [libheic](https://github.com/nokiatech/heif)
        * dependency: [libde256](https://github.com/strukturag/libde265)
    * [webp](https://developers.google.com/speed/webp/)
    * [jp2](https://github.com/uclouvain/openjpeg)

## Build

### Run docker with AMI

* Create workspace directory and goto
* Create `build` direcotry in workspace
* Download the source of libraries to the `build`

```bash
docker run -it -v $PWD:/var/task lambci/lambda:build-nodejs6.10 bash
```
* Goto `build` direcotry
```bash
cd build/
```

### Build libheic

Goto source code and build (The install dir is `/usr/local/lib`)

#### Build libde256

```bash
cd libde265-1.0.3/ && ./autogen.sh && ./configure && make && make install
cd ../
```
**Node:** The libde265-1.0.3 has a issue when load heic image the issue num [56](https://github.com/strukturag/libheif/issues/56#issuecomment-412515858), so need use the `master` branch to build

#### Build heic
* Set env
```bash
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
export LDFLAGS=-L/usr/local/lib
export CPPFLAGS='-I/usr/local/include/libde265 -Wno-error=literal-suffix'
```
* Build heic

```bash
cd libheif-1.4.0/ && ./autogen.sh && ./configure && make && make install
cd ../
```

### Build JP2

```bash
cd openjpeg-2.3.1/ && mkdir build && cd build/
cmake .. -DCMAKE_BUILD_TYPE=Release
make && make install
cd ../../
```
### Build Webp

```bash
cd libwebp-1.0.2/ && ./configure && make && make install
cd ../
```

### Build ImageMagick

#### Configuration

* build static library with flag `--enable-static=yes --enable-shared=no`,
* support heic with flag `--with-heic`
* support webp with flag `--with-openjp2`
* support webp with flag `--with-webp`

#### Build ImageMagick

NOTE: target path: `/etc/imagemagick`

```bash
cd ImageMagick-7.0.8-42/
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
export LDFLAGS=-L/usr/local/lib
export CPPFLAGS='-I/usr/local/include/libheif -I/usr/local/include/openjpeg-2.3 -I/usr/local/include/webp'
./configure --prefix=/etc/imagemagick --enable-shared=no --enable-static=yes --with-heic --with-openjp2 --with-webp
make && make install
```

## Copy libraries

* Go to workspace `/var/task`, create `lib` directory
```bash
cd /var/task && mkdir lib
```
* Copy files
    * copy libs
    ```bash
    cp /usr/local/lib/libde265.so.0 lib/
    cp /usr/local/lib/libheif.so.1 lib/
    cp /usr/local/lib/libopenjp2.so.7 lib/
    cp /usr/local/lib/libwebpdemux.so.2 lib/
    cp /usr/local/lib/libwebp.so.7 lib/
    ```
    * copy binaries
    ```bash
    cp -r /etc/imagemagick/bin /var/task/
    ```

