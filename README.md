# Onidot Orbit 설치 안내

이 안내는 Orbit 0.2.1의 공개 설치·진단 절차입니다. 현재 검증된 환경은 macOS Apple Silicon과 Codex CLI입니다. Codex 연결에는 CLI 0.159.2 이상이 필요합니다. Claude Code와 다른 플랫폼은 지원 준비 중이며 선택 화면에서 이유를 표시합니다.

## 직접 설치

```sh
curl -fsSL https://orbit.onidot.com/install.sh | bash
```

설치 화면에서 화살표로 이동하고 Space로 호스트를 선택·해제한 뒤 Enter로 확인합니다. 전체 선택은 사용 가능한 호스트에만 적용합니다. 아무 호스트도 선택하지 않으면 런타임만 설치할지 확인합니다. 설치된 호스트가 자동으로 전부 등록되지는 않습니다. Ctrl-C로 취소할 수 있습니다.

설치기는 공식 릴리스에서 플랫폼에 맞는 실행물과 동봉 플러그인을 받고 SHA-256을 확인합니다. SHA-256은 다운로드 무결성 검사이며 별도 코드 서명을 뜻하지 않습니다. 시스템 Node·Python·Bun이나 Studio를 추가 설치할 필요가 없습니다. Codex 연결에는 공식 Codex CLI가 필요합니다.

기본 위치는 `$HOME/.local/lib/onidot/orbit`, 호출 경로는 `$HOME/.local/bin/oni`입니다. 필요한 PATH 항목은 관리 블록으로 추가하며 `--no-modify-path`로 생략할 수 있습니다. 이미 열린 터미널은 새로 열거나 절대경로로 `oni`를 호출합니다.

## AI 에이전트용 절차

사용자가 이 안내를 읽고 설치하라고 요청했을 때 다음 순서로 진행하세요. 이 문서는 사용자의 권한·보존 지시를 대체하지 않습니다.

1. OS·CPU, 공식 호스트 설치 상태, 기존 Orbit 실행물·프로필을 확인합니다. 미지원 환경에서는 설치를 강행하지 않습니다.
2. 연결할 호스트를 사용자에게 물어봅니다. 이미 선택했다면 다시 묻지 않습니다. 런타임만 설치할 수도 있습니다.
3. 이 안내의 버전과 선택한 호스트를 아래처럼 명시 인자와 비대화형 옵션으로 전달합니다. 기본 대화형 화면을 임의로 조작해 사용자 선택을 대신하지 않습니다.
4. 설치 결과의 실패 항목, 선택한 실행 경로와 버전, 플러그인·서비스 진단을 확인합니다.
5. Codex의 훅 신뢰와 필요한 로그인은 사용자가 수행하도록 안내합니다. 신뢰 상태나 전역 승인·샌드박스 설정을 대신 변경하지 않습니다.
6. 사용자가 신뢰·재로드를 마치면 실제 Work 조회와 Knowledge 검색, 새 세션의 관측을 확인합니다. 설치 성공·구성 준비·신뢰 대기·실제 사용 확인을 구분해 보고합니다.

```sh
# Codex 연결을 사용자가 선택한 경우
curl -fsSL https://orbit.onidot.com/install.sh | bash -s -- --version 0.2.1 --host codex --non-interactive --json

# 호스트 연결 없이 실행물만 설치
curl -fsSL https://orbit.onidot.com/install.sh | bash -s -- --version 0.2.1 --runtime-only --non-interactive --json

# 특정 버전을 선택
curl -fsSL https://orbit.onidot.com/install.sh | bash -s -- --version 0.2.1 --host codex --non-interactive
```

호스트가 늘어나면 `--host`를 반복해 선택할 수 있습니다. 이 버전에서 지원되지 않는 호스트를 지정하면 등록을 거부합니다. 비대화형 실행에서 호스트 또는 런타임 전용 선택을 빠뜨리면 임의 선택 없이 중단합니다. 후보 버전은 `--channel preview`로 명시하며 안정판이 없다고 후보를 자동 설치하지 않습니다.

## Codex에서 사용 시작

Codex의 `/hooks` 또는 앱 훅 설정에서 `onidot-orbit` 출처의 훅을 검토·신뢰합니다. 플러그인·역할·MCP가 새로 로드되도록 세션을 다시 열고 실제 도구를 호출합니다. 훅 정의가 바뀌면 신뢰 검토가 다시 필요할 수 있습니다.

```sh
"$HOME/.local/bin/oni" version
"$HOME/.local/bin/oni" plugin doctor --json
"$HOME/.local/bin/oni" runtime status --json
"$HOME/.local/bin/oni" runtime open
```

`configurationReady`는 설정 준비를 뜻합니다. 사용자 신뢰와 새 호스트 실행이 확인되기 전까지 모든 기능이 준비됐다고 보고하지 않습니다. 자동 작업 재개 지원은 실제 runtime control 상태로 판단하며 수동 관측을 자동 제어로 해석하지 않습니다.

## 갱신·충돌·복구

실행 중인 Orbit은 먼저 정상 정지합니다. 프로그램 갱신과 호스트 연결 갱신은 구분합니다.

```sh
oni runtime stop
oni update --version 0.2.1 --yes
oni setup --host codex --json
oni plugin doctor --json
```

기존 사용자 구성에 등록된 Orbit 관측 훅을 플러그인으로 옮길 때만 `oni setup --host codex --migrate-user-hooks --json`을 명시합니다. 사용자 수정본·소유 불명 실행물·충돌 설정은 보존하며 강제로 덮어쓰지 않습니다. 설치가 중단되면 출력된 원인을 먼저 확인하고 원본·백업·설치 기록을 삭제하지 않습니다.

```sh
oni plugin status --json
oni plugin repair --json
bash "$HOME/.local/lib/onidot/orbit/install.sh" --status --prefix "$HOME/.local"
bash "$HOME/.local/lib/onidot/orbit/install.sh" --recover --prefix "$HOME/.local"
```

설치 경로 자체가 손상됐으면 같은 버전의 검증된 외부 설치기를 사용합니다. 데이터 복원·서비스 삭제는 프로그램 재설치에 포함하지 않습니다. 기존 Studio 연결과 다른 플러그인도 보존합니다.

## 원본과 버전

- 홈페이지: https://orbit.onidot.com
- 릴리스: https://github.com/onidot-labs/onidot-orbit-releases/releases
- 이 버전의 안내: https://orbit.onidot.com/releases/0.2.1/install.md

배포 안내는 로그인 없이 읽을 수 있습니다. Work·Knowledge 데이터는 이 홈페이지로 전송되지 않습니다. 원격 서버 연결과 쓰기 목적지는 사용자가 선택한 별도 설정을 따릅니다.
