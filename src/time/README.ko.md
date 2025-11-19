# Time MCP Server

시간 및 시간대 변환 기능을 제공하는 Model Context Protocol server입니다. 이 server를 사용하면 LLM이 현재 시간 정보를 얻고 자동 시스템 시간대 감지와 함께 IANA 시간대 이름을 사용하여 시간대 변환을 수행할 수 있습니다.

### Available Tools

- `get_current_time` - 특정 시간대 또는 시스템 시간대의 현재 시간을 가져옵니다.
  - Required arguments:
    - `timezone` (string): IANA 시간대 이름(예: 'America/New_York', 'Europe/London')

- `convert_time` - 시간대 간 시간을 변환합니다.
  - Required arguments:
    - `source_timezone` (string): 소스 IANA 시간대 이름
    - `time` (string): 24시간 형식의 시간(HH:MM)
    - `target_timezone` (string): 대상 IANA 시간대 이름

## Installation

### Using uv (권장)

[`uv`](https://docs.astral.sh/uv/)를 사용하는 경우 특별한 설치가 필요하지 않습니다. [`uvx`](https://docs.astral.sh/uv/guides/tools/)를 사용하여 *mcp-server-time*을 직접 실행합니다.

### Using PIP

또는 pip를 통해 `mcp-server-time`을 설치할 수 있습니다:

```bash
pip install mcp-server-time
```

설치 후 다음과 같이 script로 실행할 수 있습니다:

```bash
python -m mcp_server_time
```

## Configuration

### Configure for Claude.app

Claude 설정에 다음을 추가하세요:

<details>
<summary>Using uvx</summary>

```json
{
  "mcpServers": {
    "time": {
      "command": "uvx",
      "args": ["mcp-server-time"]
    }
  }
}
```
</details>

<details>
<summary>Using docker</summary>

```json
{
  "mcpServers": {
    "time": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "-e", "LOCAL_TIMEZONE", "mcp/time"]
    }
  }
}
```
</details>

<details>
<summary>Using pip installation</summary>

```json
{
  "mcpServers": {
    "time": {
      "command": "python",
      "args": ["-m", "mcp_server_time"]
    }
  }
}
```
</details>

### Configure for Zed

Zed settings.json에 다음을 추가하세요:

<details>
<summary>Using uvx</summary>

```json
"context_servers": [
  "mcp-server-time": {
    "command": "uvx",
    "args": ["mcp-server-time"]
  }
],
```
</details>

<details>
<summary>Using pip installation</summary>

```json
"context_servers": {
  "mcp-server-time": {
    "command": "python",
    "args": ["-m", "mcp_server_time"]
  }
},
```
</details>

### Configure for VS Code

빠른 설치를 위해 아래의 원클릭 설치 버튼 중 하나를 사용하세요...

[![Install with UV in VS Code](https://img.shields.io/badge/VS_Code-UV-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=time&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-time%22%5D%7D) [![Install with UV in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-UV-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=time&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-time%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=time&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Ftime%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=time&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Ftime%22%5D%7D&quality=insiders)

수동 설치의 경우 VS Code의 User Settings (JSON) 파일에 다음 JSON 블록을 추가합니다. `Ctrl + Shift + P`를 누르고 `Preferences: Open User Settings (JSON)`을 입력하면 됩니다.

선택적으로 작업 공간의 `.vscode/mcp.json` 파일에 추가할 수 있습니다. 이렇게 하면 다른 사람과 구성을 공유할 수 있습니다.

> `mcp.json` 파일을 사용할 때는 `mcp` 키가 필요합니다.

<details>
<summary>Using uvx</summary>

```json
{
  "mcp": {
    "servers": {
      "time": {
        "command": "uvx",
        "args": ["mcp-server-time"]
      }
    }
  }
}
```
</details>

<details>
<summary>Using Docker</summary>

```json
{
  "mcp": {
    "servers": {
      "time": {
        "command": "docker",
        "args": ["run", "-i", "--rm", "mcp/time"]
      }
    }
  }
}
```
</details>

### Configure for Zencoder

1. Zencoder 메뉴(...)로 이동합니다
2. 드롭다운 메뉴에서 `Agent Tools`를 선택합니다
3. `Add Custom MCP`를 클릭합니다
4. 아래의 이름과 server 구성을 추가하고 `Install` 버튼을 클릭해야 합니다

<details>
<summary>Using uvx</summary>

```json
{
    "command": "uvx",
    "args": ["mcp-server-time"]
  }
```
</details>

### Customization - System Timezone

기본적으로 server는 시스템의 시간대를 자동으로 감지합니다. 구성의 `args` 목록에 `--local-timezone` 인수를 추가하여 이를 재정의할 수 있습니다.

예제:
```json
{
  "command": "python",
  "args": ["-m", "mcp_server_time", "--local-timezone=America/New_York"]
}
```

## Example Interactions

1. 현재 시간 가져오기:
```json
{
  "name": "get_current_time",
  "arguments": {
    "timezone": "Europe/Warsaw"
  }
}
```
응답:
```json
{
  "timezone": "Europe/Warsaw",
  "datetime": "2024-01-01T13:00:00+01:00",
  "is_dst": false
}
```

2. 시간대 간 시간 변환:
```json
{
  "name": "convert_time",
  "arguments": {
    "source_timezone": "America/New_York",
    "time": "16:30",
    "target_timezone": "Asia/Tokyo"
  }
}
```
응답:
```json
{
  "source": {
    "timezone": "America/New_York",
    "datetime": "2024-01-01T12:30:00-05:00",
    "is_dst": false
  },
  "target": {
    "timezone": "Asia/Tokyo",
    "datetime": "2024-01-01T12:30:00+09:00",
    "is_dst": false
  },
  "time_difference": "+13.0h",
}
```

## Debugging

MCP inspector를 사용하여 server를 디버그할 수 있습니다. uvx 설치의 경우:

```bash
npx @modelcontextprotocol/inspector uvx mcp-server-time
```

또는 특정 디렉토리에 패키지를 설치했거나 개발 중인 경우:

```bash
cd path/to/servers/src/time
npx @modelcontextprotocol/inspector uv run mcp-server-time
```

## Examples of Questions for Claude

1. "지금 몇 시야?" (시스템 시간대 사용)
2. "도쿄는 지금 몇 시야?"
3. "뉴욕이 오후 4시일 때 런던은 몇 시야?"
4. "도쿄 시간 오전 9시 30분을 뉴욕 시간으로 변환해줘"

## Build

Docker build:

```bash
cd src/time
docker build -t mcp/time .
```

## Contributing

mcp-server-time을 확장하고 개선하는 데 도움을 주시기 바랍니다. 새로운 시간 관련 tool을 추가하거나 기존 기능을 향상시키거나 문서를 개선하고 싶으시다면 여러분의 의견은 소중합니다.

다른 MCP server 및 구현 패턴의 예는 다음을 참조하세요:
https://github.com/modelcontextprotocol/servers

Pull request를 환영합니다! mcp-server-time을 더욱 강력하고 유용하게 만들기 위해 새로운 아이디어, 버그 수정 또는 개선 사항을 자유롭게 기여해 주세요.

## License

mcp-server-time은 MIT License에 따라 라이선스가 부여됩니다. 즉, MIT License의 조건에 따라 소프트웨어를 자유롭게 사용, 수정 및 배포할 수 있습니다. 자세한 내용은 프로젝트 repository의 LICENSE 파일을 참조하세요.
