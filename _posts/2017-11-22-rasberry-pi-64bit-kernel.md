#### 라즈베리 파이 64비트 커널 설치[1/2]



라즈베리파이에 64비트 커널을 깔겠다고 한바탕 삽질을 하고 나서 시도 했던 방법들을 공유하고자 한다.



**native tools 툴을 이용한 라즈베리 파이 3 64비트 커널 빌드 **

아쉽게도 라즈베리 파이에 범용적인 커널인 라즈비안은 64비트를 지원하지 않는다T_T

다른 사람들이 64비트 커널을 지원하는 이미지 파일들을 공유하였고, 이를 하나하나 ^0^ 다 시도 해봤지만 작은 이슈들이 많이 있었다. (~~와이파이 드라이버 호환 이슈 같은?~~)



그래서 직접 64비트 커널을 빌드하여 라즈베리파이 디바이스에 직접 올려보고자 한다.



크게 3가지 방법이 있는데,

 	1. 다른 64비트 ARM 플랫폼에서 native build
	2. 다른 플랫폼에서 cross compiling
	3. pi 3 기종 자체에서  cross compiling



PI 3에서 직접 빌드를 시도해봤는데, 굉장히 느렸기 때문에 2번째 방법을 매우 추천한다. 

커널을 빌드하기 위해서는 [gcc](https://gcc.gnu.org/)와 [aarch64 binutils](https://www.gnu.org/software/binutils/) 두가지 툴이 필요하다.



빌드에 사용한 컴의 스펙은 **Ubuntu 16.04.3 LTS x86_64** 기종이다.



우선 해당 툴들을 본 컴에서 사용하기 위해 denpencies를 맞춰줘야 한다.

`sudo apt-get install build-essential libgmp-dev libmpfr-dev libmpc-dev bc git-core`



---



##### Cross Compliation 툴 빌드

ARM64 컴파일을 다른 아키텍쳐에서 하기 위해선 몇가지 환경이 갖춰져야 하는데, 이 에 필요한 것이 aarch64 버전의 binutils(assembler, linker)와  gcc(C compiler)이다. 해당 툴들을 인스톨 하기 위해서 prefix  혹은 해당 툴들이 인스톨될 경로가 필요하다.

> /opt/aarch64



###### binutils를 깔아보자

최신 binutils를 깔아보자! (이 글을 쓸 시기엔 해당 버전이 최신이었고 나중에 참고하시면 링크 들어가서 최신 버전 확인 해보시길)

> ```
> wget https://ftp.gnu.org/gnu/binutils/binutils-2.29.1.tar.bz2
> ```



그 다음에 아카이브를 풀어보자.

> ```
> tar xf binutils-2.29.1.tar.bz2
> ```



그 다음엔 configure build와 install을 하자.

> ```
> cd binutils-2.29.1
> ./configure --prefix=/opt/aarch64 --target=aarch64-linux-gnu --disable-nls
> make -j4
> sudo make install
> ```



Binutils이 인스톨됐고, 이를 사용하기 위해서 `/opt/aarch64/bin`경로를 path에 추가해야 함.

> ```
> export PATH=$PATH:/opt/aarch64/bin/
> ```



---



##### GCC를 빌드해보자

GCC를 빌드하는 두가지 방식이 있는데,

1. apt-get install -y gcc-aarch64-linux-gnu를 해서 repository에 있는 gcc를 받는 방법
2. gcc source를 다운 로드 해서 빌드하는 방법

repository에는 최신 gcc 버전이 없을 수 있고 기호에 따라 원하는 버전을 다운 받고 싶다면 두 번째 방법을 추천한다



현재 기준으로 가장 안정성 있는 gcc 를 받아 보자

> wget https://ftp.gnu.org/gnu/gcc/gcc-6.4.0/gcc-6.4.0.tar.xz

gcc 아카이브를 풀고

> tar xf gcc-6.4.0.tar.xz

다음 configure gcc로 gcc를 빌드해보자. 해당 configuration은 커널 빌드용으로 minimal C 컴파일러를 빌드한다.

> ```
> mkdir gcc-out
> cd gcc-out
> ../gcc-6.4.0/configure --prefix=/opt/aarch64 --target=aarch64-linux-gnu --with-newlib --without-headers \
>  --disable-shared --disable-threads --disable-libssp --disable-decimal-float \
>  --disable-libquadmath --disable-libvtv --disable-libgomp --disable-libatomic \
>  --enable-languages=c
> make all-gcc -j4
> sudo make install-gcc
> ```



이후 인스톨이 잘 됐다면, 

> ```
> aarch64-linux-gnu-gcc -v
> ```

해당 커맨드로 버전 확인을 해보시라.



---



##### 커널을 빌드해보자

이제 당신은 64 비트 ARM 커널을 빌드할 수 있는 툴체인을 갖고 있다. 이 다음 단계는 라즈베리파이 커널 소스를 다운 받고 세팅한후 빌드하는 것이다.



###### 리눅스 커널 소스 다운로드

[라즈베리 파이 repository](https://github.com/raspberrypi/linux)에서 가장 안정적인 버전을 확인 한 후, 소스를 받도록 하자

> ```
> git clone --depth=1 -b rpi-4.9.y https://github.com/raspberrypi/linux.git
> ```



**라즈베리 파이 3 64비트 커널용 configure **

커널 빌드를 위한 폴더를 만들고 64 비트를 위한 세팅을 한다

> ```
> mkdir kernel-out
> cd linux
> make O=../kernel-out/ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-  bcmrpi3_defconfig
> ```



**마지막으로 커널과 모듈들을 빌드해보자**

해당 커맨드로 커널을 빌드할 수 있다.

> ```
> make -j4 O=../kernel-out/ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
> ```



현재 단계 까지 시도하였다면, 64 비트 ARM 커널을 획득!



[reference]

[raspberry-pi3-build-64-bit-kernel](http://www.tal.org/tutorials/raspberry-pi3-build-64-bit-kernel)

[build-a-64-bit-kernel-for-your-raspberry-pi-3](https://devsidestory.com/build-a-64-bit-kernel-for-your-raspberry-pi-3/)

