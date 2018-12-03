<!--
{
    "title": "vscode相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "vscode",
        "visual studio code"
    ],
    "info": []
}
-->

## vscode配置

扩展：

```vscode
Chinese (Simplified) Language Pack for Visual Studio Code
Markdown All in One
markdownlint
vscode-icons
```

配置：

```vscode
settings.json:
{
    "workbench.iconTheme": "vscode-icons",
    "workbench.colorTheme": "Monokai Dimmed",
    "extensions.ignoreRecommendations": true,
    "markdownlint.config": {
        "MD002": false,
        "MD007": {
            "indent": 4
        },
        "MD009": {
            "br_spaces": 2
        },
        "MD013": false,
        "MD041": false
    }
}

keybindings.json:
[
]
```
