---
title: vscode一些配置
date: 2019-04-25 00:19:52
tags:
  - vscode
---

```json
{
  "editor.formatOnSave": true,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "editor.wordWrapColumn": 120,
  "prettier.printWidth": 120,
  "vetur.format.defaultFormatterOptions": {
    "prettyhtml": {
      "printWidth": 120, // No line exceeds 120 characters
      "singleQuote": false // Prefer double quotes over single quotes
    }
  }
}

```

[vetur插件更多配置详情](<https://vuejs.github.io/vetur/>)

