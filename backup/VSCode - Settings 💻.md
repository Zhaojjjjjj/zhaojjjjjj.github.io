```json
	// 打字丝滑
	"editor.smoothScrolling": true,
	"editor.cursorBlinking": "expand",
	"editor.cursorSmoothCaretAnimation": "on",
	"workbench.list.smoothScrolling": true,

	// 鼠标控制页面大小
	"editor.mouseWheelZoom": true,

	// 彩虹括号与作用域块线条提示
	"editor.guides.bracketPairs": true,
	"editor.bracketPairColorization.enabled": true,

	// 自带的代码提示
	// 控制活动代码段是否阻止快速建议
	"editor.suggest.snippetsPreventQuickSuggestions": false,
	// 除了 `Tab` 键以外， `Enter` 键是否同样可以接受建议
	// 这能减少“插入新行”和“接受建议”命令之间的歧义
	"editor.acceptSuggestionOnEnter": "smart",
	// 代码补全列表中，优先选择最近的建议
	"editor.suggestSelection": "recentlyUsedByPrefix",

	// 自动补全括号引号
	"editor.autoClosingBrackets": "beforeWhitespace",
	"editor.autoClosingDelete": "always",
	"editor.autoClosingOvertype": "always",
	"editor.autoClosingQuotes": "beforeWhitespace",

	// window下美化窗口 ( 改变的是编辑器的提示框，Mac不生效 )
	// "window.dialogStyle": "custom",

	// 自动换行和行高
	"editor.wordWrap": "on",
	"editor.lineHeight": 1.5,

	// 文件夹紧凑模式
	"explorer.compactFolders": false,
	"notebook.compactView": false,

	// 格式化自动删除无意义的分号
	"javascript.format.semicolons": "remove",
	"typescript.format.semicolons": "remove",

	// Vue 自动补全 .value 和缺失属性提醒
	"vue.inlayHints.missingProps": true,
	"vue.autoInsert.dotValue": true,
	"workbench.startupEditor": "none",

	// 自动猜测文本的编码
	"files.autoGuessEncoding": true,

	// 保存自动删除末尾空格
	"files.trimTrailingWhitespace": false,

	// 终端代码补全
	"terminal.integrated.shellIntegration.suggestEnabled": true

	// 搜索排除目录
	"search.exclude": {
	  "**/node_modules": true,
	  "**/pnpm-lock.yaml": true,
	  "**/package-lock.json": true,
	  "**/.DS_Store": true,
	  "**/.git": true,
	  "**/.gitignore": true,
	  "**/.idea": true,
	  "**/.svn": true,
	  "**/.vscode": true,
	  "**/build": true,
	  "**/dist": true,
	  "**/tmp": true,
	  "**/yarn.lock": true
	}

```