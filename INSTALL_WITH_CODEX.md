# 시사 Windows 0.3.2 설치 안내 — Codex에 전달하는 파일

## 사용하는 방법

Windows PC의 Codex에 이 파일을 첨부하거나, 파일을 저장한 폴더를 Codex에서 열고 다음과 같이 요청하세요.

> 이 Markdown 파일의 절차에 따라 내 Windows PC에 시사 0.3.2를 다운로드하고 설치해줘. 파일 검증과 실행 확인까지 진행해줘.

파일을 첨부하는 것만으로 프로그램이 실행되지는 않습니다. 위와 같이 설치를 요청해야 합니다. Codex의 다운로드·프로그램 실행 권한 승인이 뜨면 내용을 확인하고 승인하세요. 이 문서는 로컬 Windows에서 명령을 실행할 수 있는 Codex용입니다.

설치 대상은 **기존 캐릭터 그림·영상이 포함된 0.3.2 원본 설치본**입니다. 0.4.0이나 최신 버전으로 바꾸지 않습니다. 공유자의 할 일·대화·계정 정보는 포함하지 않습니다.

## Codex가 수행할 작업

사용자가 이 파일에 따른 설치를 요청했다면, 아래 절차를 실제로 수행하세요. 설명만 하고 끝내지 말고 다운로드, SHA-256 검사, 설치, 실행 확인을 이어서 진행합니다. 실행 환경의 권한 정책과 사용자의 추가 지시를 따릅니다.

### 1. 설치할 PC 확인

- 사용자가 쓰는 **로컬 Windows 10/11 64비트 PC**인지 확인합니다. WSL·컨테이너·원격 Linux에 설치하지 않습니다. 그 환경만 접근 가능하면 로컬 Windows에서 진행해야 한다고 안내합니다.
- Git·Node.js·Python·Claude Code 설치나 저장소 복제는 필요하지 않습니다. 아래 EXE만 받으면 됩니다.
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
- 대화 기능은 시사의 설정에서 **본인의 AI 계정 또는 API 키**를 연결해야 합니다. 이번 설치 요청만으로 AI 도구 추가 설치나 Slack·메일 연결까지 진행하지 않습니다. 계정 로그인이나 API 키 입력은 사용자가 직접 합니다.
- 다음부터는 시작 메뉴의 **시사**로 실행합니다. 캐릭터 두 번 클릭으로 대화를 열 수 있습니다.
- 0.3.2의 그림을 유지하려는 목적이므로 새 버전 알림이 나와도 이 설치 작업에서 업데이트하지 않습니다.

## 직접 설치하는 경우

Codex 없이도 [설치 파일 받기](https://github.com/jangyunseo-ineeji/shisa-windows-share/releases/download/v0.3.2-share.1/Shisa-0.3.2-Setup.exe)를 눌러 다운로드한 뒤 실행할 수 있습니다. 이미 시사를 쓰고 있다면 먼저 위의 기존 설치 확인 사항을 확인하세요.

## 출처와 이용 안내

- 공유 저장소: https://github.com/jangyunseo-ineeji/shisa-windows-share
- 고정 릴리스: https://github.com/jangyunseo-ineeji/shisa-windows-share/releases/tag/v0.3.2-share.1
- 원작 프로젝트: https://github.com/cheo0y/shisa-agent
- 원본 설치 파일 SHA-256: `086f254aabc30b9a1b06f5ab10b03b5ee63ea145ad33967f636720e834e9c5f7`
- 코드의 MIT 라이선스와 캐릭터의 별도 권리·비상업적 이용 안내는 [LICENSE](https://github.com/jangyunseo-ineeji/shisa-windows-share/blob/main/LICENSE), 외부 라이브러리 고지는 [THIRD_PARTY_NOTICES](https://github.com/jangyunseo-ineeji/shisa-windows-share/blob/main/THIRD_PARTY_NOTICES.md)를 확인하세요.

이 파일은 설치 지침입니다. Codex의 실행 권한, Windows 보안 확인, 본인 계정 로그인까지 자동으로 승인하는 파일은 아닙니다.
