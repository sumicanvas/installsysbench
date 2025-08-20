# Install sysbench for mysql benchmark test
How to install sysbench for benchmark testing.

## 0. 설치 환경
오라클 클라우드(OCI) VM 환경
OS : OL 9

## 1.Dependency설치

sudo yum -y install make automake libtool pkgconfig libaio-devel

## 2.Mysql 관련 라이브러리 설치 전 GPG 키 설치 (mysql8.4에서는 반드시 필요함)
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-202


## 3.Mysql 관련 라이브러리 설치
- MySQL 을 테스트할 것이기 때문에 이 부분이 필요

sudo dnf install https://repo.mysql.com/mysql80-community-release-el9-1.noarch.rpm  
<img width="685" height="603" alt="image" src="https://github.com/user-attachments/assets/9ccbe065-3cbf-4f3f-8dcd-dca0863671b5" />

sudo dnf module list mysql 
<img width="686" height="206" alt="image" src="https://github.com/user-attachments/assets/0286e3b7-a7bc-4eda-8c30-061db430a248" />


sudo dnf install --nogpgcheck mysql-community-devel openssl-devel  
<img width="829" height="646" alt="image" src="https://github.com/user-attachments/assets/83c6200b-fc92-4885-8965-8eb6d54082ee" />

### 만약 이전에 시도했다가 에러가 났다면 캐시 클린 후 다시 설치 시도
sudo dnf clean all
sudo dnf clean packages

## 4.설치파일 압축 해제
tar -xvf sysbench-1.0.20.tar.gz 

cd sysbench-1.0.20/

## 5. configure 스크립트를 자동 생성

sudo ./autogen.sh  
소스 코드 내부에 configure 스크립트를 자동 생성하는 과정으로 autotools 기반 프로젝트에서 빌드를 준비하는 전처리 단계로 의존성 파일을 구성하고 컴파일 설정 준비

## 6. 컴파일 환경체크 및 makefile 생성
sudo ./configure  --prefix=/usr/sysbench/ --with-mysql-includes=/usr/include/mysql/  --with-mysql-libs=/usr/lib64/mysql/

### 옵션 설명  
    - --prefix=/usr/sysbench/ : sysbench가 설치될 기본 디렉토리를 지정합니다.
    - --with-mysql-includes=/usr/include/mysql/ : MySQL 헤더 파일이 위치한 경로 지정
    - --with-mysql-libs=/usr/lib64/mysql/ : MySQL 클라이언트 라이브러리가 위치한 경로 지정
    - 시스템에 필요한 라이브러리 및 헤더 유무를 확인하고, 빌드 시 경로를 정의

## 7. 소스 컴파일 실행 명령

sudo make -j
- -j 옵션은 병렬 컴파일을 의미하며, CPU 코어 수에 맞춰 빠르게 빌드
- 컴파일러가 소스 코드를 바이너리로 변환하는 과정

## 8. 설치
sudo make install

- 컴파일된 바이너리 파일과 관련된 리소스들을 앞서 --prefix로 지정한 경로(/usr/sysbench/)에 설치
- 설치가 완료되면 sysbench 실행 파일과 관련 파일들이 지정 위치에 배치


## 9. 환경 변수 설정
vi /home/opc/.bashrc 
export PATH=$PATH:$HOME/bin:/usr/sysbench/bin


## 10. 버전 확인
$ sysbench --version

## 11. 테스트 전 prepare
### prepare 전 반드시 DB를 생성해줘야 함. ex) sysbench
sysbench /usr/sysbench/share/sysbench/oltp_read_write.lua    --table-size=1000000   --tables=6  --time=100   --report-interval=10  --rand-type=uniform --db-driver=mysql   --mysql-host=127.0.0.1   --mysql-user='user'   --mysql-password='password'   --mysql-port=3306 --mysql-db=*sysbench*  --threads=64  prepare

## 12. sysbench 테스트

sysbench /usr/sysbench/share/sysbench/oltp_read_write.lua    --table-size=1000000   --tables=6  --time=100   --report-interval=10  --rand-type=uniform --db-driver=mysql   --mysql-host=127.0.0.1   --mysql-user='user'   --mysql-password='password'   --mysql-port=3306 --mysql-db=sysbench  --threads=64  run



