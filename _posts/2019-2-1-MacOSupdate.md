# 맥북 High sierra로 업데이트를 해보자  </br> (~~부제: backup을 안한 자를 위한 복구 팁~~)

2015 late macbook pro를 쓰고 있던 본인은 OS를 업뎃안하는 버릇?이 있었다(~~mac parallels가 OS를 업뎃해버리면 추가 요금을 받아버리는 바람에..~~)

Yosemite 버전을 쓰고있었고 이번에 mojave버전이 나왔길래 appstore에서 업데이트를 했는데.. 다운받는 도중에 뻗어버리고 무한재부팅에 들어가버렸다.

글쓴이는 무한 재부팅을 보고나서 백업의 필요성을 다시 한번 생각하게 되었고 데이터 복원을 목적으로 OS reinstall을 해보기로 함.

1. 첫번째 시도 (recovery mode)</br>

macbook이 꺼진 상태에서 **cmd + R**을 누른 상태에서 전원을 누르면, recovery mode로 들어가진다. macOS 유틸리티 창이 나오면 꼭 두번째 **"macOS 설치"**를 눌러야 데이터가 복원됨.
결론적으로 같은 재부팅 상황으로 들어감

2. 두번째 시도 (booting usb 사용)  

ubuntu bootable usb를 만들고 mac에 꽂은 상태에서 **opt**키를 누른 상태에서 전원을 키게 되면, 원하는 booting option을 고르는 창이 나옴. 이 상태에서 ubuntu usb를 선택함. </br>
이후에 grub창이 나올텐데 이상황에서 **Try Ubuntu without installing**을 누른다. 이후에 우분투 화면이 나오는데, 터미널을 키고 sudo fdisk -l을 통해 맥에서 사용했던 hdd를 찾아야 한다.
</br>
</br>
sudo fdisk -l을 치면 연결된 디스크들이 나오는데, 글쓴이의 상황에서는 /dev/sda2가 관련 디스크였음.
대상 디스크를 찾았으면 이는 raw data이기때문에 관련 file system 으로 mount를 해줘야 함.
high sierra 이전버전은 hfs+ 이고 이후 버전은 apfs로 사용한다.
</br>
</br>
만약 이전버전이라면 hfs+이라면, 다음 커맨드를 사용해서 확인할 수 있음
```
sudo fsck.hfsplus -f /dev/sdXY
```
위 커맨드로 데이터가 해석된다면, 다음이어지는 커맨드로 mount!

```
sudo apt-get install hfsprogs

sudo mount -t hfsplus -o force,rw /dev/sdXY folder_you_want_to_mount
```


만약 mac OS 버전이 high sierra 이후 버전이라면! apfs를 해석해주는 tool을 찾아야 함. 애플이 apfs spec을 공개한 것이 아니라서..
글쓴이가 찾아봤을 때 [apfs fuse](https://github.com/sgan81/apfs-fuse)라는 읽기만 가능한 tool이 있었음!
이를 다운 받아 compile하고
```
apfs-fuse /dev/sda2 target_folder_path
```
를 한 후 들어가보면 이전 데이터들을 확인할 수 있음!



[reference]
- [afps-fuse](https://github.com/sgan81/apfs-fuse)
- [ask-ubuntu how to mount hfs+](https://askubuntu.com/questions/332315/how-to-read-and-write-hfs-journaled-external-hdd-in-ubuntu-without-access-to-os)
