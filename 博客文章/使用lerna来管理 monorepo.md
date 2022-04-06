

> 参考文档： https://zhuanlan.zhihu.com/p/71385053

## yarn 的 workspace

熟悉要改造一下 package.json，加上下面两行：

```json
"workspaces": [
	"packages/*"
],
"private": true,
```

### 常见问题

如果忘记了 workspaces 字段，那么在为某个包单独添加依赖的时候，就会报以下错误：

```bash
yarn workspace v1.22.15
error Cannot find the root of your workspace - are you sure you're currently in a workspace?
info Visit https://yarnpkg.com/en/docs/cli/workspace for documentation about this command.
```

如果报下面的错误：

```bash
error Unknown workspace "@qunyou/lk-editor-react".
info Visit https://yarnpkg.com/en/docs/cli/workspace for documentation about this command.
```

则需要检查一下模块包里 package.json 里的 name 字段是否有对应的值。