```
mkdir npm-test    // 创建文件夹

npm init    // 初始化

// 注册 npm 账号

npm config get registry    // 是否为 npm 官方的源，不是就切回

npm config set registry https://registry.npmjs.org

npm login    // 登录 npm 账户

npm publish    // 发布 npm 包 

npm whoami    // 查看当前登录 npm 的账户

npm version patch    // 补丁版本，版本号最后一位数加1
npm version minor    // 增加了新功能，版本号中间数加1
npm version major    // 大改动，不向下兼容，版本号第一位数加1
```
### package.json
- main 定义入口文件，默认是 index.js
- name 发布后的 npm 包名
- keywords 关键词
- description 描述