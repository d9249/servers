# Knowledge Graph Memory Server

로컬 지식 그래프를 사용한 영구 메모리의 기본 구현입니다. 이를 통해 Claude는 채팅 간에 사용자에 대한 정보를 기억할 수 있습니다.

## Core Concepts

### Entities
Entity는 지식 그래프의 기본 노드입니다. 각 entity는 다음을 가집니다:
- 고유한 이름(식별자)
- entity 유형(예: "person", "organization", "event")
- observation 목록

예제:
```json
{
  "name": "John_Smith",
  "entityType": "person",
  "observations": ["Speaks fluent Spanish"]
}
```

### Relations
Relation은 entity 간의 방향성 연결을 정의합니다. 항상 능동태로 저장되며 entity가 상호작용하거나 관련되는 방식을 설명합니다.

예제:
```json
{
  "from": "John_Smith",
  "to": "Anthropic",
  "relationType": "works_at"
}
```
### Observations
Observation은 entity에 대한 개별 정보입니다. 다음과 같습니다:

- 문자열로 저장됩니다
- 특정 entity에 첨부됩니다
- 독립적으로 추가하거나 제거할 수 있습니다
- 원자적이어야 합니다(observation당 하나의 사실)

예제:
```json
{
  "entityName": "John_Smith",
  "observations": [
    "Speaks fluent Spanish",
    "Graduated in 2019",
    "Prefers morning meetings"
  ]
}
```

## API

### Tools
- **create_entities**
  - 지식 그래프에 여러 개의 새 entity를 생성합니다
  - Input: `entities` (객체 배열)
    - 각 객체에는 다음이 포함됩니다:
      - `name` (string): Entity 식별자
      - `entityType` (string): 유형 분류
      - `observations` (string[]): 관련 observation
  - 기존 이름을 가진 entity는 무시합니다

- **create_relations**
  - entity 간에 여러 개의 새 relation을 생성합니다
  - Input: `relations` (객체 배열)
    - 각 객체에는 다음이 포함됩니다:
      - `from` (string): 소스 entity 이름
      - `to` (string): 대상 entity 이름
      - `relationType` (string): 능동태로 표현된 관계 유형
  - 중복 relation은 건너뜁니다

- **add_observations**
  - 기존 entity에 새 observation을 추가합니다
  - Input: `observations` (객체 배열)
    - 각 객체에는 다음이 포함됩니다:
      - `entityName` (string): 대상 entity
      - `contents` (string[]): 추가할 새 observation
  - entity당 추가된 observation을 반환합니다
  - entity가 존재하지 않으면 실패합니다

- **delete_entities**
  - entity와 그 relation을 제거합니다
  - Input: `entityNames` (string[])
  - 관련 relation의 계단식 삭제
  - entity가 존재하지 않으면 자동 작업

- **delete_observations**
  - entity에서 특정 observation을 제거합니다
  - Input: `deletions` (객체 배열)
    - 각 객체에는 다음이 포함됩니다:
      - `entityName` (string): 대상 entity
      - `observations` (string[]): 제거할 observation
  - observation이 존재하지 않으면 자동 작업

- **delete_relations**
  - 그래프에서 특정 relation을 제거합니다
  - Input: `relations` (객체 배열)
    - 각 객체에는 다음이 포함됩니다:
      - `from` (string): 소스 entity 이름
      - `to` (string): 대상 entity 이름
      - `relationType` (string): 관계 유형
  - relation이 존재하지 않으면 자동 작업

- **read_graph**
  - 전체 지식 그래프를 읽습니다
  - 입력 필요 없음
  - 모든 entity와 relation을 포함하는 전체 그래프 구조를 반환합니다

- **search_nodes**
  - 쿼리를 기반으로 노드를 검색합니다
  - Input: `query` (string)
  - 다음에서 검색합니다:
    - Entity 이름
    - Entity 유형
    - Observation 콘텐츠
  - 일치하는 entity와 그 relation을 반환합니다

- **open_nodes**
  - 이름으로 특정 노드를 검색합니다
  - Input: `names` (string[])
  - Returns:
    - 요청된 entity
    - 요청된 entity 간의 relation
  - 존재하지 않는 노드는 자동으로 건너뜁니다

# Usage with Claude Desktop

### Setup

claude_desktop_config.json에 다음을 추가하세요:

#### Docker

```json
{
  "mcpServers": {
    "memory": {
      "command": "docker",
      "args": ["run", "-i", "-v", "claude-memory:/app/dist", "--rm", "mcp/memory"]
    }
  }
}
```

#### NPX
```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-memory"
      ]
    }
  }
}
```

#### NPX with custom setting

다음 환경 변수를 사용하여 server를 구성할 수 있습니다:

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-memory"
      ],
      "env": {
        "MEMORY_FILE_PATH": "/path/to/custom/memory.jsonl"
      }
    }
  }
}
```

- `MEMORY_FILE_PATH`: 메모리 저장 JSONL 파일 경로(기본값: server 디렉토리의 `memory.jsonl`)

# VS Code Installation Instructions

빠른 설치를 위해 아래의 원클릭 설치 버튼 중 하나를 사용하세요:

[![Install with NPX in VS Code](https://img.shields.io/badge/VS_Code-NPM-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=memory&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-memory%22%5D%7D) [![Install with NPX in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-NPM-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=memory&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-memory%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=memory&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22-v%22%2C%22claude-memory%3A%2Fapp%2Fdist%22%2C%22--rm%22%2C%22mcp%2Fmemory%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=memory&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22-v%22%2C%22claude-memory%3A%2Fapp%2Fdist%22%2C%22--rm%22%2C%22mcp%2Fmemory%22%5D%7D&quality=insiders)

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
    "memory": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-memory"
      ]
    }
  }
}
```

#### Docker

```json
{
  "servers": {
    "memory": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "-v",
        "claude-memory:/app/dist",
        "--rm",
        "mcp/memory"
      ]
    }
  }
}
```

### System Prompt

메모리를 활용하기 위한 prompt는 사용 사례에 따라 다릅니다. prompt를 변경하면 model이 생성된 메모리의 빈도와 유형을 결정하는 데 도움이 됩니다.

다음은 채팅 개인화를 위한 예제 prompt입니다. 이 prompt를 [Claude.ai Project](https://www.anthropic.com/news/projects)의 "Custom Instructions" 필드에서 사용할 수 있습니다.

```
각 상호작용에 대해 다음 단계를 따르세요:

1. User Identification:
   - default_user와 상호작용하고 있다고 가정해야 합니다
   - default_user를 식별하지 못한 경우 적극적으로 식별을 시도하세요.

2. Memory Retrieval:
   - 항상 채팅을 시작할 때 "Remembering..."이라고만 말하고 지식 그래프에서 모든 관련 정보를 검색하세요
   - 항상 지식 그래프를 "memory"라고 부르세요

3. Memory
   - 사용자와 대화하는 동안 다음 범주에 속하는 새로운 정보에 주의하세요:
     a) Basic Identity (나이, 성별, 위치, 직책, 교육 수준 등)
     b) Behaviors (관심사, 습관 등)
     c) Preferences (의사소통 스타일, 선호하는 언어 등)
     d) Goals (목표, 대상, 열망 등)
     e) Relationships (최대 3단계 분리까지의 개인 및 전문 관계)

4. Memory Update:
   - 상호작용 중에 새로운 정보가 수집된 경우 다음과 같이 메모리를 업데이트하세요:
     a) 반복되는 조직, 사람 및 중요한 이벤트에 대한 entity를 생성합니다
     b) relation을 사용하여 현재 entity에 연결합니다
     c) observation으로 그들에 대한 사실을 저장합니다
```

## Building

Docker:

```sh
docker build -t mcp/memory -f src/memory/Dockerfile .
```

참고: 이전 mcp/memory volume에는 새 컨테이너에 의해 덮어쓸 수 있는 index.js 파일이 포함되어 있습니다. 저장을 위해 docker volume을 사용하는 경우 새 컨테이너를 시작하기 전에 이전 docker volume의 `index.js` 파일을 삭제하세요.

## License

이 MCP server는 MIT License에 따라 라이선스가 부여됩니다. 즉, MIT License의 조건에 따라 소프트웨어를 자유롭게 사용, 수정 및 배포할 수 있습니다. 자세한 내용은 프로젝트 repository의 LICENSE 파일을 참조하세요.
