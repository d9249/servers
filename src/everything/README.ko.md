# Everything MCP Server

이 MCP server는 MCP protocol의 모든 기능을 구현하려고 시도합니다. 유용한 server가 되기 위한 것이 아니라 MCP client 개발자를 위한 테스트 server입니다. prompt, tool, resource, sampling 등을 구현하여 MCP 기능을 보여줍니다.

## Components

### Tools

1. `echo`
   - 입력 메시지를 그대로 반환하는 간단한 tool
   - Input:
     - `message` (string): 반환할 메시지
   - Returns: 반환된 메시지가 포함된 텍스트 콘텐츠

2. `add`
   - 두 숫자를 더합니다
   - Inputs:
     - `a` (number): 첫 번째 숫자
     - `b` (number): 두 번째 숫자
   - Returns: 덧셈 결과 텍스트

3. `longRunningOperation`
   - 오래 실행되는 작업에 대한 진행 상황 알림을 보여줍니다
   - Inputs:
     - `duration` (number, default: 10): 실행 시간(초)
     - `steps` (number, default: 5): 진행 단계 수
   - Returns: 실행 시간과 단계가 포함된 완료 메시지
   - 실행 중 진행 상황 알림을 전송합니다

4. `printEnv`
   - 모든 환경 변수를 출력합니다
   - MCP server 구성 디버깅에 유용합니다
   - 입력 필요 없음
   - Returns: 모든 환경 변수의 JSON 문자열

5. `sampleLLM`
   - MCP sampling 기능을 사용하여 LLM sampling 기능을 보여줍니다
   - Inputs:
     - `prompt` (string): LLM에 전송할 prompt
     - `maxTokens` (number, default: 100): 생성할 최대 토큰 수
   - Returns: 생성된 LLM 응답

6. `getTinyImage`
   - 작은 테스트 이미지를 반환합니다
   - 입력 필요 없음
   - Returns: Base64로 인코딩된 PNG 이미지 데이터

7. `annotatedMessage`
   - annotation을 사용하여 콘텐츠에 대한 메타데이터를 제공하는 방법을 보여줍니다
   - Inputs:
     - `messageType` (enum: "error" | "success" | "debug"): 다양한 annotation 패턴을 보여주기 위한 메시지 유형
     - `includeImage` (boolean, default: false): 예제 이미지 포함 여부
   - Returns: 다양한 annotation이 포함된 콘텐츠:
     - Error 메시지: 높은 우선순위(1.0), 사용자와 assistant 모두에게 표시
     - Success 메시지: 중간 우선순위(0.7), 사용자 중심
     - Debug 메시지: 낮은 우선순위(0.3), assistant 중심
     - 선택적 이미지: 중간 우선순위(0.5), 사용자 중심
   - 예제 annotation:
     ```json
     {
       "priority": 1.0,
       "audience": ["user", "assistant"]
     }
     ```

8. `getResourceReference`
   - MCP client가 사용할 수 있는 resource reference를 반환합니다
   - Inputs:
     - `resourceId` (number, 1-100): 참조할 resource의 ID
   - Returns: 다음을 포함하는 resource reference:
     - 텍스트 소개
     - `type: "resource"`가 포함된 임베디드 resource
     - resource URI 사용에 대한 텍스트 지침

9. `startElicitation`
   - MCP client 내에서 elicitation(상호작용)을 시작합니다
   - Inputs:
      - `color` (string): 좋아하는 색상
      - `number` (number, 1-100): 좋아하는 숫자
      - `pets` (enum): 좋아하는 애완동물
   - Returns: 선택 요약과 함께 elicitation 데모 확인

10. `structuredContent`
   - 사양의 예제를 사용하여 구조화된 콘텐츠를 반환하는 tool을 보여줍니다
   - client가 스키마를 사용하여 결과를 검증해야 한다는 권고 사항을 테스트할 수 있도록 출력 스키마를 제공합니다
   - Inputs:
     - `location` (string): 위치 또는 우편번호, 값에 관계없이 모의 데이터가 반환됩니다
   - Returns: 다음을 포함하는 응답
     - 출력 스키마를 준수하는 `structuredContent` 필드
     - 사양의 권고 사항인 이전 버전과 호환되는 Text Content 필드

11. `listRoots`
   - client가 제공한 현재 MCP root를 나열합니다
   - 이 server는 파일에 액세스하지 않지만 root protocol 기능을 보여줍니다
   - 입력 필요 없음
   - Returns: URI와 이름이 포함된 현재 root 목록 또는 root가 설정되지 않은 경우 메시지
   - server가 MCP root protocol과 상호작용하는 방법을 보여줍니다

### Resources

server는 두 가지 형식으로 100개의 테스트 resource를 제공합니다:
- 짝수 번호 resource:
  - 일반 텍스트 형식
  - URI 패턴: `test://static/resource/{even_number}`
  - Content: 간단한 텍스트 설명

- 홀수 번호 resource:
  - 바이너리 blob 형식
  - URI 패턴: `test://static/resource/{odd_number}`
  - Content: Base64로 인코딩된 바이너리 데이터

Resource 기능:
- 페이지네이션 지원(페이지당 10개 항목)
- resource 업데이트 구독 허용
- resource template 시연
- 구독된 resource를 5초마다 자동 업데이트

### Prompts

1. `simple_prompt`
   - 인수가 없는 기본 prompt
   - Returns: 단일 메시지 교환

2. `complex_prompt`
   - 인수 처리를 보여주는 고급 prompt
   - 필수 인수:
     - `temperature` (string): 온도 설정
   - 선택적 인수:
     - `style` (string): 출력 스타일 선호도
   - Returns: 이미지가 포함된 다중 턴 대화

3. `resource_prompt`
   - prompt에 resource reference를 포함하는 방법을 보여줍니다
   - 필수 인수:
     - `resourceId` (number): 포함할 resource의 ID(1-100)
   - Returns: 임베디드 resource reference가 포함된 다중 턴 대화
   - prompt 메시지에 직접 resource를 포함하는 방법을 보여줍니다

### Roots

server는 MCP root protocol 기능을 보여줍니다:

- root 지원을 나타내기 위해 `roots: { listChanged: true }` 기능을 선언합니다
- client의 `roots/list_changed` 알림을 처리합니다
- server 초기화 중에 초기 root를 요청합니다
- 현재 root를 표시하는 `listRoots` tool을 제공합니다
- 시연 목적으로 root 관련 이벤트를 기록합니다

참고: 이 server는 실제로 파일에 액세스하지 않지만 파일 작업에 사용할 수 있는 디렉토리를 이해해야 하는 client를 위해 server가 root protocol과 상호작용하는 방법을 보여줍니다.

### Logging

server는 15초마다 임의 레벨의 로그 메시지를 전송합니다. 예:

```json
{
  "method": "notifications/message",
  "params": {
	"level": "info",
	"data": "Info-level message"
  }
}
```

## Usage with Claude Desktop ([stdio Transport](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports#stdio) 사용)

`claude_desktop_config.json`에 다음을 추가하세요:

```json
{
  "mcpServers": {
    "everything": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-everything"
      ]
    }
  }
}
```

## Usage with VS Code

빠른 설치를 위해 아래의 원클릭 설치 버튼 중 하나를 사용하세요...

[![Install with NPX in VS Code](https://img.shields.io/badge/VS_Code-NPM-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=everything&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-everything%22%5D%7D) [![Install with NPX in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-NPM-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=everything&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-everything%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=everything&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Feverything%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=everything&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Feverything%22%5D%7D&quality=insiders)

수동 설치의 경우 다음 방법 중 하나를 사용하여 MCP server를 구성할 수 있습니다:

**방법 1: 사용자 구성(권장)**
사용자 수준 MCP 구성 파일에 구성을 추가합니다. Command Palette(`Ctrl + Shift + P`)를 열고 `MCP: Open User Configuration`을 실행합니다. 그러면 server 구성을 추가할 수 있는 사용자 `mcp.json` 파일이 열립니다.

**방법 2: 작업 공간 구성**
또는 작업 공간의 `.vscode/mcp.json` 파일에 구성을 추가할 수 있습니다. 이렇게 하면 다른 사람과 구성을 공유할 수 있습니다.

> VS Code의 MCP 구성에 대한 자세한 내용은 [공식 VS Code MCP 문서](https://code.visualstudio.com/docs/copilot/mcp)를 참조하세요.

#### NPX

```json
{
  "servers": {
    "everything": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-everything"]
    }
  }
}
```

## Running from source with [HTTP+SSE Transport](https://modelcontextprotocol.io/specification/2024-11-05/basic/transports#http-with-sse) ([2025-03-26](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports) 기준 deprecated)

```shell
cd src/everything
npm install
npm run start:sse
```

## Run from source with [Streamable HTTP Transport](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports#streamable-http)

```shell
cd src/everything
npm install
npm run start:streamableHttp
```

## Running as an installed package
### Install
```shell
npm install -g @modelcontextprotocol/server-everything@latest
````

### Run the default (stdio) server
```shell
npx @modelcontextprotocol/server-everything
```

### Or specify stdio explicitly
```shell
npx @modelcontextprotocol/server-everything stdio
```

### Run the SSE server
```shell
npx @modelcontextprotocol/server-everything sse
```

### Run the streamable HTTP server
```shell
npx @modelcontextprotocol/server-everything streamableHttp
```
