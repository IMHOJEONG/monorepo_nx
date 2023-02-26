## Nx.dex

- https://nx.dev/getting-started/package-based-repo-tutorial

### 1. 새로운 작업공간 만들기 

- 새 작업 공간을 만들어 시작하자 

  - 설정하는 데 도움이 되는 다음 명령어 

```shell
npx create-nx-workspace@latest package-based --preset=npm
```

### 2. 패키지 만들기 

- 패키지 폴더는 우리가 monorepo 라이브러리를 호스팅하는 곳 

  - 다음 구조로 새 `is-even` 폴더를 만듬 

```
package-based/
├── packages/
│   └── is-even/
│       ├── index.ts
│       └── package.json
├── nx.json
└── package.json
```

- TypeScript를 설치함 

  - 왜? package.json에서 빌드 스크립트에 tsc를 사용하고 있음에 유의!

- 패키지 수준에서 Typescript를 설치할 수 있지만 전체 단일 저장소에 대해 전역적으로 설치하는 것이 더 편리 

```shell 
npm i typescript -D -W

npx nx build is-even
```

### 3. 패키지의 로컬 연결 

- 패키지 기반 mono repository 스타일에서 로컬로 패키지를 연결하는 것은 NPM/Yarn/PNPM 작업 영역에서 수행됨 

  - 이 특정 설정에서는 NPM 작업 영역을 사용 

    - 작업 영역 root에 있는 package.json 파일의 작업 영역 속성 참조 

- 패키지를 로컬로 연결하는 방법을 설명하려면?

  - `is-odd`라는 다른 패키지 연결 

- `is-odd`는 `is-even` 패키지에서 isEven 함수를 가져옴 

  - 따라서, package.json 파일은 package.json 파일의 `is-even` 패키지를 종속성으로 나열해야 함 

- root 수준 package.json의 workspace 속성은 package 디렉터리에 있는 모든 패키지에 대한 링크를 생성하도록 NPM에 지시 

  - 이렇게 하면 먼저 NPM 레지스트리에 게시할 필요가 없음 

  - Yarn 및 PNPM 작업영역에도 유사한 기능 존재

```shell
npm install 
```

- npm은 파일 시스템에서 `node_modules/is-even` 및 `node_modules/is-odd`의 심볼릭 링크를 생성하므로 

  - 변경 사항이 발생하면 `packages/is-even` 및 `packages/is-odd` 디렉토리에 반영됨 

### 4. Task Dependencies

- 대부분의 monorepo는 서로 다른 패키지 사이뿐만 아니라 task 사이에도 종속성이 있음 

- ex) `is-odd`를 빌드할 때마다 `is-even`이 미리 빌드되었는지 확인해야 함 

- Nx는 nx.json에 targetDefaults 속성을 추가하여 이러한 task 종속성을 정의할 수 있음 

```json
"targetDefaults": {
    "build": {
      "dependsOn" : ["^build"]
    }
}
```

- 패키지 자체의 빌드 대상이 실행되기 전에 모든 종속 프로젝트의 빌드 대상을 먼저 실행하도록 Nx에 지시 

- 기존 dist 폴더를 제거하고 다음을 실행 
```shell
npx nx build is-odd
```

- is-even 패키지에 대한 빌드를 자동으로 먼저 실행한 다음, is-odd에 대한 빌드를 실행 

  - is-even이 이전에 빌드된 경우 캐시에서 복원됨 

### 5. 캐시 빌드 결과 

```shell
npx nx build is-even
```

- 빌드 스크립트의 캐시는 이전에 이 튜토리얼에서 실행했을 때 이미 채워져 있었음 

### 6. 여러 Task를 실행 

```shell
npx nx run-many --target=build
```

- 두 빌드 모두 캐시에서 재생됨 

  - `--skip-nx-cache` 옵션을 추가해 캐시를 건너뛸 수 있음 

```shell
npx nx run-many --target=build --skip-nx-cache
```

- 이 방법을 사용하면 is-even 빌드가 is-odd 빌드보다 먼저 실행되고 is-even 빌드가 한 번만 발생했음을 알 수 있음 

  - 이전에 설정한 targetDefaults가 run-many에 알리는 방법을 보여줌 

- 다음 명령을 사용해 변경된 패키지에서만 작업을 실행할 수도 있음 

```shell
npx nx affected --target=build
```

- base 및 head 옵션은 기본값으로 채워져 있음 

  - 필요에 따라 고유한 옵션을 여기에 제공할 수 있음 

- 영향을 받는 명령과 함께 cache도 사용된다는 점에 유의

  



