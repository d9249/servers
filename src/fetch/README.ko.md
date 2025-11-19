# Fetch MCP Server

웹 콘텐츠 가져오기 기능을 제공하는 Model Context Protocol server입니다. 이 server를 사용하면 LLM이 웹 페이지에서 콘텐츠를 검색하고 처리할 수 있으며, HTML을 쉽게 사용할 수 있도록 markdown으로 변환합니다.

> [!CAUTION]
> 이 server는 로컬/내부 IP 주소에 액세스할 수 있으며 보안 위험이 될 수 있습니다. 민감한 데이터가 노출되지 않도록 이 MCP server를 사용할 때 주의하세요.

fetch tool은 응답을 잘라내지만 `start_index` 인수를 사용하여 콘텐츠 추출을 시작할 위치를 지정할 수 있습니다. 이를 통해 model이 필요한 정보를 찾을 때까지 웹페이지를 청크 단위로 읽을 수 있습니다.

### Available Tools

- `fetch` - 인터넷에서 URL을 가져와 콘텐츠를 markdown으로 추출합니다.
    - `url` (string, required): 가져올 URL
    - `max_length` (integer, optional): 반환할 최대 문자 수(기본값: 5000)
    - `start_index` (integer, optional): 이 문자 인덱스부터 콘텐츠 시작(기본값: 0)
    - `raw` (boolean, optional): markdown 변환 없이 원시 콘텐츠 가져오기(기본값: false)

### Prompts

- **fetch**
  - URL을 가져와 콘텐츠를 markdown으로 추출합니다
  - Arguments:
    - `url` (string, required): 가져올 URL

## Installation

선택사항: node.js를 설치하면 fetch server가 더 강력한 다른 HTML simplifier를 사용합니다.

### Using uv (권장)

[`uv`](https://docs.astral.sh/uv/)를 사용하는 경우 특별한 설치가 필요하지 않습니다. [`uvx`](https://docs.astral.sh/uv/guides/tools/)를 사용하여 *mcp-server-fetch*를 직접 실행합니다.

### Using PIP

또는 pip를 통해 `mcp-server-fetch`를 설치할 수 있습니다:

```
pip install mcp-server-fetch
```

설치 후 다음과 같이 script로 실행할 수 있습니다:

```
python -m mcp_server_fetch
```

## Configuration

### Configure for Claude.app

Claude 설정에 다음을 추가하세요:

<details>
<summary>Using uvx</summary>

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
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
    "fetch": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "mcp/fetch"]
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
    "fetch": {
      "command": "python",
      "args": ["-m", "mcp_server_fetch"]
    }
  }
}
```
</details>

### Configure for VS Code

빠른 설치를 위해 아래의 원클릭 설치 버튼 중 하나를 사용하세요...

[![Install with UV in VS Code](https://img.shields.io/badge/VS_Code-UV-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=fetch&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-fetch%22%5D%7D) [![Install with UV in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-UV-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=fetch&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-fetch%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=fetch&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Ffetch%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=fetch&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Ffetch%22%5D%7D&quality=insiders)

수동 설치의 경우 VS Code의 User Settings (JSON) 파일에 다음 JSON 블록을 추가합니다. `Ctrl + Shift + P`를 누르고 `Preferences: Open User Settings (JSON)`을 입력하면 됩니다.

선택적으로 작업 공간의 `.vscode/mcp.json` 파일에 추가할 수 있습니다. 이렇게 하면 다른 사람과 구성을 공유할 수 있습니다.

> `mcp.json` 파일을 사용할 때는 `mcp` 키가 필요합니다.

<details>
<summary>Using uvx</summary>

```json
{
  "mcp": {
    "servers": {
      "fetch": {
        "command": "uvx",
        "args": ["mcp-server-fetch"]
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
      "fetch": {
        "command": "docker",
        "args": ["run", "-i", "--rm", "mcp/fetch"]
      }
    }
  }
}
```
</details>

### Customization - robots.txt

기본적으로 server는 요청이 model(tool을 통해)에서 온 경우 웹사이트의 robots.txt 파일을 준수하지만 사용자가 시작한(prompt를 통해) 요청인 경우 준수하지 않습니다. 구성의 `args` 목록에 `--ignore-robots-txt` 인수를 추가하여 비활성화할 수 있습니다.

### Customization - User-agent

기본적으로 요청이 model(tool을 통해)에서 온 것인지 사용자가 시작한(prompt를 통해) 것인지에 따라 server는 다음 user-agent를 사용합니다:
```
ModelContextProtocol/1.0 (Autonomous; +https://github.com/modelcontextprotocol/servers)
```
또는
```
ModelContextProtocol/1.0 (User-Specified; +https://github.com/modelcontextprotocol/servers)
```

구성의 `args` 목록에 `--user-agent=YourUserAgent` 인수를 추가하여 사용자 정의할 수 있습니다.

### Customization - Proxy

`--proxy-url` 인수를 사용하여 proxy를 사용하도록 server를 구성할 수 있습니다.

## Windows Configuration

Windows에서 timeout 문제가 발생하는 경우 적절한 문자 인코딩을 보장하기 위해 `PYTHONIOENCODING` 환경 변수를 설정해야 할 수 있습니다:

<details>
<summary>Windows configuration (uvx)</summary>

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"],
      "env": {
        "PYTHONIOENCODING": "utf-8"
      }
    }
  }
}
```
</details>

<details>
<summary>Windows configuration (pip)</summary>

```json
{
  "mcpServers": {
    "fetch": {
      "command": "python",
      "args": ["-m", "mcp_server_fetch"],
      "env": {
        "PYTHONIOENCODING": "utf-8"
      }
    }
  }
}
```
</details>

이는 Windows 시스템에서 server timeout을 발생시킬 수 있는 문자 인코딩 문제를 해결합니다.

## Debugging

MCP inspector를 사용하여 server를 디버그할 수 있습니다. uvx 설치의 경우:

```
npx @modelcontextprotocol/inspector uvx mcp-server-fetch
```

또는 특정 디렉토리에 패키지를 설치했거나 개발 중인 경우:

```
cd path/to/servers/src/fetch
npx @modelcontextprotocol/inspector uv run mcp-server-fetch
```

## Contributing

mcp-server-fetch를 확장하고 개선하는 데 도움을 주시기 바랍니다. 새로운 tool을 추가하거나 기존 기능을 향상시키거나 문서를 개선하고 싶으시다면 여러분의 의견은 소중합니다.

다른 MCP server 및 구현 패턴의 예는 다음을 참조하세요:
https://github.com/modelcontextprotocol/servers

Pull request를 환영합니다! mcp-server-fetch를 더욱 강력하고 유용하게 만들기 위해 새로운 아이디어, 버그 수정 또는 개선 사항을 자유롭게 기여해 주세요.

## License

mcp-server-fetch는 MIT License에 따라 라이선스가 부여됩니다. 즉, MIT License의 조건에 따라 소프트웨어를 자유롭게 사용, 수정 및 배포할 수 있습니다. 자세한 내용은 프로젝트 repository의 LICENSE 파일을 참조하세요.
