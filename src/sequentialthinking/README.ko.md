# Sequential Thinking MCP Server

구조화된 사고 프로세스를 통해 동적이고 반성적인 문제 해결을 위한 tool을 제공하는 MCP server 구현입니다.

## Features

- 복잡한 문제를 관리 가능한 단계로 분해
- 이해가 깊어짐에 따라 생각을 수정하고 개선
- 대안적인 추론 경로로 분기
- 총 생각 수를 동적으로 조정
- 솔루션 가설 생성 및 검증

## Tool

### sequential_thinking

문제 해결 및 분석을 위한 자세한 단계별 사고 프로세스를 촉진합니다.

**Inputs:**
- `thought` (string): 현재 사고 단계
- `nextThoughtNeeded` (boolean): 다른 사고 단계가 필요한지 여부
- `thoughtNumber` (integer): 현재 생각 번호
- `totalThoughts` (integer): 필요한 총 생각 수 추정
- `isRevision` (boolean, optional): 이전 사고를 수정하는지 여부
- `revisesThought` (integer, optional): 재고되는 생각
- `branchFromThought` (integer, optional): 분기 지점 생각 번호
- `branchId` (string, optional): 분기 식별자
- `needsMoreThoughts` (boolean, optional): 더 많은 생각이 필요한지 여부

## Usage

Sequential Thinking tool은 다음과 같은 경우에 설계되었습니다:
- 복잡한 문제를 단계로 나누기
- 수정의 여지가 있는 계획 및 설계
- 방향 수정이 필요할 수 있는 분석
- 전체 범위가 처음에 명확하지 않을 수 있는 문제
- 여러 단계에서 컨텍스트를 유지해야 하는 작업
- 관련 없는 정보를 필터링해야 하는 상황

## Configuration

### Usage with Claude Desktop

`claude_desktop_config.json`에 다음을 추가하세요:

#### npx

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sequential-thinking"
      ]
    }
  }
}
```

#### docker

```json
{
  "mcpServers": {
    "sequentialthinking": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "mcp/sequentialthinking"
      ]
    }
  }
}
```

생각 정보의 로깅을 비활성화하려면 환경 변수 `DISABLE_THOUGHT_LOGGING`을 `true`로 설정하세요.
Comment

### Usage with VS Code

빠른 설치를 위해 아래의 설치 버튼 중 하나를 클릭하세요...

[![Install with NPX in VS Code](https://img.shields.io/badge/VS_Code-NPM-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=sequentialthinking&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-sequential-thinking%22%5D%7D) [![Install with NPX in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-NPM-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=sequentialthinking&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-sequential-thinking%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=sequentialthinking&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22--rm%22%2C%22-i%22%2C%22mcp%2Fsequentialthinking%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=sequentialthinking&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22--rm%22%2C%22-i%22%2C%22mcp%2Fsequentialthinking%22%5D%7D&quality=insiders)

수동 설치의 경우 다음 방법 중 하나를 사용하여 MCP server를 구성할 수 있습니다:

**방법 1: 사용자 구성(권장)**
사용자 수준 MCP 구성 파일에 구성을 추가합니다. Command Palette(`Ctrl + Shift + P`)를 열고 `MCP: Open User Configuration`을 실행합니다. 그러면 server 구성을 추가할 수 있는 사용자 `mcp.json` 파일이 열립니다.

**방법 2: 작업 공간 구성**
또는 작업 공간의 `.vscode/mcp.json` 파일에 구성을 추가할 수 있습니다. 이렇게 하면 다른 사람과 구성을 공유할 수 있습니다.

> VS Code의 MCP 구성에 대한 자세한 내용은 [공식 VS Code MCP 문서](https://code.visualstudio.com/docs/copilot/mcp)를 참조하세요.

NPX 설치의 경우:

```json
{
  "servers": {
    "sequential-thinking": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sequential-thinking"
      ]
    }
  }
}
```

Docker 설치의 경우:

```json
{
  "servers": {
    "sequential-thinking": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "mcp/sequentialthinking"
      ]
    }
  }
}
```

## Building

Docker:

```bash
docker build -t mcp/sequentialthinking -f src/sequentialthinking/Dockerfile .
```

## License

이 MCP server는 MIT License에 따라 라이선스가 부여됩니다. 즉, MIT License의 조건에 따라 소프트웨어를 자유롭게 사용, 수정 및 배포할 수 있습니다. 자세한 내용은 프로젝트 repository의 LICENSE 파일을 참조하세요.
