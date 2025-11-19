# Filesystem MCP Server

파일 시스템 작업을 위한 Model Context Protocol (MCP)을 구현하는 Node.js server입니다.

## Features

- 파일 읽기/쓰기
- 디렉토리 생성/목록/삭제
- 파일/디렉토리 이동
- 파일 검색
- 파일 메타데이터 가져오기
- [Roots](https://modelcontextprotocol.io/docs/learn/client-concepts#roots)를 통한 동적 디렉토리 액세스 제어

## Directory Access Control

server는 유연한 디렉토리 액세스 제어 시스템을 사용합니다. 디렉토리는 명령줄 인수를 통해 또는 [Roots](https://modelcontextprotocol.io/docs/learn/client-concepts#roots)를 통해 동적으로 지정할 수 있습니다.

### Method 1: Command-line Arguments
server 시작 시 허용된 디렉토리를 지정합니다:
```bash
mcp-server-filesystem /path/to/dir1 /path/to/dir2
```

### Method 2: MCP Roots (권장)
[Roots](https://modelcontextprotocol.io/docs/learn/client-concepts#roots)를 지원하는 MCP client는 허용된 디렉토리를 동적으로 업데이트할 수 있습니다.

Client에서 Server로 알림된 Root는 제공된 경우 server 측 허용 디렉토리를 완전히 대체합니다.

**중요**: server가 명령줄 인수 없이 시작되고 client가 root protocol을 지원하지 않거나(또는 빈 root를 제공하는 경우) server는 초기화 중에 오류를 발생시킵니다.

이 방법은 server를 재시작하지 않고도 `roots/list_changed` 알림을 통해 런타임 디렉토리 업데이트를 가능하게 하여 보다 유연하고 현대적인 통합 경험을 제공하므로 권장됩니다.

### How It Works

server의 디렉토리 액세스 제어는 다음 흐름을 따릅니다:

1. **Server Startup**
   - Server는 명령줄 인수의 디렉토리로 시작합니다(제공된 경우)
   - 인수가 제공되지 않으면 server는 빈 허용 디렉토리로 시작합니다

2. **Client Connection & Initialization**
   - Client가 연결되고 기능과 함께 `initialize` 요청을 보냅니다
   - Server는 client가 root protocol(`capabilities.roots`)을 지원하는지 확인합니다

3. **Roots Protocol Handling** (client가 root를 지원하는 경우)
   - **초기화 시**: Server가 `roots/list`를 통해 client에서 root를 요청합니다
   - Client가 구성된 root로 응답합니다
   - Server가 client의 root로 모든 허용 디렉토리를 대체합니다
   - **런타임 업데이트 시**: Client가 `notifications/roots/list_changed`를 보낼 수 있습니다
   - Server가 업데이트된 root를 요청하고 허용 디렉토리를 다시 대체합니다

4. **Fallback Behavior** (client가 root를 지원하지 않는 경우)
   - Server는 명령줄 디렉토리만 계속 사용합니다
   - 동적 업데이트가 불가능합니다

5. **Access Control**
   - 모든 파일 시스템 작업은 허용된 디렉토리로 제한됩니다
   - 현재 디렉토리를 보려면 `list_allowed_directories` tool을 사용하세요
   - Server는 작동하려면 최소 하나의 허용된 디렉토리가 필요합니다

**참고**: server는 `args` 또는 Roots를 통해 지정된 디렉토리 내에서만 작업을 허용합니다.



## API

### Tools

- **read_text_file**
  - 파일의 전체 내용을 텍스트로 읽습니다
  - Inputs:
    - `path` (string)
    - `head` (number, optional): 처음 N줄
    - `tail` (number, optional): 마지막 N줄
  - 확장자에 관계없이 항상 파일을 UTF-8 텍스트로 처리합니다
  - `head`와 `tail`을 동시에 지정할 수 없습니다

- **read_media_file**
  - 이미지 또는 오디오 파일을 읽습니다
  - Inputs:
    - `path` (string)
  - 파일을 스트리밍하고 해당 MIME 타입과 함께 base64 데이터를 반환합니다

- **read_multiple_files**
  - 여러 파일을 동시에 읽습니다
  - Input: `paths` (string[])
  - 읽기 실패가 전체 작업을 중단하지 않습니다

- **write_file**
  - 새 파일 생성 또는 기존 파일 덮어쓰기(주의하여 사용)
  - Inputs:
    - `path` (string): 파일 위치
    - `content` (string): 파일 내용

- **edit_file**
  - 고급 패턴 매칭 및 서식 지정을 사용하여 선택적 편집을 수행합니다
  - Features:
    - 줄 기반 및 다중 줄 콘텐츠 매칭
    - 들여쓰기 보존과 함께 공백 정규화
    - 올바른 위치 지정으로 여러 동시 편집
    - 들여쓰기 스타일 감지 및 보존
    - 컨텍스트가 있는 Git 스타일 diff 출력
    - 드라이 런 모드로 변경 사항 미리보기
  - Inputs:
    - `path` (string): 편집할 파일
    - `edits` (array): 편집 작업 목록
      - `oldText` (string): 검색할 텍스트(하위 문자열 가능)
      - `newText` (string): 교체할 텍스트
    - `dryRun` (boolean): 적용하지 않고 변경 사항 미리보기(기본값: false)
  - 드라이 런의 경우 자세한 diff 및 매치 정보를 반환하고, 그렇지 않으면 변경 사항을 적용합니다
  - Best Practice: 적용하기 전에 항상 dryRun을 먼저 사용하여 변경 사항을 미리 봅니다

- **create_directory**
  - 새 디렉토리 생성 또는 존재 확인
  - Input: `path` (string)
  - 필요한 경우 상위 디렉토리를 생성합니다
  - 디렉토리가 존재하면 자동으로 성공합니다

- **list_directory**
  - [FILE] 또는 [DIR] 접두사와 함께 디렉토리 내용을 나열합니다
  - Input: `path` (string)

- **list_directory_with_sizes**
  - 파일 크기를 포함하여 [FILE] 또는 [DIR] 접두사와 함께 디렉토리 내용을 나열합니다
  - Inputs:
    - `path` (string): 나열할 디렉토리 경로
    - `sortBy` (string, optional): "name" 또는 "size"로 항목 정렬(기본값: "name")
  - 파일 크기 및 요약 통계가 포함된 자세한 목록을 반환합니다
  - 총 파일, 디렉토리 및 결합된 크기를 보여줍니다

- **move_file**
  - 파일 및 디렉토리 이동 또는 이름 변경
  - Inputs:
    - `source` (string)
    - `destination` (string)
  - 대상이 존재하면 실패합니다

- **search_files**
  - 패턴과 일치하거나 일치하지 않는 파일/디렉토리를 재귀적으로 검색합니다
  - Inputs:
    - `path` (string): 시작 디렉토리
    - `pattern` (string): 검색 패턴
    - `excludePatterns` (string[]): 제외할 패턴
  - Glob 스타일 패턴 매칭
  - 일치하는 항목의 전체 경로를 반환합니다

- **directory_tree**
  - 디렉토리 내용의 재귀적 JSON 트리 구조를 가져옵니다
  - Inputs:
    - `path` (string): 시작 디렉토리
    - `excludePatterns` (string[]): 제외할 패턴. Glob 형식이 지원됩니다.
  - Returns:
    - 각 항목에 다음이 포함된 JSON 배열:
      - `name` (string): 파일/디렉토리 이름
      - `type` ('file'|'directory'): 항목 유형
      - `children` (array): 디렉토리에만 존재
        - 빈 디렉토리의 경우 빈 배열
        - 파일의 경우 생략됨
  - 출력은 가독성을 위해 2칸 들여쓰기로 형식화됩니다

- **get_file_info**
  - 자세한 파일/디렉토리 메타데이터를 가져옵니다
  - Input: `path` (string)
  - Returns:
    - 크기
    - 생성 시간
    - 수정 시간
    - 액세스 시간
    - 유형(파일/디렉토리)
    - 권한

- **list_allowed_directories**
  - server가 액세스할 수 있는 모든 디렉토리를 나열합니다
  - 입력 필요 없음
  - Returns:
    - 이 server가 읽기/쓰기할 수 있는 디렉토리

## Usage with Claude Desktop
`claude_desktop_config.json`에 다음을 추가하세요:

참고: `/projects`에 마운트하여 server에 샌드박스 디렉토리를 제공할 수 있습니다. `ro` 플래그를 추가하면 server에서 디렉토리가 읽기 전용이 됩니다.

### Docker
참고: 기본적으로 모든 디렉토리는 `/projects`에 마운트되어야 합니다.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--mount", "type=bind,src=/Users/username/Desktop,dst=/projects/Desktop",
        "--mount", "type=bind,src=/path/to/other/allowed/dir,dst=/projects/other/allowed/dir,ro",
        "--mount", "type=bind,src=/path/to/file.txt,dst=/projects/path/to/file.txt",
        "mcp/filesystem",
        "/projects"
      ]
    }
  }
}
```

### NPX

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop",
        "/path/to/other/allowed/dir"
      ]
    }
  }
}
```

## Usage with VS Code

빠른 설치를 위해 아래의 설치 버튼을 클릭하세요...

[![Install with NPX in VS Code](https://img.shields.io/badge/VS_Code-NPM-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=filesystem&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-filesystem%22%2C%22%24%7BworkspaceFolder%7D%22%5D%7D) [![Install with NPX in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-NPM-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=filesystem&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-filesystem%22%2C%22%24%7BworkspaceFolder%7D%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=filesystem&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22--mount%22%2C%22type%3Dbind%2Csrc%3D%24%7BworkspaceFolder%7D%2Cdst%3D%2Fprojects%2Fworkspace%22%2C%22mcp%2Ffilesystem%22%2C%22%2Fprojects%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=filesystem&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22--mount%22%2C%22type%3Dbind%2Csrc%3D%24%7BworkspaceFolder%7D%2Cdst%3D%2Fprojects%2Fworkspace%22%2C%22mcp%2Ffilesystem%22%2C%22%2Fprojects%22%5D%7D&quality=insiders)

수동 설치의 경우 다음 방법 중 하나를 사용하여 MCP server를 구성할 수 있습니다:

**방법 1: 사용자 구성(권장)**
사용자 수준 MCP 구성 파일에 구성을 추가합니다. Command Palette(`Ctrl + Shift + P`)를 열고 `MCP: Open User Configuration`을 실행합니다. 그러면 server 구성을 추가할 수 있는 사용자 `mcp.json` 파일이 열립니다.

**방법 2: 작업 공간 구성**
또는 작업 공간의 `.vscode/mcp.json` 파일에 구성을 추가할 수 있습니다. 이렇게 하면 다른 사람과 구성을 공유할 수 있습니다.

> VS Code의 MCP 구성에 대한 자세한 내용은 [공식 VS Code MCP 문서](https://code.visualstudio.com/docs/copilot/mcp)를 참조하세요.

`/projects`에 마운트하여 server에 샌드박스 디렉토리를 제공할 수 있습니다. `ro` 플래그를 추가하면 server에서 디렉토리가 읽기 전용이 됩니다.

### Docker
참고: 기본적으로 모든 디렉토리는 `/projects`에 마운트되어야 합니다.

```json
{
  "servers": {
    "filesystem": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--mount", "type=bind,src=${workspaceFolder},dst=/projects/workspace",
        "mcp/filesystem",
        "/projects"
      ]
    }
  }
}
```

### NPX

```json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "${workspaceFolder}"
      ]
    }
  }
}
```

## Build

Docker build:

```bash
docker build -t mcp/filesystem -f src/filesystem/Dockerfile .
```

## License

이 MCP server는 MIT License에 따라 라이선스가 부여됩니다. 즉, MIT License의 조건에 따라 소프트웨어를 자유롭게 사용, 수정 및 배포할 수 있습니다. 자세한 내용은 프로젝트 repository의 LICENSE 파일을 참조하세요.
