---
title: 关于cannot find module 'xxxx’的一个可能解决方法
date: 2017-04-11 12:14:02
tags:
---
由于学习angular2，想单独学习一下typescript下angular2使用的‘rxjs’是怎么使用的，我用npm自己安装了rxjs，并使用了如下语句    
import { Observable } from 'rxjs';    
报错如下：   
```css 
cannot find module 'rxjs'，    
```
但是同样的语句在angular/cli生成的angular项目下是不报错的，我找了半天，各种解决方法都不适用于我遇到的情况。由于这个是typescript import时报的错误，我查看了typescript module对应的讲解：   
```html 
Module Resolution Strategies    
There are two possible module resolution strategies: Node and Classic. You can use the --moduleResolution flag to specify the module resolution strategy. If not specified, the default is Classic for --module AMD | System | ES2015or Node otherwise.   
``` 
也就是说typescript是有两种module管理的方式的，默认的不是node，是classic，我们看看node和classic分别是怎么进行import时的module查找的，我只说non-relative的情况，因为我这里要找node_modules下的第三方包，属于nonrelative：       
先说classic：       
```html 
Classic
This used to be TypeScript’s default resolution strategy. Nowadays, this strategy is mainly present for backward compatibility.    
For non-relative module imports, however, the compiler walks up the directory tree starting with the directory containing the importing file, trying to locate a matching definition file.     

For example:    

A non-relative import to moduleB such as import { b } from "moduleB", in a source file /root/src/folder/A.ts, would result in attempting the following locations for locating "moduleB":    
```
```css
/root/src/folder/moduleB.ts
/root/src/folder/moduleB.d.ts
/root/src/moduleB.ts
/root/src/moduleB.d.ts
/root/moduleB.ts
/root/moduleB.d.ts
/moduleB.ts
/moduleB.d.ts
```
找的可以看出其并不会找到nodejs的node_modules中，node的方式则会找到nodejs中，类似上面的方式，如果我也是import{b} from "moduleB", 逐级向上查找，代码如下所示：    
```css
/root/src/node_modules/moduleB.js
/root/src/node_modules/moduleB/package.json (if it specifies a "main" property)
/root/src/node_modules/moduleB/index.js 

/root/node_modules/moduleB.js
/root/node_modules/moduleB/package.json (if it specifies a "main" property)
/root/node_modules/moduleB/index.js 

/node_modules/moduleB.js
/node_modules/moduleB/package.json (if it specifies a "main" property)
/node_modules/moduleB/index.js
```
所以在项目tsconfig.json中添加一句话    
"moduleResolution": "node",    
覆盖掉默认配置classic，将能按node方式查找，问题解决。    
相关连接： https://www.typescriptlang.org/docs/handbook/module-resolution.html    