> Global install nrm

> NPM error message during Mac installation may be due to insufficient permissions. Add ***sudo*** before the command
```
npm install -g nrm  /  sudo npm install -g nrm
```
> View all mirrors
```
nrm ls 
```
> Example
```
@sdeMacBook-Pro ~ % nrm ls
  npm ---------- https://registry.npmjs.org/
  yarn --------- https://registry.yarnpkg.com/
  tencent ------ https://mirrors.cloud.tencent.com/npm/
  cnpm --------- https://r.cnpmjs.org/
  taobao ------- https://registry.npmmirror.com/
  npmMirror ---- https://skimdb.npmjs.com/registry/

@sdeMacBook-Pro ~ % nrm use taobao  // Change to taobao
```
