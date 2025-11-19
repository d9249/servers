# mcp-server-git: A git MCP server

## Overview

Git repository 상호작용 및 자동화를 위한 Model Context Protocol server입니다. 이 server는 Large Language Model을 통해 Git repository를 읽고, 검색하고, 조작할 수 있는 tool을 제공합니다.

mcp-server-git는 현재 초기 개발 단계에 있습니다. 기능과 사용 가능한 tool은 계속 개발하고 개선함에 따라 변경되고 확장될 수 있습니다.

### Tools

1. `git_status`
   - 작업 트리 상태를 표시합니다
   - Input:
     - `repo_path` (string): Git repository 경로
   - Returns: 작업 디렉토리의 현재 상태를 텍스트 출력으로 반환

2. `git_diff_unstaged`
   - 아직 stage되지 않은 작업 디렉토리의 변경 사항을 표시합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `context_lines` (number, optional): 표시할 컨텍스트 줄 수(기본값: 3)
   - Returns: unstaged 변경 사항의 diff 출력

3. `git_diff_staged`
   - commit을 위해 stage된 변경 사항을 표시합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `context_lines` (number, optional): 표시할 컨텍스트 줄 수(기본값: 3)
   - Returns: staged 변경 사항의 diff 출력

4. `git_diff`
   - 브랜치 또는 commit 간의 차이점을 표시합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `target` (string): 비교할 대상 브랜치 또는 commit
     - `context_lines` (number, optional): 표시할 컨텍스트 줄 수(기본값: 3)
   - Returns: 현재 상태와 대상을 비교하는 diff 출력

5. `git_commit`
   - repository에 변경 사항을 기록합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `message` (string): Commit 메시지
   - Returns: 새 commit hash와 함께 확인

6. `git_add`
   - staging 영역에 파일 내용을 추가합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `files` (string[]): stage할 파일 경로 배열
   - Returns: staged 파일 확인

7. `git_reset`
   - 모든 staged 변경 사항을 unstage합니다
   - Input:
     - `repo_path` (string): Git repository 경로
   - Returns: reset 작업 확인

8. `git_log`
   - 선택적 날짜 필터링과 함께 commit 로그를 표시합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `max_count` (number, optional): 표시할 최대 commit 수(기본값: 10)
     - `start_timestamp` (string, optional): commit 필터링을 위한 시작 타임스탬프. ISO 8601 형식(예: '2024-01-15T14:30:25'), 상대 날짜(예: '2 weeks ago', 'yesterday') 또는 절대 날짜(예: '2024-01-15', 'Jan 15 2024')를 허용합니다
     - `end_timestamp` (string, optional): commit 필터링을 위한 종료 타임스탬프. ISO 8601 형식(예: '2024-01-15T14:30:25'), 상대 날짜(예: '2 weeks ago', 'yesterday') 또는 절대 날짜(예: '2024-01-15', 'Jan 15 2024')를 허용합니다
   - Returns: hash, 작성자, 날짜 및 메시지가 포함된 commit 항목 배열

9. `git_create_branch`
   - 새 브랜치를 생성합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `branch_name` (string): 새 브랜치 이름
     - `base_branch` (string, optional): 생성할 기본 브랜치(기본값은 현재 브랜치)
   - Returns: 브랜치 생성 확인
10. `git_checkout`
   - 브랜치를 전환합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `branch_name` (string): checkout할 브랜치 이름
   - Returns: 브랜치 전환 확인
11. `git_show`
   - commit의 내용을 표시합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `revision` (string): 표시할 revision(commit hash, 브랜치 이름, 태그)
   - Returns: 지정된 commit의 내용

12. `git_branch`
   - Git 브랜치를 나열합니다
   - Inputs:
     - `repo_path` (string): Git repository 경로
     - `branch_type` (string): 로컬 브랜치('local'), 원격 브랜치('remote') 또는 모든 브랜치('all')를 나열할지 여부
     - `contains` (string, optional): 브랜치가 포함해야 하는 commit sha. commit sha가 지정되지 않은 경우 이 param에 아무것도 전달하지 마세요
     - `not_contains` (string, optional): 브랜치가 포함하지 않아야 하는 commit sha. commit sha가 지정되지 않은 경우 이 param에 아무것도 전달하지 마세요
   - Returns: 브랜치 목록

## Installation

### Using uv (권장)

[`uv`](https://docs.astral.sh/uv/)를 사용하는 경우 특별한 설치가 필요하지 않습니다. [`uvx`](https://docs.astral.sh/uv/guides/tools/)를 사용하여 *mcp-server-git*를 직접 실행합니다.

### Using PIP

또는 pip를 통해 `mcp-server-git`를 설치할 수 있습니다:

```
pip install mcp-server-git
```

설치 후 다음과 같이 script로 실행할 수 있습니다:

```
python -m mcp_server_git
```

## Configuration

### Usage with Claude Desktop

`claude_desktop_config.json`에 다음을 추가하세요:

<details>
<summary>Using uvx</summary>

```json
"mcpServers": {
  "git": {
    "command": "uvx",
    "args": ["mcp-server-git", "--repository", "path/to/git/repo"]
  }
}
```
</details>

<details>
<summary>Using docker</summary>

* 참고: '/Users/username'을 이 tool에서 액세스할 수 있도록 하려는 경로로 바꾸세요

```json
"mcpServers": {
  "git": {
    "command": "docker",
    "args": ["run", "--rm", "-i", "--mount", "type=bind,src=/Users/username,dst=/Users/username", "mcp/git"]
  }
}
```
</details>

<details>
<summary>Using pip installation</summary>

```json
"mcpServers": {
  "git": {
    "command": "python",
    "args": ["-m", "mcp_server_git", "--repository", "path/to/git/repo"]
  }
}
```
</details>

### Usage with VS Code

빠른 설치를 위해 아래의 원클릭 설치 버튼 중 하나를 사용하세요...

[![Install with UV in VS Code](https://img.shields.io/badge/VS_Code-UV-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=git&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-git%22%5D%7D) [![Install with UV in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-UV-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=git&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-git%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=git&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22--rm%22%2C%22-i%22%2C%22--mount%22%2C%22type%3Dbind%2Csrc%3D%24%7BworkspaceFolder%7D%2Cdst%3D%2Fworkspace%22%2C%22mcp%2Fgit%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=git&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22--rm%22%2C%22-i%22%2C%22--mount%22%2C%22type%3Dbind%2Csrc%3D%24%7BworkspaceFolder%7D%2Cdst%3D%2Fworkspace%22%2C%22mcp%2Fgit%22%5D%7D&quality=insiders)

수동 설치의 경우 다음 방법 중 하나를 사용하여 MCP server를 구성할 수 있습니다:

**방법 1: 사용자 구성(권장)**
사용자 수준 MCP 구성 파일에 구성을 추가합니다. Command Palette(`Ctrl + Shift + P`)를 열고 `MCP: Open User Configuration`을 실행합니다. 그러면 server 구성을 추가할 수 있는 사용자 `mcp.json` 파일이 열립니다.

**방법 2: 작업 공간 구성**
또는 작업 공간의 `.vscode/mcp.json` 파일에 구성을 추가할 수 있습니다. 이렇게 하면 다른 사람과 구성을 공유할 수 있습니다.

> VS Code의 MCP 구성에 대한 자세한 내용은 [공식 VS Code MCP 문서](https://code.visualstudio.com/docs/copilot/mcp)를 참조하세요.

```json
{
  "servers": {
    "git": {
      "command": "uvx",
      "args": ["mcp-server-git"]
    }
  }
}
```

Docker 설치의 경우:

```json
{
  "mcp": {
    "servers": {
      "git": {
        "command": "docker",
        "args": [
          "run",
          "--rm",
          "-i",
          "--mount", "type=bind,src=${workspaceFolder},dst=/workspace",
          "mcp/git"
        ]
      }
    }
  }
}
```

### Usage with [Zed](https://github.com/zed-industries/zed)

Zed settings.json에 다음을 추가하세요:

<details>
<summary>Using uvx</summary>

```json
"context_servers": [
  "mcp-server-git": {
    "command": {
      "path": "uvx",
      "args": ["mcp-server-git"]
    }
  }
],
```
</details>

<details>
<summary>Using pip installation</summary>

```json
"context_servers": {
  "mcp-server-git": {
    "command": {
      "path": "python",
      "args": ["-m", "mcp_server_git"]
    }
  }
},
```
</details>

### Usage with [Zencoder](https://zencoder.ai)

1. Zencoder 메뉴(...)로 이동합니다
2. 드롭다운 메뉴에서 `Agent Tools`를 선택합니다
3. `Add Custom MCP`를 클릭합니다
4. 아래의 이름(예: git)과 server 구성을 추가하고 `Install` 버튼을 클릭해야 합니다

<details>
<summary>Using uvx</summary>

```json
{
    "command": "uvx",
    "args": ["mcp-server-git", "--repository", "path/to/git/repo"]
}
```
</details>

## Debugging

MCP inspector를 사용하여 server를 디버그할 수 있습니다. uvx 설치의 경우:

```
npx @modelcontextprotocol/inspector uvx mcp-server-git
```

또는 특정 디렉토리에 패키지를 설치했거나 개발 중인 경우:

```
cd path/to/servers/src/git
npx @modelcontextprotocol/inspector uv run mcp-server-git
```

`tail -n 20 -f ~/Library/Logs/Claude/mcp*.log`를 실행하면 server의 로그가 표시되며 문제를 디버그하는 데 도움이 될 수 있습니다.

## Development

로컬 개발을 하는 경우 변경 사항을 테스트하는 두 가지 방법이 있습니다:

1. MCP inspector를 실행하여 변경 사항을 테스트합니다. 실행 지침은 [Debugging](#debugging)을 참조하세요.

2. Claude 데스크톱 앱을 사용하여 테스트합니다. `claude_desktop_config.json`에 다음을 추가하세요:

### Docker

```json
{
  "mcpServers": {
    "git": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "--mount", "type=bind,src=/Users/username/Desktop,dst=/projects/Desktop",
        "--mount", "type=bind,src=/path/to/other/allowed/dir,dst=/projects/other/allowed/dir,ro",
        "--mount", "type=bind,src=/path/to/file.txt,dst=/projects/path/to/file.txt",
        "mcp/git"
      ]
    }
  }
}
```

### UVX
```json
{
"mcpServers": {
  "git": {
    "command": "uv",
    "args": [
      "--directory",
      "/<path to mcp-servers>/mcp-servers/src/git",
      "run",
      "mcp-server-git"
    ]
    }
  }
}
```

## Build

Docker build:

```bash
cd src/git
docker build -t mcp/git .
```

## License

이 MCP server는 MIT License에 따라 라이선스가 부여됩니다. 즉, MIT License의 조건에 따라 소프트웨어를 자유롭게 사용, 수정 및 배포할 수 있습니다. 자세한 내용은 프로젝트 repository의 LICENSE 파일을 참조하세요.
