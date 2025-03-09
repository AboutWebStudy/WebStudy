### Ubuntu Server 설치를 공부하게 된 계기

---

- 개인 서버를 구비하게 됐는데 가상 서버랑 달라서 Ubuntu 설치를 어떻게 해야 할지 알아보게 됨
<br>
<br>

### Ubuntu Server 설치 과정

---

- Ubuntu 서버를 검색해 iso 파일을 설치
    - [Get Ubuntu Server | Download | Ubuntu](https://ubuntu.com/download/server)
    - 🤔 iso 파일이란?
        - CD나 DVD 같은 디스크를 하나의 파일로 만든 것으로, OS 설치 디스크 같은 것을 그대로 복사해서 파일로 만들었다고 보면 됨
        <br>
- Rufus를 다운로드
    - 🤔 Rufus의 역할은?
        - iso 파일을 USB에 올바르게 넣어, 부팅이 가능하도록 만드는 프로그램
        <br>
- 서버 설치용 USB를 준비해서 꽂고 Rufus를 통해 iso 파일 부팅하는 USB로 만듦
    - USB를 컴퓨터에 연결하고 Rufus 실행
    - USB를 선택하고 부트 유형 → 선택에서 다운로드한 Ubuntu Server의 iso 파일을 선택
    - 시작 후 팝업창에서 권장 모드로 진행
    <br>
- LAN선을 연결한 서버에 USB를 꽂고 전원을 켜면 Ubuntu server 설치가 진행
<br>
<br>

### 서버에 추가 HDD mount 과정

---

- `sudo fdisk -l` 명령어를 통해 현재 서버에 HDD가 설치 되어 있는지 확인
    
    ```powershell
    Disk /dev/sda: 465.76 GiB, 500107862016 bytes, 976773168 sectors
    Disk model: ST500DM002-1BD14
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 4096 bytes
    I/O size (minimum/optimal): 4096 bytes / 4096 bytes
    
    Disk /dev/nvme0n1: 119.24 GiB, 128035676160 bytes, 250069680 sectors
    Disk model: SAMSUNG MZVLQ128HBHQ-000
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 256B4EEE-E592-4B15-9904-0A73936CF55E
    
    Device           Start       End   Sectors   Size Type
    /dev/nvme0n1p1    2048   2203647   2201600     1G EFI System
    /dev/nvme0n1p2 2203648   6397951   4194304     2G Linux filesystem
    /dev/nvme0n1p3 6397952 250066943 243668992 116.2G Linux filesystem
    
    Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 58.09 GiB, 62377689088 bytes, 121831424 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    ```
    <br>
    
- `df -h`명령어를 통해 사용 중인 디스크 확인
    
    ```powershell
    Filesystem                         Size  Used Avail Use% Mounted on
    tmpfs                              777M  4.6M  772M   1% /run
    efivarfs                           128K   62K   62K  50% /sys/firmware/efi/efivars
    /dev/mapper/ubuntu--vg-ubuntu--lv   57G  7.2G   47G  14% /
    tmpfs                              3.8G     0  3.8G   0% /dev/shm
    tmpfs                              5.0M     0  5.0M   0% /run/lock
    /dev/nvme0n1p2                     2.0G  186M  1.7G  11% /boot
    /dev/nvme0n1p1                     1.1G  6.2M  1.1G   1% /boot/efi
    tmpfs                              777M   12K  777M   1% /run/user/1000
    ```
    
    - sda가 사용 중이지 않은 상태
    <br>
- 마운트 할 디렉터리 생성
    
    ```powershell
    sudo mkdir -p /mnt/hdd1
    ```
    
    - hdd1에는 원하는 이름을 넣으면 됨
    <br>
- 디스크 마운트 명령어
    
    ```powershell
    sudo mount /dev/sda /mnt/hdd1
    ```
    <br>
    
- `df -h`명령어를 통해 마운트 확인
    
    ```powershell
    Filesystem                         Size  Used Avail Use% Mounted on
    tmpfs                              777M  4.6M  772M   1% /run
    efivarfs                           128K   62K   62K  50% /sys/firmware/efi/efivars
    /dev/mapper/ubuntu--vg-ubuntu--lv   57G  7.2G   47G  14% /
    tmpfs                              3.8G     0  3.8G   0% /dev/shm
    tmpfs                              5.0M     0  5.0M   0% /run/lock
    /dev/nvme0n1p2                     2.0G  186M  1.7G  11% /boot
    /dev/nvme0n1p1                     1.1G  6.2M  1.1G   1% /boot/efi
    /dev/sda                           458G   28K  435G   1% /mnt/hdd1
    tmpfs                              777M   12K  777M   1% /run/user/1000
    ```
    <br>
    
- 서버 부팅 시에 자동 마운트가 되도록 설정 : /etc/fstab 파일 수정 필요
    - `sudo blkid /dev/sda`로 UUID 확인
        
        ```powershell
        /dev/sda: UUID="1234-ABCD" TYPE="ext4"
        ```
        
        - 위 결과의 UUID는 임의로 넣음
    - `sudo vi /etc/fstab`로 파일 수정 후 저장
        - 파일의 끝에 TYPE이 ext4인 경우 아래처럼 작성해서 추가
            
            ```powershell
            UUID=1234-ABCD /mnt/hdd1 ext4 defaults 0 2
            ```
            
    - `sudo mount -a`로 변경 사항 적용
    <br>
    <br>

### 마운트 과정 중에 발생한 문제

---

- 마운트 명령어를 입력하니 아래와 같은 문제 발생
    
    ```powershell
    mount: /mnt/hdd1: wrong fs type, bad option, bad superblock on /dev/sda, missing codepage or helper program, or other error.
           dmesg(1) may have more information after failed mount system call.
    ```
    
- 확인해보니 HDD에 파일 시스템 타입이 없었음(TYPE에 대한 값이 표기 되지 않았었음)
    
    ```powershell
    /dev/sda: PTUUID="1234-ABCD" PTTYPE="gpt"
    ```
    

### HDD에 파일 시스템 타입 설정

---

- `sudo fdisk -l` 명령어로 디스크에 파티션이 존재하는지 확인
    
    ```powershell
    Disk /dev/sda: 500GB
    Disklabel type: gpt
    Device     Start       End   Sectors  Size Type
    /dev/sda1  2048  976773167  976771120 465G Linux filesystem
    ```
    
    - 위와 같이 sda1 같은 값이 있다면 파티션이 있는 것으로 파티션 포맷 후 마운트 필요
    <br>
- 파티션을 만들어서 마운트 할 거라면 아래 파티션 나누는 과정 필요
    - 파티션이 없다면, `sudo fdisk /dev/sda`에 진입하여 GPT 타입 파티션 생성
        - `n`(새 파티션 생성) → `p`(기본 파티션) → 기본 값 사용 후, `w`(저장) 입력
        - 🤔 GPT 파티션 테이블이란?
            - 컴퓨터에서 하드디스크를 사용할 때는 파티션(논리적 구역)을 만들어서 사용
            - MBR(예전 방식)과 GPT(최신 방식)이 있는데, MBR은 2TB 초과에서는 사용하기 어렵고 안정성이 낮기 때문에 GPT 방식을 사용
                
                
                | 구분 | MBR (오래된 방식) | GPT (최신 방식) |
                | --- | --- | --- |
                | 최대 지원 용량 | 2TB까지만 가능 | 8ZB(제타바이트, 엄청 큼)까지 가능 |
                | 파티션 개수 | 최대 4개만 가능 | 128개 이상 가능 |
                | 안정성 | 부트 섹터가 손상되면 데이터가 날아갈 수 있음 | 백업 파티션 정보가 있어서 더 안전함 |
                | UEFI 지원 | X (구형 BIOS에서만 사용) | O (최신 UEFI 지원) |
                - UEFI : 컴퓨터를 켰을 때, 어디 있는 운영 체제를 실행할 건지 먼저 결정해야 하는데 이를 도와주는 것
    - `sudo fdisk -l`로 생성된 파티션 확인
    <br>
- 파티션을 안 나눴다면 `sudo mkfs.ext4 /dev/sda`, 나눴다면 파티션 이름을 넣어서 `sudo mkfs.ext4 /dev/sda1`와 같은 명령어로 파일 시스템 생성
    - `mkfs.ext4`를 입력하면 기존 데이터가 삭제될 수 있음
    - 파일 시스템 종류
        - ext4 : 일반적인 리눅스용
        - NTFS : 윈도우와 호환
        - XFS : 고성능 파일 시스템
        <br>
- 마운트 수행
    
    ```powershell
    sudo mkdir -p /mnt/hdd1
    sudo mount /dev/sda1 /mnt/hdd1
    ```
    <br>
    
- 필요하다면 자동 마운트 설정(서버에 추가 HDD mount 진행 부분 참고)
