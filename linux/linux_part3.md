# Disk mount 절차

서버에 새 디스크가 추가되면 파티션 생성, 포맷, 마운트, 부팅 시 자동 마운트 설정을 진행해야 한다.



- 예시 환경
  - NVMe 디스크의 파티션 이름은 nvme1n1p1처럼 p가 붙으며, SATA 디스크는 sdb1처럼 붙는다.

| 항목               | 값                       |
| ------------------ | ------------------------ |
| OS                 | Ubuntu                   |
| 기존 시스템 디스크 | nvme0n1 (루트 / 사용 중) |
| 신규 디스크        | nvme1n1 (1T, NVMe SSD)   |
| 신규 파티션        | nvme1n1p1                |
| 파일시스템         | ext4                     |
| 마운트 지점        | /data                    |



- 주의 사항

  - 디스크 이름을 잘못 지정하면 기존 데이터가 지워질 수 있으므로, 명령어를 실행하기 전에 항상 대상 디스크 이름을 확인해야 한다.

  - 파티션 생성과 포맷은 대상 디스크에만 영향을 준다.

  - sudo 권한이 필요하다.




- 디스크 추가 여부 확인

  - 디스크가 추가 되었는지 확인
    - 기존 디스크는 파티션이 나뉘어 있고 FSTYPE, MOUNTPOINTS 열에 값이 있다.
    - 신규 디스크는 파티션, FSTYPE, MOUNTPOINTS가 모두 비어 있는 항목이다.
    - loop로 시작하는 항목은 snap 패키지용 가상 장치이므로 무시해도 된다.

  ```bash
  $ lsblk -f
  ```

  - 용량과 모델명 확인
    - 추가한 디스크의 용량, 모델명과 일치하는지 확인한다.

  ```bash
  $ lsblk -o NAME,SIZE,TYPE,MODEL /dev/nvme1n1
  ```

  - 현재 마운트 현황 확인
    - 신규 디스크가 어디에도 마운트되어 있지 않은지 확인한다.

  ```bash
  $ df -h
  ```



- 디스크가 빈 디스크인지 확인

  - 파티션 테이블, 파일 시스템 흔적 확인
    - 출력이 없으면 파티션 테이블이나 파일시스템 흔적이 없는 빈 디스크이다.

  ```bash
  $ sudo wipefs /dev/nvme1n1
  ```

  - 마운트되어 사용 중인지 확인
    - 출력이 없으면 어디에도 마운트되어 있지 않은 것이다.

  ```bash
  $ findmnt -S /dev/nvme1n1
  ```

  - 데이터 존재 여부 확인
    - data라고만 출력되면 비어 있는 디스크이다.

  ```bash
  $ sudo file -s /dev/nvme1n1
  ```

  

- 파티션 생성

  - 디스크 전체를 사용하는 GPT 파티션 하나를 만든다.

  ```bash
  $ sudo parted /dev/nvme1n1 --script mklabel gpt mkpart primary ext4 0% 100%
  ```

  - 명령 구성은 아래와 같다.

    | 부분                        | 의미                                                         |
    | --------------------------- | ------------------------------------------------------------ |
    | sudo                        | 관리자 권한으로 실행                                         |
    | parted                      | 파티션 관리 프로그램                                         |
    | /dev/nvme1n1                | 작업 대상 디스크                                             |
    | --script                    | 중간 질문 없이 바로 실행                                     |
    | mklabel gpt                 | GPT 방식의 파티션 테이블 생성                                |
    | mkpart primary ext4 0% 100% | 디스크 처음부터 끝까지 파티션 하나 생성 (ext4는 표시일 뿐 실제 포맷은 다음 단계에서 진행) |
  
  - 생성 결과 확인.
    - nvme1n1 아래에 nvme1n1p1 파티션이 보이고 용량이 디스크 전체와 같으면 정상이다.
  
  ```bash
  $ lsblk /dev/nvme1n1
  ```



- 파일 시스템 포맷

  - 포맷 전 상태 확인
    - FSTYPE이 비어 있고 file -s 결과가 data이면 포맷해도 되는 상태이다.

  ```bash
  $ lsblk -f /dev/nvme1n1
  $ sudo file -s /dev/nvme1n1p1
  ```

  - 포맷
    - 예시에서는 ext4로 포맷하는데, 다양한 형식이 가능하다.
    - 마지막 줄에 Writing superblocks and filesystem accounting information: done이 출력되면 완료된 것이다.

  ```bash
  $ sudo mkfs.ext4 /dev/nvme1n1p1
  ```

  - 결과 확인
    - FSTYPE에 ext4가 표시되고 UUID 값이 생겼으면 정상이다.

  ```bash
  $ lsblk -f /dev/nvme1n1
  ```



- 마운트

  - 마운트 지점 폴더 확인

  ```bash
  $ ls -la /data
  ```

  - 폴더 생성

  ```bash
  $ sudo mkdir -p /data
  ```

  - 마운트

  ```bash
  $ sudo mount /dev/nvme1n1p1 /data
  ```

  - 확인
    - Filesystem 열에 /dev/nvme1n1p1이, Mounted on 열에 /data가 표시되고, 용량이 보이면 성공이다.
    - 현재 마운트는 임시 상태이므로 재부팅하면 풀린다.

  ```bash
  $ df -h /data
  ```



- 자동 마운트 설정

  - fstab 백업
    - fstab을 잘못 수정하면 부팅에 문제가 생길 수 있으므로 반드시 먼저 백업한다.

  ```bash
  $ sudo cp /etc/fstab ~/fstab.back
  $ ls -l ~/fstab.back
  ```

  - 현재 fstab 내용 확인
    - 기존 항목의 형식(/dev/disk/by-uuid/UUID 방식, 항목 위의 주석 한 줄)을 확인하고, 새 항목도 같은 형식으로 맞춘다.

  ```bash
  $ cat /etc/fstab
  ```

  - 신규 파티션의 UUID 확인

  ```bash
  $ lsblk -f /dev/nvme1n1
  ```

  - fstab에 항목 추가
    - 위에서 확인한 실제 UUID를 입력한다.


  ```bash
  $ sudo vi /etc/fstab
  
  /dev/disk/by-uuid/11111111-2222-3333-4444-555555555555 /data ext4 defaults,nofail 0 2
  ```

  - 각 항목의 의미

    | 순서 | 값                     | 의미                                                         |
    | ---- | ---------------------- | ------------------------------------------------------------ |
    | 1    | /dev/disk/by-uuid/UUID | 연결할 파티션의 고유 ID (장치 이름이 바뀌어도 안전)          |
    | 2    | /data                  | 마운트 지점                                                  |
    | 3    | ext4                   | 파일시스템 종류                                              |
    | 4    | defaults,nofail        | 기본 옵션과 nofail (디스크에 문제가 있어도 부팅은 계속 진행) |
    | 5    | 0                      | dump 백업 대상 여부 (사용 안 함)                             |
    | 6    | 2                      | 부팅 시 파일시스템 검사 순서 (루트 다음)                     |

  - systemd에 변경된 fstab 반영

  ```bash
  $ sudo systemctl daemon-reload
  ```

  - fstsb 문법 검증
    - 0 parse errors, 0 errors, 0 warnings 형태로 출력되면 정상.

  ```bash
  $ sudo findmnt --verify
  ```

  - 마운트 해제 후 fstsb 설정으로 다시 마운트
    - `sudo mount /data`: 마운트 지점만 지정하면 fstab의 해당 항목을 읽어 마운트한다.
    - `/dev/nvme1n1p1`이 `/data`에 연결되어 있는지 확인한다.

  ```bash
  $ sudo umount /data
  $ sudo mount /data
  $ df -h /data
  ```



- 최종 확인

  - 신규 파티션이 /data에 마운트되어 있고 용량과 사용률이 올바르게 표시되는지 확인한다.

  ```bash
  $ lsblk -f
  $ df -h
  ```

