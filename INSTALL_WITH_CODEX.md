# 시사 Windows 0.3.2 설치와 Codex 연결 안내

## 사용하는 방법

Windows PC의 Codex에 이 파일을 첨부하거나, 파일을 저장한 폴더를 Codex에서 열고 다음과 같이 요청하세요.

> 이 Markdown 파일의 절차에 따라 내 Windows PC에 시사 0.3.2를 설치하고 Codex 연결까지 진행해줘. 파일 검증과 실행 확인도 해줘. 본인 계정 로그인은 내가 할게.

파일을 첨부하는 것만으로 프로그램이 실행되지는 않습니다. 위와 같이 설치를 요청해야 합니다. Codex의 다운로드·프로그램 실행 권한 승인이 뜨면 내용을 확인하고 승인하세요. 이 문서는 로컬 Windows에서 명령을 실행할 수 있는 Codex용입니다.

설치 대상은 **기존 캐릭터 그림·영상이 포함된 0.3.2 원본 설치본**입니다. 0.4.0이나 최신 버전으로 바꾸지 않습니다. 공유자의 할 일·대화·계정 정보는 포함하지 않습니다.

## Codex가 수행할 작업

사용자가 이 파일에 따른 설치를 요청했다면, 아래 절차를 실제로 수행하세요. 설명만 하고 끝내지 말고 다운로드, SHA-256 검사, 설치, 실행 확인을 이어서 진행합니다. 실행 환경의 권한 정책과 사용자의 추가 지시를 따릅니다.

### 1. 설치할 PC 확인

- 사용자가 쓰는 **로컬 Windows 10/11 64비트 PC**인지 확인합니다. WSL·컨테이너·원격 Linux에 설치하지 않습니다. 그 환경만 접근 가능하면 로컬 Windows에서 진행해야 한다고 안내합니다.
- 시사 자체 설치에는 Git·Node.js·Python·Claude Code 설치나 저장소 복제가 필요하지 않습니다. 아래 EXE만 받으면 됩니다. Codex 연결에 필요한 CLI는 5단계에서 별도로 확인합니다.
- 설치 폴더는 현재 사용자 기준 `%LOCALAPPDATA%\Shisa`, 데이터 폴더는 `%USERPROFILE%\.cy-agent`입니다. 특정 사람의 사용자 이름을 경로에 넣지 않습니다.
- 기존 설치 또는 데이터가 있으면 먼저 설치 버전과 실행 상태를 확인합니다. 같은 0.3.2가 이미 설치되어 있으면 재설치 대신 실행 상태를 확인합니다. 다른 버전이거나 버전 확인이 안 되면 덮어 설치하지 말고, 기존 데이터 백업과 교체 여부를 사용자와 확인합니다. 특히 0.4.0에서의 다운그레이드는 검증되지 않았습니다.
- 기존 데이터와 설정을 삭제하지 않습니다. 인증정보가 있을 수 있는 `config.json` 내용을 채팅이나 로그에 출력하지 않습니다.

### 2. 새 PC에 다운로드·검증·설치

아래 PowerShell은 **기존 시사 설치와 데이터가 없는 새 PC**용입니다. 기존 환경에서는 1단계에 따라 처리하세요. 이 문서를 작성하거나 검토하는 작업에서는 실행하지 말고, 사용자가 실제 설치를 요청한 PC에서만 실행합니다.

```powershell
$ErrorActionPreference = 'Stop'

if ([Environment]::OSVersion.Platform -ne [PlatformID]::Win32NT -or
    -not [Environment]::Is64BitOperatingSystem) {
    throw 'Windows 64비트 로컬 PC에서 실행해야 합니다.'
}

$shisaInstallRoot = Join-Path $env:LOCALAPPDATA 'Shisa'
$shisaDataRoot = Join-Path $env:USERPROFILE '.cy-agent'
if ((Test-Path -LiteralPath $shisaInstallRoot) -or
    (Test-Path -LiteralPath $shisaDataRoot) -or
    (Get-Process -Name Shisa -ErrorAction SilentlyContinue)) {
    throw '기존 시사 설치·데이터·프로세스가 있습니다. 기존 버전과 데이터 보존을 먼저 확인하세요.'
}

$shisaDownloadDir = Join-Path ([IO.Path]::GetTempPath()) ('Shisa-Install-' + [Guid]::NewGuid().ToString('N'))
New-Item -ItemType Directory -Path $shisaDownloadDir | Out-Null
$shisaInstaller = Join-Path $shisaDownloadDir 'Shisa-0.3.2-Setup.exe'
$shisaUrl = 'https://github.com/jangyunseo-ineeji/shisa-windows-share/releases/download/v0.3.2-share.1/Shisa-0.3.2-Setup.exe'
$shisaExpectedHash = '086f254aabc30b9a1b06f5ab10b03b5ee63ea145ad33967f636720e834e9c5f7'

[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
Invoke-WebRequest -Uri $shisaUrl -OutFile $shisaInstaller -UseBasicParsing -TimeoutSec 600
$shisaActualHash = (Get-FileHash -LiteralPath $shisaInstaller -Algorithm SHA256).Hash
if ($shisaActualHash -ne $shisaExpectedHash) {
    throw 'SHA-256 불일치: 설치 파일을 실행하지 않습니다. 다운로드 출처와 파일을 확인하세요.'
}
Write-Output 'SHA-256 일치: 원본 설치 파일 확인 완료'

$shisaSetup = Start-Process -FilePath $shisaInstaller -WindowStyle Hidden -PassThru -Wait
if ($shisaSetup.ExitCode -ne 0) {
    throw ('설치 프로그램 종료 코드: ' + $shisaSetup.ExitCode + '. 설치 상태와 오류를 확인하세요.')
}
Write-Output ('설치 프로그램 종료. 다음 단계에서 설치와 실행을 확인하세요. 설치 파일: ' + $shisaInstaller)
```

다운로드는 약 297MB입니다. 정상 다운로드가 진행되는 동안 기다립니다. 네트워크가 막혀 있으면 실패 원인을 확인하고 해당 환경에서 필요한 권한을 요청합니다. 다른 출처의 파일로 바꾸거나 해시 검사를 생략하지 않습니다.

일반적으로 사용자 계정에 설치되므로 관리자 권한은 필요하지 않습니다. 운영체제나 회사 보안 정책이 실행을 차단하면 보호 기능을 끄거나 정책을 우회하지 말고 사용자에게 필요한 조치를 설명합니다.

### 3. 설치·실행 확인

설치 프로그램이 끝난 것만으로 성공이라고 판단하지 않습니다. 설치가 마무리될 시간을 두고 아래를 확인하세요.

```powershell
$shisaApp = Join-Path $env:LOCALAPPDATA 'Shisa\app-0.3.2\Shisa.exe'
if (-not (Test-Path -LiteralPath $shisaApp)) {
    throw '0.3.2 실행 파일을 아직 찾지 못했습니다. 설치 진행 상태와 오류를 확인하세요.'
}

Get-Item -LiteralPath $shisaApp |
    Select-Object FullName, @{Name='ProductVersion'; Expression={$_.VersionInfo.ProductVersion}}

$shisaRunning = Get-Process -Name Shisa -ErrorAction SilentlyContinue |
    Where-Object { $_.Path -eq $shisaApp }
if (-not $shisaRunning) {
    Start-Process -FilePath $shisaApp -WindowStyle Hidden | Out-Null
}
```

실행 요청 후 해당 설치 경로의 `Shisa.exe` 프로세스가 실행 중인지 다시 확인합니다. 프로세스가 종료되면 원인을 확인하고, 실제로 확인한 상태만 보고합니다. 화면 확인 도구가 있으면 캐릭터 또는 첫 설정 화면까지 확인하고, 없으면 화면은 직접 확인하지 못했다고 알립니다.

완료 보고에는 설치 버전, 파일 해시 검사 결과, 설치 위치, 프로세스 실행 여부와 남은 사용자 설정을 짧게 적습니다.

### 4. 처음 사용하는 사람에게 안내

- 첫 화면에서 이름·근무 시간 등 개인 설정을 합니다. AI 없이도 기본 할 일·알림 기능을 사용할 수 있습니다.
- 대화 기능은 아래 5단계에 따라 **본인의 Codex**를 연결합니다. 사용자가 시사 설치만 요청했으면 연결 안내만 하고, Codex 연결까지 요청했으면 CLI 확인·필요한 설치·시사 연결 확인까지 진행합니다. Slack·메일·Claude Code 연결은 이번 절차에 포함되지 않습니다. 계정 로그인은 사용자가 직접 합니다.
- 다음부터는 시작 메뉴의 **시사**로 실행합니다. 캐릭터 두 번 클릭으로 대화를 열 수 있습니다.
- 0.3.2의 그림을 유지하려는 목적이므로 새 버전 알림이 나와도 이 설치 작업에서 업데이트하지 않습니다.

### 5. 시사에서 Codex 연결하기

시사 0.3.2는 **같은 Windows PC에서 실행되는 Codex CLI**를 찾아 사용합니다. VS Code 확장이나 Codex 앱을 쓰고 있어도, Windows PowerShell에서 `codex` 명령이 실행되는지 별도로 확인해야 합니다. 기존 VS Code 대화방에 시사가 접속하거나 그 대화 전체를 자동으로 이어받는 방식은 아닙니다.

#### 5-1. CLI가 이미 있는지 확인

시사를 쓰는 것과 같은 Windows 사용자 계정으로 새 PowerShell을 열어 실행합니다.

```powershell
Get-Command codex -ErrorAction SilentlyContinue
codex --version
codex login status
```

버전과 로그인 상태가 정상이라면 재설치하거나 로그아웃하지 말고 5-3으로 넘어갑니다. 명령을 찾지 못하면 5-2, 설치는 되어 있고 로그인만 안 되어 있으면 아래 로그인 단계로 넘어갑니다. WSL 안에만 설치된 CLI는 이 Windows 앱의 연결 확인을 대신하지 못합니다.

#### 5-2. 없는 경우 CLI 설치와 본인 로그인

**설치 방법은 다음 중 하나만 사용합니다.** 정상 작동하는 기존 CLI는 그대로 사용합니다.

- `npm.cmd --version`이 동작하는 PC: 시사 0.3.2 화면에 나오는 것과 같은 npm 설치 방법입니다.

  ```powershell
  npm.cmd install -g @openai/codex
  ```

- npm이 없는 PC: OpenAI가 제공하는 Windows 독립 설치 방법입니다. 새 PowerShell에서 실행합니다.

  ```powershell
  powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
  ```

이 명령은 공식 OpenAI 설치 스크립트를 내려받아 실행합니다. 회사 정책이 차단하면 우회하지 않고 담당자에게 확인합니다. 설치 방법의 최신 원문은 [OpenAI Codex CLI 안내](https://learn.chatgpt.com/docs/codex/cli)에서 확인할 수 있습니다.

설치 후 PowerShell을 새로 열어 다음을 실행합니다.

```powershell
codex --version
codex login
```

브라우저가 열리면 **받는 사람 본인의 ChatGPT 계정**으로 로그인합니다. 브라우저가 자동으로 열리지 않으면 터미널에 표시된 로그인 주소를 직접 엽니다. Codex를 사용할 수 있는 계정·워크스페이스여야 하며, 현재 이용 가능 여부는 로그인 결과로 확인합니다.

Codex가 설치 작업을 대신 진행 중이라면 이 로그인 단계에서 사용자가 로그인할 수 있도록 안내하고 기다립니다. 비밀번호·인증 코드·토큰을 채팅에 요청하거나 다른 사람의 로그인 파일을 복사하지 않습니다. 로그인 완료 후 아래 명령으로 확인합니다.

```powershell
codex login status
```

로그인 및 상태 확인 명령은 [OpenAI 인증 안내](https://learn.chatgpt.com/docs/auth)를 따릅니다. 이 안내는 ChatGPT 계정 로그인을 기준으로 하며, 별도 과금이 가능한 API 키 방식으로 임의 전환하지 않습니다.

#### 5-3. 시사 화면에서 연결

1. Windows 알림 영역(시계 옆)의 시사를 **종료**한 후 시작 메뉴에서 다시 실행합니다. 새 CLI 경로와 로그인 상태를 읽게 하기 위한 단계입니다.
2. 첫 설치 화면의 **「뇌(AI)를 연결할까요?」**에서 **Codex**를 선택합니다. 이미 설정을 마쳤다면 패널의 **「설치」** 탭 또는 설정(⚙)의 **「설치 다시 하기」**로 해당 화면을 엽니다. 이는 설정 화면을 다시 여는 것이며 앱 제거·데이터 삭제를 뜻하지 않습니다.
3. 로그인 확인 또는 완료 표시가 나오면 **「다음」**으로 진행합니다. 패널 **「모델」** 탭에 Codex의 **「로그인 확인됨」 → 「연결」**이 보이면 그 버튼으로도 연결할 수 있습니다.
4. 모델 선택란에서는 **본인 계정에 표시되는 모델**을 고릅니다. 다른 사람의 모델 이름을 무조건 입력하지 않습니다. 모델 목록이 비어 있으면 PowerShell에서 `codex`를 한 번 열어 초기화를 마치고 종료한 뒤 시사를 다시 실행해 봅니다.
5. 캐릭터를 두 번 클릭하여 채팅을 열고 **「안녕, 한 문장으로 대답해줘」**라고 입력합니다. 실제 답변을 받아야 대화 연결까지 확인한 것입니다. 설치 확인과 로그인 확인만 끝났다면 그 상태를 구분해 보고합니다.

위 메뉴와 연결 동작은 공유 중인 0.3.2 [설정 화면 코드](https://github.com/jangyunseo-ineeji/shisa-windows-share/blob/main/desktop/src/setup-flow.js), [모델 화면 코드](https://github.com/jangyunseo-ineeji/shisa-windows-share/blob/main/desktop/renderer/models-view.js)를 확인한 안내입니다. 새 CLI와 각 PC의 연결 성공까지 미리 보장하는 것은 아니므로 마지막 대화 확인을 수행합니다.

#### 연결되지 않을 때

| 증상 | 확인할 내용 |
|---|---|
| `codex` 명령을 찾지 못함 | 설치 후 PowerShell을 새로 열었는지 확인합니다. `Get-Command codex -All`로 경로를 확인하고 Windows 쪽 설치인지 확인합니다. |
| `codex.ps1` 실행 정책 오류 | npm 설치본이면 `codex.cmd --version`, `codex.cmd login`을 사용해 봅니다. 시스템 실행 정책을 영구적으로 완화하지 않습니다. |
| CLI는 로그인됐지만 시사에서 찾지 못함 | 시사를 완전히 종료·재실행합니다. 같은 Windows 사용자인지, `CODEX_HOME`을 별도로 쓴 경우 시사도 같은 환경을 받는지 확인합니다. 인증 파일의 내용은 출력하지 않습니다. |
| 로그인은 되지만 모델 오류 | CLI와 시사에서 본인 계정에 실제 제공되는 모델을 선택합니다. 0.3.2와 새 CLI의 호환 문제 가능성도 있으므로 오류를 확인하고, 자동으로 시사를 0.4.0으로 교체하지 않습니다. |
| 사용량·권한 오류 | 본인 계정의 Codex 사용 가능 여부와 사용량을 확인합니다. 다른 사람의 계정·토큰을 사용하지 않습니다. |

## 직접 설치하는 경우

Codex 없이도 [설치 파일 받기](https://github.com/jangyunseo-ineeji/shisa-windows-share/releases/download/v0.3.2-share.1/Shisa-0.3.2-Setup.exe)를 눌러 다운로드한 뒤 실행할 수 있습니다. 이미 시사를 쓰고 있다면 먼저 위의 기존 설치 확인 사항을 확인하세요.

## 출처와 이용 안내

- 공유 저장소: https://github.com/jangyunseo-ineeji/shisa-windows-share
- 고정 릴리스: https://github.com/jangyunseo-ineeji/shisa-windows-share/releases/tag/v0.3.2-share.1
- 원작 프로젝트: https://github.com/cheo0y/shisa-agent
- 원본 설치 파일 SHA-256: `086f254aabc30b9a1b06f5ab10b03b5ee63ea145ad33967f636720e834e9c5f7`
- 코드의 MIT 라이선스와 캐릭터의 별도 권리·비상업적 이용 안내는 [LICENSE](https://github.com/jangyunseo-ineeji/shisa-windows-share/blob/main/LICENSE), 외부 라이브러리 고지는 [THIRD_PARTY_NOTICES](https://github.com/jangyunseo-ineeji/shisa-windows-share/blob/main/THIRD_PARTY_NOTICES.md)를 확인하세요.

이 파일은 설치 지침입니다. Codex의 실행 권한, Windows 보안 확인, 본인 계정 로그인까지 자동으로 승인하는 파일은 아닙니다.
