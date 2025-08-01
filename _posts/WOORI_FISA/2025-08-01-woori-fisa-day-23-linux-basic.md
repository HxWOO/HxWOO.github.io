---
layout: post
title: "[우리가 만든 피사] 5주차 - 23일차: Linux 기초 다지기"
date: 2025-08-01 00:00:00 +0900
categories: WOORI_FISA
tags: [linux, basic, cli, docker, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

# 📅 5주차 - 23일차

## 🐧 Linux 기초 다지기: 필수 명령어부터 시스템 관리까지

오늘은 리눅스 환경 설정부터 일상적인 파일 및 디렉토리 관리, 그리고 프로세스 제어에 이르는 다양한 리눅스 기본 명령어를 학습했다. 특히 Docker 컨테이너 환경에서 리눅스를 직접 다루며 실용적인 사용법을 익혔다.

> ### 💡 리눅스, 왜 중요할까?
> 리눅스는 서버 운영체제의 표준이자, 클라우드 환경의 핵심이다. 개발자라면 리눅스 환경에 대한 이해는 필수적이다. 오늘 배운 명령어들은 리눅스 시스템을 효율적으로 다루고 문제를 해결하는 데 있어 강력한 기반이 될 것이다.

---

### ⚙️ Docker를 이용한 리눅스 환경 설정

Docker를 활용하여 가볍고 빠르게 리눅스 개발 환경을 구축하는 방법을 실습했다.

#### 1. Ubuntu 컨테이너 실행 및 필수 패키지 설치

```powershell
# Ubuntu 컨테이너에 접속
docker exec -it ubuntu2 /bin/bash

# 필수 외 패키지 모두 설치 (unminimize는 최소 설치된 시스템에서 추가 패키지를 설치하여 일반적인 사용 환경으로 만듦)
$ unminimize

# 서버1에서 기본 유틸리티 및 SSH 서버 설치
apt-get update 
apt-get install net-tools vim openssh-server ssh sudo info man-db less psmisc nano tzdata cron

# 시간대 설정 (예: Asia / Seoul)
# 6. Asia / 69. Seoul

# SSH 원격 접속을 위한 설정 변경: PermitRootLogin yes
# /etc/ssh/sshd_config 파일에서 PermitRootLogin 옵션을 찾아 'yes'로 변경

# root 비밀번호 설정 (외부 접속용)
passwd root  

# SSH 서비스 시작
service ssh start
```

#### 2. SSH를 이용한 컨테이너 간 접속

```bash
# 서버2에서 서버1(ubuntu01) 또는 다른 컨테이너(ubuntu02)로 SSH 접속
# 예: ssh root@127.0.0.1 -p 23 (ID@접속할주소)
```

---

### 🛠️ 리눅스 기본 명령어 익히기

파일 및 디렉토리 관리, 시스템 정보 확인 등 리눅스 사용에 필수적인 명령어들을 살펴보았다.

#### 0. 명령어 사용법 찾기 🔍

리눅스 명령어의 사용법을 모를 때 활용할 수 있는 다양한 방법들이다.

| 명령어 | 설명 | 특징 |
| :--- | :--- | :--- |
| **`whatis`** | 가장 간략한 설명 제공 | 키워드가 완전히 일치해야 함. `mandb` 업데이트 필요 시 `sudo mandb` |
| **`info`** | 상세한 정보 제공 | U: 한 단계 위로, Q: Info 화면 종료 |
| **`명령어 --help`** | 명령어의 간략한 도움말 출력 | 가장 빠르고 기본적인 도움말 |
| **`man`** | 가장 상세한 매뉴얼 페이지 제공 | 엔터: 한 줄씩, 스페이스: 한 화면씩, B: 뒤로, /단어: 검색, n: 다음 검색, q: 종료 |

> ### 💡 `man` 페이지 활용 팁
> `man` 페이지는 일반적으로 8개의 섹션으로 나뉘어 있다. 특정 섹션의 매뉴얼을 보고 싶다면 `man [섹션 번호] [명령어]` 형식으로 사용한다. (예: `man 5 passwd`는 `passwd` 파일 형식에 대한 매뉴얼을 보여줌)

#### 1. `ls` (list) 📂: 파일 및 디렉토리 목록

현재 디렉토리의 파일과 서브 디렉토리를 나열한다.

```bash
$ ls             # 현재 디렉토리 목록
$ ls /usr        # 특정 디렉토리 목록
$ ls -l          # 상세 정보 (long format)
$ ll             # 'ls -l'의 단축 명령어
$ ls -a          # 숨겨진 파일 포함 모든 파일 출력
$ ls -F          # 파일 종류를 나타내는 기호 추가 (/: 디렉토리, *: 실행 파일, @: 심볼릭 링크)
$ ls -aF         # 여러 옵션 함께 사용
$ ls -w 30       # 출력 너비 지정 (long option: --width 30, --width=30)
$ ls --quote-name # 파일 이름을 "로 묶어서 출력 (long option: --quote)
```

#### 2. `pwd` (print working directory) 📍: 현재 작업 디렉토리 출력

현재 작업 중인 디렉토리의 절대 경로를 표시한다.

> ### 💡 절대 경로 vs. 상대 경로
> - **절대 경로**: 루트 디렉토리(`/`)부터 시작하는 전체 경로 (예: `/home/user/documents`). 명확하지만 길고 시스템 의존적일 수 있다.
> - **상대 경로**: 현재 디렉토리를 기준으로 하는 경로. `.` (현재 디렉토리), `..` (부모 디렉토리)를 사용한다. (예: `./file.txt`, `../dir1`)

#### 3. `cd` (change directory) ➡️: 디렉토리 변경

현재 작업 디렉토리를 변경한다.

```bash
$ cd /home/user/Documents # 절대 경로로 이동
$ cd                    # 홈 디렉토리로 이동
$ cd ~                  # 홈 디렉토리로 이동
$ cd ~/hello            # 홈 디렉토리 하위의 hello 디렉토리로 이동
```

#### 4. `mkdir` (make directory) ➕: 새 디렉토리 생성

새로운 디렉토리를 생성한다.

```bash
$ mkdir new_directory         # 새 디렉토리 생성
$ mkdir -p hello/test/dir1    # 중간 경로가 없어도 디렉토리 생성 (-p 옵션)
```

#### 5. `cat` (concatenate) 📄: 파일 내용 출력 및 연결

파일의 내용을 화면에 표시하거나 여러 파일을 연결하여 출력한다.

```bash
$ echo hello >> file.txt    # file.txt에 "hello" 추가
$ cat file.txt              # file.txt 내용 출력
$ cat file1.txt file2.txt > combined.txt # 두 파일 내용을 합쳐 combined.txt 생성
$ cat -n file1.txt          # 행 번호와 함께 파일 내용 출력
$ cat /etc/hostname         # 시스템 호스트 이름 파일 내용 출력
```

#### 6. `cp` (copy) 📋: 파일 또는 디렉토리 복사

파일 또는 디렉토리를 복사한다.

```bash
$ cp file1.txt file3.txt    # file1.txt를 file3.txt로 복사
$ cp -r dir1 dir2           # 디렉토리 복사 (-r: 재귀적 복사)
$ cp -i file.txt new_file.txt # 덮어쓰기 전 확인 (-i 옵션)
```

#### 7. `mv` (move) ↔️: 파일 또는 디렉토리 이동/이름 변경

파일 또는 디렉토리를 이동하거나 이름을 변경한다.

```bash
$ mv file.txt file3.txt     # file.txt를 file3.txt로 이름 변경
$ mv file3.txt dir3/        # file3.txt를 dir3 디렉토리로 이동
$ mv -i file.txt new_file.txt # 덮어쓰기 전 확인 (-i 옵션)
```

#### 8. `rm` (remove) 🗑️: 파일 또는 디렉토리 삭제

파일 또는 디렉토리를 삭제한다. **주의: 삭제 시 복구 불가능!**

```bash
$ rm file.txt               # 파일 삭제
$ rm file.txt file1.txt     # 여러 파일 삭제
$ rm *.txt                  # .txt 확장자를 가진 모든 파일 삭제
$ rm -r directory           # 디렉토리와 그 내용 삭제 (-r: 재귀적 삭제)
$ rm -i *.txt               # 삭제 전 확인 (-i 옵션)
```

> ### ⚠️ `rm` 명령어 사용 시 주의!
> 리눅스에서 `rm` 명령어는 삭제 확인 메시지 없이 즉시 파일을 삭제한다. `-i` 옵션을 사용하면 삭제 전 확인 메시지가 출력되어 실수를 방지할 수 있다.

#### 9. `rmdir` (remove directory) ❌: 빈 디렉토리 제거

비어있는 디렉토리를 제거한다. 비어있지 않으면 에러가 발생한다.

```bash
$ rmdir empty_directory     # 빈 디렉토리 삭제
```

#### 10. `touch` (touch) 📝: 파일 생성 또는 타임스탬프 업데이트

새로운 빈 파일을 생성하거나 기존 파일의 타임스탬프를 업데이트한다.

```bash
$ touch new_file.txt        # 새 빈 파일 생성
$ touch file.txt            # 기존 파일의 타임스탬프 업데이트
$ touch file{1..100}        # file1부터 file100까지 여러 파일 생성
```

#### 11. `which` (which) ❓: 프로그램 경로 확인

명령어의 실행 파일 경로를 확인한다.

```bash
$ which crontab             # crontab 명령어의 실행 파일 경로 확인
```

#### 12. `whereis` (where is) 🗺️: 프로그램 관련 파일 확인

명령어와 관련된 모든 파일(실행 파일, 소스, 매뉴얼 페이지 등)의 위치를 보여준다.

```bash
$ whereis crontab           # crontab 관련 모든 파일 위치 확인
```

#### 13. `less` (less) 📜: 파일 내용 스크롤 표시

긴 파일의 내용을 스크롤하며 볼 수 있게 해준다. `cat`보다 큰 파일 보기에 적합하다.

```bash
$ less /etc/bash.bashrc     # bash.bashrc 파일 내용 스크롤하며 보기
```

> ### 💡 `less` 사용법
> - `Space`: 한 화면 아래로 이동
> - `b`: 한 화면 위로 이동
> - `/검색어`: 아래 방향으로 검색
> - `?검색어`: 위 방향으로 검색
> - `n`: 다음 검색 결과로 이동
> - `q`: 종료

#### 14. `grep` (global regular expression print) 🔎: 특정 패턴 검색

파일 또는 출력에서 특정 패턴을 검색한다. 파이프(`|`)와 함께 자주 사용된다.

```bash
$ grep "search_term" file.txt # file.txt에서 "search_term" 검색
$ ps -ef | grep ssh          # 실행 중인 프로세스 중 ssh 관련 프로세스 검색
$ grep -i man /etc/passwd    # 대소문자 구분 없이 "man" 검색
$ grep -v man /etc/passwd    # "man"이 포함되지 않은 줄 검색
$ grep -e 검색어1 -e 검색어2 # 여러 검색어 사용
```

#### 15. `echo` (echo) 💬: 텍스트 출력

텍스트 줄을 화면에 표시한다.

```bash
$ echo "Hello, World!"      # "Hello, World!" 출력
```

#### 16. `df` (disk free) 📊: 디스크 여유 공간 확인

디스크 공간 사용량을 표시한다.

```bash
$ df -h                     # 사람이 읽기 쉬운 형식으로 디스크 사용량 표시
```

#### 17. `du` (disk usage) 📦: 디스크 사용량 확인

파일 및 디렉토리의 디스크 사용량을 표시한다.

```bash
$ du -sh /home/user/        # /home/user/ 디렉토리의 총 사용량 요약
```

#### 18. `ps` (process status) 🏃: 프로세스 상태 확인

현재 실행 중인 프로세스를 나열한다.

```bash
$ ps aux                    # 모든 사용자 프로세스 상세 정보 출력
$ ps -ef | grep init        # init 프로세스 검색
```

#### 19. `kill` (kill) 💥: 프로세스 종료

지정된 프로세스를 종료한다.

```bash
$ kill 12345                # PID 12345인 프로세스 종료
```

#### 20. `top` (top) 📈: 실시간 프로세스 모니터링

시스템 프로세스에 대한 동적 뷰를 실시간으로 표시한다.

```bash
$ top                       # 실시간 프로세스 모니터링 시작
```

#### 21. `ln` (link) 🔗: 링크 생성 (하드링크, 심볼릭 링크)

파일에 별명(링크)을 생성한다. 하드링크와 심볼릭(소프트) 링크 두 종류가 있다.

| 구분 | **하드링크 (Hard Link)** | **소프트링크 / 심볼릭 링크 (Soft Link / Symbolic Link)** |
| :--- | :--- | :--- |
| **개념** | 원본 파일과 동일한 물리적 위치(inode)를 가리키는 새 디렉토리 항목 | 다른 파일이나 디렉토리의 경로를 가리키는 특수한 유형의 파일 (바로가기) |
| **특징** | - 원본과 사본 구분이 없음<br>- 다른 파일 시스템/파티션에 걸쳐 있을 수 없음<br>- 디렉토리에는 생성 불가<br>- 원본 삭제 시 모든 하드링크 삭제 전까지 데이터 유지 | - 원본 파일이 삭제되면 "깨진" 링크가 됨<br>- 여러 파일 시스템/파티션에 걸쳐 있을 수 있음<br>- 파일 및 디렉토리 모두에 생성 가능 |
| **생성** | `ln source_file hardlink` | `ln -s source_file_or_directory softlink` |
| **예시** | `ln file.txt hardlink_to_file.txt` | `ln -s file.txt softlink_to_file.txt`<br>`ln -s my_directory softlink_to_my_directory` |

> ### 💡 심볼릭 링크 활용 사례
> - **긴 경로명 단축**: 자주 사용하는 긴 경로를 짧은 링크로 만들어 접근성 향상.
> - **여러 버전 프로그램 관리**: 프로그램의 특정 버전을 가리키는 `latest`와 같은 심볼릭 링크를 사용하여 버전 관리 용이.

#### 22. `find` (find) 🔍: 파일 검색

특정 조건에 맞는 파일이나 디렉토리를 검색한다.

```bash
$ find / | grep log         # 루트 디렉토리에서 "log" 문자열이 포함된 파일/디렉토리 검색
$ find / -type f -name "*log" # 루트 디렉토리에서 이름이 "*log"로 끝나는 파일 검색
```

#### 23. `tree` (tree) 🌳: 디렉토리 구조 출력

디렉토리의 계층적 구조를 트리 형태로 출력한다.

```bash
$ tree                      # 현재 디렉토리의 트리 구조 출력
$ tree / | grep log | less  # 루트 디렉토리의 트리 구조에서 "log" 검색 후 less로 보기
```

#### 24. 텍스트 편집기 (`nano`, `vi`, `vim`, `emacs`) ✍️

파일을 편집하기 위한 다양한 텍스트 편집기.

```bash
$ nano file.txt             # nano 편집기로 file.txt 열기
$ vi file.txt               # vi 편집기로 file.txt 열기
$ vim file.txt              # vim 편집기로 file.txt 열기
```

#### 25. `apt` (Advanced Package Tool) 📦: 패키지 관리

데비안 기반 리눅스(Ubuntu 등)에서 패키지를 관리하는 명령어.

```bash
$ sudo apt update           # 패키지 목록 업데이트
$ sudo apt-get install 패키지명 # 특정 패키지 설치
```

#### 26. `awk` (awk) ✂️: 텍스트 처리 도구

파일이나 데이터 스트림에서 특정 패턴을 찾아 처리하고 분석하는 데 사용되는 강력한 도구.

```bash
# sample.txt 내용:
# John Doe 30
# Jane Smith 25
# Sam Brown 20

$ awk '{ print $2 }' sample.txt         # 두 번째 필드(이름) 출력
$ awk '$3 == 25 { print $0 }' sample.txt # 세 번째 필드(나이)가 25인 줄 전체 출력
$ awk '{ sum += $3; count++ } END { print "Average age:", sum/count }' sample.txt # 평균 나이 계산
$ awk 'BEGIN { print "Processing..." } { print $0 }' sample.txt # BEGIN 블록 사용

# 실습: sample.txt에서 나이가 25살 이상인 사람들의 last 이름과 나이를 출력
$ awk '$3 >= 25 { print $2, $3 }' sample.txt

# sample2.csv 내용:
# John,Doe,30
# Jane,Smith,25
# Sam,Brown,20

$ awk -F',' '{ print $1, $2 }' sample2.csv # 콤마(,)를 필드 구분자로 사용하여 첫 번째, 두 번째 필드 출력

# 프로세스 관리 예시
$ yes > /dev/null &   # 부하 발생
$ ps aux | awk '$3 > 50 {print $1, $3, $11}' # CPU 사용률 50% 초과하는 프로세스 확인
$ ps aux | awk '$3 > 50 { print $2 }' | xargs kill -9 # CPU 사용률 50% 초과하는 프로세스 강제 종료
```

#### 27. `visudo` (visual sudo) 🛡️: sudo 권한 관리

`sudoers` 파일을 안전하게 편집하여 특정 사용자에게 `sudo` 명령 권한을 부여한다.

```bash
$ sudo visudo               # sudoers 파일 편집기 실행
```

> ### 💡 `visudo` 사용 이유
> `sudoers` 파일을 직접 편집하면 문법 오류 시 시스템에 심각한 문제를 초래할 수 있다. `visudo`는 저장 전 문법 검사를 수행하여 안전하게 편집할 수 있도록 돕는다.

---

### ✨ 오늘의 회고

- **`ls`, `cd`, `cp`, `mv`, `rm`** 등 파일 및 디렉토리 관리의 핵심 명령어를 숙달했다.
- **`grep`, `awk`** 와 같은 강력한 텍스트 처리 도구를 활용하여 **cli** 환경에서 데이터를 분석하고 필터링하는 능력을 길렀다.
