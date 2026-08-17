\# Ubuntu 기반 웹 서버 구축 프로젝트



\## 프로젝트 개요

VirtualBox를 이용해 Ubuntu Server 가상 환경을 구축하고,

사용자 관리, SSH 원격 접속, Apache 웹 서버 운영, 방화벽 설정,

서비스 장애 대응까지 실무 서버 관리 기초를 직접 실습한 프로젝트입니다.



\## 목표

\- Linux 서버 설치 및 기본 관리

\- 사용자/권한 관리

\- SSH 원격 접속 환경 구성

\- Apache 웹 서버 운영

\- 방화벽(UFW) 설정

\- 서비스 장애 대응 경험



\## 사용 기술

\- Ubuntu Server 26.04 LTS

\- VirtualBox

\- SSH (OpenSSH)

\- Apache2

\- UFW Firewall

\- Git / GitHub



\## 시스템 구성도

내 PC (Windows)

└─ VirtualBox

└─ Ubuntu Server (web-server01)

├─ 사용자 관리 (sy 관리자 계정, engineer 운영 계정)

├─ SSH 원격 접속 (포트 포워딩 2222→22)

├─ Apache 웹 서버 (포트 포워딩 8080→80)

├─ UFW 방화벽 (SSH, HTTP만 허용)

└─ 로그 관리 (access.log, error.log)



\---



\## 1. 서버 기본 설정



\### hostname 설정

서버 식별을 위해 hostname을 `web-server01`로 설정했습니다.



```bash

sudo hostnamectl set-hostname web-server01

```



!\[hostname 설정](images/01-hostname-setting.png)



\---



\## 2. 사용자 계정 관리



관리자 계정(sy)과 별도로, 실무 시나리오를 재현하기 위해 운영 계정(engineer)을 생성하고 sudo 권한을 부여했습니다.



```bash

sudo useradd -m engineer

sudo passwd engineer

sudo usermod -aG sudo engineer

```



> 관리자 계정과 일반 운영 계정을 분리하여 권한 관리 환경을 구성함



!\[사용자 관리](images/02-user-management.png)



\---



\## 3. SSH 원격 접속 환경 구축



서버는 물리적으로 접근하지 않고 원격으로 관리하는 것이 실무 표준이므로, SSH 서버를 설치하고 원격 접속 환경을 구성했습니다.



\### SSH 서버 설치 및 실행

```bash

sudo apt install openssh-server -y

sudo systemctl enable --now ssh

```



!\[SSH 서비스 상태](images/03-ssh-service-running.png)



\### VirtualBox 포트 포워딩 설정

NAT 네트워크 환경에서 호스트(내 PC)가 게스트(VM)의 SSH에 접속할 수 있도록 포트 포워딩을 설정했습니다.



| 이름 | 프로토콜 | 호스트 포트 | 게스트 포트 |

|------|----------|-------------|-------------|

| SSH  | TCP      | 2222        | 22          |

| HTTP | TCP      | 8080        | 80          |



!\[포트 포워딩 설정](images/04-portforward-setting.png)



\### 원격 접속 테스트

Windows 명령 프롬프트에서 실제로 원격 접속에 성공했습니다.



```bash

ssh sy@localhost -p 2222

```



> SSH 기반 원격 서버 관리 환경 구축 완료



\---



\## 4. Apache 웹 서버 구축



```bash

sudo apt install apache2 -y

```



\### 서비스 상태 확인

```bash

sudo systemctl status apache2

```



!\[Apache 상태 확인](images/05-apache-status-running.png)



\### 브라우저 접속 확인

http://localhost:8080



성공적으로 웹 서비스를 제공하는 것을 확인했습니다.



!\[Apache 브라우저 접속 성공](images/06-apache-browser-success.png)



> 내가 만든 Linux 서버가 웹 서비스를 제공함을 확인



\---



\## 5. 방화벽(UFW) 설정



보안 원칙(최소 권한 원칙)에 따라, 필요한 서비스(SSH, HTTP)만 허용하고 나머지 포트는 차단했습니다.



```bash

sudo ufw allow OpenSSH

sudo ufw allow Apache

sudo ufw enable

```



> SSH를 먼저 허용하지 않고 방화벽을 켜면 원격 접속이 끊길 수 있으므로,

> SSH 허용을 방화벽 활성화보다 먼저 수행하는 순서가 중요함을 확인



!\[UFW 방화벽 상태](images/07-ufw-status.png)



\---



\## 6. 서비스 장애 대응 실습



실무에서는 서버가 정상 작동하는 것보다, 장애 발생 시 원인을 진단하고 신속히 복구하는 능력이 중요합니다.

이를 실습하기 위해 두 가지 장애 상황을 의도적으로 재현하고 복구했습니다.



\### 시나리오 1: Apache 서비스 중단 및 복구



\*\*장애 발생\*\*

```bash

sudo systemctl stop apache2

```

서비스 상태가 `inactive (dead)`로 전환되며, 브라우저 접속이 실패하는 것을 확인했습니다.



!\[Apache 장애 발생](images/08-incident-apache-stop.png)



\*\*장애 복구\*\*

```bash

sudo systemctl start apache2

```

서비스가 다시 `active (running)` 상태로 복구되었습니다.



!\[Apache 복구 완료](images/09-incident-apache-recovered.png)



\---



\### 시나리오 2: 방화벽 규칙 삭제로 인한 접속 차단 및 복구



\*\*장애 발생\*\*

```bash

sudo ufw delete allow Apache

```

방화벽 규칙에서 Apache 항목이 삭제되어, SSH는 정상 작동하지만 웹 접속만 차단되는 상황을 재현했습니다.



!\[방화벽 규칙 삭제](images/10-incident-firewall-block-status.png)

!\[브라우저 접속 실패](images/10-incident-firewall-block-browser.png)



\*\*원인 진단\*\*

`ufw status`로 확인한 결과, SSH는 여전히 ALLOW 상태였으나 Apache(HTTP) 규칙만 사라진 것을 확인하여

원인을 방화벽 설정으로 특정했습니다.



\*\*장애 복구\*\*

```bash

sudo ufw allow Apache

```

규칙을 재등록하여 웹 서비스 접속을 정상화했습니다.



!\[방화벽 규칙 복구](images/11-incident-firewall-recovered-status.png)

!\[브라우저 접속 성공](images/11-incident-firewall-recovered-browser.png)



\---



\## 트러블슈팅 기록



| 문제 상황 | 원인 | 해결 방법 |

|-----------|------|-----------|

| VirtualBox 부팅 시 vmwgfx 그래픽 드라이버 오류 발생 | VirtualBox 하이퍼바이저와 vmwgfx 드라이버 간 호환성 문제 | GRUB 메뉴에서 "safe graphics" 모드로 부팅하여 해결 |

| SSH 상태 확인 시 "Unit ssh.service could not be found" 에러 | openssh-server 설치 전 상태 확인을 시도함 (설치 순서 오류) | openssh-server 설치 후 재확인하여 정상 작동 확인 |

| 방화벽 활성화 시 SSH 연결 끊김 위험 인지 | SSH 허용 규칙 없이 방화벽을 켜면 원격 세션이 끊길 수 있음 | SSH 허용을 방화벽 활성화보다 먼저 수행하는 순서로 작업 |

| README.md가 GitHub에서 Markdown으로 렌더링되지 않음 | 일부 편집기에서 저장 시 이스케이프 문자(\\)와 인코딩 오류 발생 | 메모장에서 UTF-8 인코딩으로 파일 재작성하여 해결 |

| 웹 브라우저로 SSH 포트(2222) 접속 시 응답 오류 | SSH와 HTTP는 서로 다른 포트를 사용해야 함을 인지하지 못함 | Apache용 포트 포워딩(8080→80)을 별도로 추가하여 해결 |



\---



\## 배운 점

\- Linux 서버의 기본적인 설치, 계정 관리, 권한 분리의 필요성을 이해함

\- SSH를 통한 원격 서버 관리가 실무 표준 방식임을 체득함

\- 방화벽 설정 시 작업 순서(특히 원격 접속 중 방화벽 활성화)의 중요성을 인지함

\- 서비스 장애는 로그와 상태 확인 명령어(`systemctl status`, `ufw status`)를 통해

&#x20; 체계적으로 진단하고 복구할 수 있음을 실습함

\- Git/GitHub를 활용해 프로젝트 진행 과정과 트러블슈팅 기록을 버전 관리하며 문서화함



