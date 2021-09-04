# mve:create a todo with vite

## 1st prepare vite environment

1. open vite Official website([Home | Vite (vitejs.dev)](https://vitejs.dev/)),follow the document,create a vanilla project with ts.
2. open this project with vscode,run ```npm i``` in the shell.
3. run ```npm run dev``` at shell,then open the url printed at the shell.

4. try edit main.ts,such as change ```Hello Vite``` to ```Hello Vite with mve```.text in browser change immediately.pretty good.

## 2nd  prepare mve requirement

1. add mve dependence

   ```sh
   npm i mve-dom --save
   ```

2. change some code of main.ts：

   ```typescript
   import { dom } from 'mve-dom'
   import './style.css'
   
   const app = document.querySelector<HTMLDivElement>('#app')!
   
   // app.innerHTML = `
   //   <h1>Hello Vite with mve!</h1>
   //   <a href="https://vitejs.dev/guide/features.html" target="_blank">Documentation</a>
   // `
   
   const root=dom.root(function(me) {
     return {
       type:"div",
       text:"Hello Vite With mve"
     }
   })
   app.append(root.element)
   if(root.init){
     root.init()
   }
   if(root.destroy){
     window.addEventListener("close",root.destroy)
   }
   ```

3. you many need to refresh browser,then saw what's new.

## 3rd  make a simple todo view

1. define the todo model.we add a todo.ts file under src folder.

   ```TypeScript
   import { mve } from "mve-core/util"
   
   interface Todo{
     content:mve.Value<string>
     finish:mve.Value<boolean>
   }
   
   const todos=mve.arrayModelOf<Todo>([])
   ```

   we simply add two field,import mve.the ```mve.Value<T>``` represent what can be changed,the ```mve.ArrayModel<T>``` represent the todo list.

2. finish the view.think about a todo interface,it should have a input area,a list to show the records,in which we can change the content、mark it to finished or just delete it.there we accomplish it with table, with who the first column is checkbox,the second column is input and the last column has delete href.there is the code:

   ```typescript
   import { mve } from "mve-core/util"
   import { dom } from "mve-dom"
   import {modelChildren} from "mve-core/modelChildren"
   
   interface Todo{
     content:mve.Value<string>
     finish:mve.Value<boolean>
   }
   
   const todos=mve.arrayModelOf<Todo>([])
   
   
   let addInput:HTMLInputElement
   export const todoView=dom({
     type:"div",
     children:[
       dom({
         type:"input",
         init(e){
           addInput=e
         }
       }),
       dom({
         type:"button",
         text:"添加",
         event:{
           click(){
             const v=addInput.value.trim()
             if(v){
               todos.unshift({
                 content:mve.valueOf(v),
                 finish:mve.valueOf(false)
               })
               addInput.value=""
             }else{
               alert("请输入内容")
             }
           }
         }
       }),
       dom({
         type:"table",
         children:modelChildren(todos,function(me,todo,i) {
           return dom({
             type:"tr",
             children:[
               dom({
                 type:"td",
                 children:[
                   dom({
                     type:"input",
                     attr:{
                       type:"checkbox"
                     },
                     prop:{
                       checked:todo.finish
                     },
                     event:{
                       change(e){
                         const checkbox=e.target as HTMLInputElement
                         todo.finish(checkbox.checked)
                       }
                     }
                   })
                 ]
               }),
               dom({
                 type:"td",
                 children:[
                   dom({
                     type:"input",
                     value:todo.content,
                     event:{
                       change(e){
                         const input=e.target as HTMLInputElement
                         const v=input.value.trim()
                         if(v){
                           todo.content(v)
                         }else{
                           input.value=todo.content()
                         }
                       }
                     }
                   })
                 ]
               }),
               dom({
                 type:"td",
                 children:[
                   dom({
                     type:"a",
                     text:"删除",
                     attr:{href:"javascript:void(0)"},
                     event:{
                       click(){
                         todos.remove(i())
                       }
                     }
                   })
                 ]
               })
             ]
           })
         })
       })
     ]
   })
   ```

   the code may be long,because mve use pure ts not xml,in order to get more reuse of code.the logic of the code is simple.you can see dom function is just like a html element such as div、input,it has attr、prop、event field,which is mapped to html element.you can just jump to ```mve-dom/index``` to saw the definition.why created in this way?because we can simply use dom api in this way.

3. link it at main.ts.we simply link the todoView like follow:delete the text field,add children array field,in which we add todoView.

   ```typescript
   import { dom } from 'mve-dom'
   import './style.css'
   import { todoView } from './todo'
   
   const app = document.querySelector<HTMLDivElement>('#app')!
   
   // app.innerHTML = `
   //   <h1>Hello Vite with mve!</h1>
   //   <a href="https://vitejs.dev/guide/features.html" target="_blank">Documentation</a>
   // `
   
   const root=dom.root(function(me) {
     return {
       type:"div",
       children:[
         todoView
       ]
     }
   })
   app.append(root.element)
   if(root.init){
     root.init()
   }
   if(root.destroy){
     window.addEventListener("close",root.destroy)
   }
   
   ```

   save,and see what happen in browser.

   ![image-20210904143456291](/Users/wangyang/Documents/github/article/mve-begin-a-simple-todo-with-vite.assets/image-20210904143456291.png)

   it may a little bit ugly.I have no reason.the main character is mve in this article.we can saw the dom structure with f12.

## 4th review what we did

1. we crate a list,in which we can add、delete、edit 

2. we can go to the definition of ```mve.Value```,it is a function,has two usage:get a value of change,or only get the value.I call it the Storage.

3. we can go to the definition of dom,at the same file there has a svg function,there parameter has attr、style、event、children etc.the attr、style is map which value can be a real value or a function will caculate to the value,when it is function,it's just like the vue bind.I think the source code is not too much,read the code is more helpful than give a document.

4. the children type is EOChildren<EO>```，we can jump to the definitation:

   ```TypeScript
   /**重复的函数节点/组件封装成mve*/
   export interface EOChildFun<EO>{
   	(parent:VirtualChild<EO>,me:mve.LifeModel):BuildResult
   }
   
   /**存放空的生命周期 */
   export class ChildLife{
   	constructor(public readonly result:BuildResult){}
   	static of(result:BuildResult){
   		return new ChildLife(result)
   	}
   }
   
   export type EOChildren<EO>= EO | EOChildFun<EO> | EOChildren<EO>[] | ChildLife
   ```

   there we design a circle include,so you can combinate your code in a flexible way.

5. modelChildren is develop for bind from ```mve.ArrayModel``` to the view.I use is at most time of my use.it is similar to 'for' repeat of other framework,but it is one-sub-item to one-fragment between model and view.mve has no component system,use function reuse to build the system.you may also need ```ifChildren```,it is under ```mve-core/ifChildren```.

6. how about filter?in Array,filter will return a new Array,but in mve,even we have a similar filter(mve-core/filterChildren),but we recommind modelChildren in mvvm style,so you may need a ```hide:mve.Value<boolean>``` field to bind with the display style.

## 5th at last

I create mve library for 3-4 year,the relay count is inspired by vue,and add many surprise functions have been added one after another.At the beginning, I mainly felt that the DSL function of js itself was very powerful, and there was no need for a set of XML. and lots of new framework doesn't support IE8 when I had to support it.

Lot's of work now use vue and react which I don't think is good for me.so I recommend my mve library.it is a more pure mvvm,and can have a higher performance,at least I wish the front-end work can be more easier.

I don't know how to tell when I already familiar with it.give a reply at my github or any where I can notice.I have a chinses qq group which is 925964319.

(I can hard to open github)

## 6th more

### the project of this article

chinese gitee

[mve-vite-demo: 使用vite来构建mve (gitee.com)](https://gitee.com/wy2010344/mve-vite-demo)

github

[wangyang2010344/mve-vite-demo (github.com)](https://github.com/wangyang2010344/mve-vite-demo)



### the original mve

chinese gitee

[mve: 以vue单线程依赖统计为核心的一个类vue框架，更准确说是函数，用纯javascript实现而无xml/css依赖，基于自己做的简单的mb.ajax.require，附带简单的JavaScript上S-Lisp实现及简单的S-Lisp版本 (gitee.com)](https://gitee.com/wy2010344/mve)

github

[wangyang2010344/mve: 以vue单线程依赖统计为核心的一个类vue框架，更准确说是函数，用纯javascript实现而无xml/css依赖，基于自己做的简单的mb.ajax.require，附带简单的JavaScript上S-Lisp实现及简单的S-Lisp版本 (github.com)](https://github.com/wangyang2010344/mve)



### the mve of npm

chinese gitee

[npm_mve: mve core for npm (gitee.com)](https://gitee.com/wy2010344/npm_mve)

[npm_mve-DOM: a simple dom birdge for mve (gitee.com)](https://gitee.com/wy2010344/npm_mve-DOM)

github

[wangyang2010344/npm_mve (github.com)](https://github.com/wangyang2010344/npm_mve)

[wangyang2010344/npm_mve-DOM (github.com)](https://github.com/wangyang2010344/npm_mve-DOM)















