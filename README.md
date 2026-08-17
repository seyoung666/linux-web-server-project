# Ubuntu 기반 웹 서버 구축 프로젝트

## 프로젝트 개요
VirtualBox를 이용해 Ubuntu Server 가상 환경을 구축하고,
사용자 관리, SSH 원격 접속, Apache 웹 서버 운영, 방화벽 설정까지
실무 서버 관리 기초를 직접 실습한 프로젝트입니다.

## 목표
- Linux 서버 설치 및 기본 관리
- 사용자/권한 관리
- SSH 원격 접속 환경 구성
- Apache 웹 서버 운영
- 방화벽(UFW) 설정
- 서비스 장애 대응 경험

## 사용 기술
- Ubuntu Server 26.04 LTS
- VirtualBox
- SSH (OpenSSH)
- Apache2
- UFW Firewall
- Git / GitHub

## 진행 과정
1. Ubuntu Server 설치 (VirtualBox 가상 환경)
2. hostname 설정 (web-server01)
3. 사용자 계정 관리 (관리자 계정 + engineer 운영 계정, sudo 권한 부여)
4. SSH 서버 설치 및 포트 포워딩을 통한 원격 접속 환경 구축
5. Apache 웹 서버 설치 및 브라우저 접속 확인
6. UFW 방화벽 설정 (SSH, HTTP 포트만 허용)
7. 서비스 장애 시나리오 실습 (Apache 중단/복구, 방화벽 규칙 삭제/복구)

## 트러블슈팅
- VirtualBox에서 vmwgfx 그래픽 드라이버 에러 발생 → safe graphics 모드로 부팅하여 해결
- SSH 서비스 미설치 상태에서 상태 확인 시도 → 설치 순서 오류 인지 및 재설치
- 방화벽 규칙 삭제로 인한 웹 서비스 접속 차단 → 규칙 재등록으로 복구