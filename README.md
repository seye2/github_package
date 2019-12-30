# github_package
## 목적 
GitHub Actions를 통한 배포 시스템을 진행 시에 repository간 공통 코드를 관리하고자 만들었습니다.

## 해당 Package 사용 방법
- https://musma.github.io/2019/09/30/github-package-registry.html
- 위 내용을 참고해 Github Package를 사용하기 위한 개별 권한 설정을 진행해야 합니다.
- 해당 패키지를 사용하기 위해서는 .npmrc파일을 생성하여 토큰을 설정하여야 합니다.

$ repository name > .npmrc
```
@{github_id}:registry=https://npm.pkg.github.com/
```

## 기능
- action.yml에 정의된 변수 값에 따라 기능이 실행됩니다.
```
name: "Slack message"
description: "Send job results on Slack via Incoming Webhooks 🔔"
inputs:
  ACTION_NAME:
    description: "Name of the action for a more descriptive message"
    required: true # 필수 옵션
  JOB:
    description: "JSON-stringified `job` variable. Must be passed to output the actual status of the job"
    required: true # 필수 옵션
  SLACK_WEBHOOK_URL:
    description: "URL of the Incoming Webhook provided by Slack"
    required: true # 필수 옵션
runs:
  using: 'node12'                                           // 설치 os
  main: 'index.js'                                           // Github Actions에 사용시 실행될 파일 명
```

### Github Actions에서는 아래와 같이 패키지 설치 후에 진행하면 됩니다.

```
 name: Example Github Actions Workflow

on:
  push:
    branches:
      - master

jobs:
  merge:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [10.x]

    steps:
      - uses: actions/checkout@v1
        with:
          ref: ${{ matrix.refs }}

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}

      - name: Package Install & Yarn Build Backdoor
        run: |
          yarn
          ls -al
        env:
          NODE_AUTH_TOKEN: ${{secrets.NODE_AUTH_TOKEN}}

      - name: Send Slack
        if: success() || failure()
        uses: ./node_modules/@seye2/github_package
        with:
          ACTION_NAME: Run test scripts
          JOB: ${{ toJson(job) }}
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # required
```

