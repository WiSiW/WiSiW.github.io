---
title: 【Webpack】模块打包原理
date: 2021-09-30 10:11:07
tags:
    - Webpack
    - 模块打包
cover_picture: /images/webpack.jpeg
---
**文件列表**
```text

```
# 1.读取文件生成AST
```javascript
const fs = require('fs')
const traverse = require('babel-traverse').default
const { transformFromAst } = require('babel-core')
/*
 * 根据输入的文件相对地址生成AST
 * fs.readFileSync()
 * traverse()
 * transformFromAst()
 */
function createAsset(fileName) {
    const content = fs.readFileSync(fileName, 'utf-8')

    const ast = babylon.parse(content, {
        sourceType: 'module'
    })
    const dependencies = []
    traverse(ast,{
        ImportDeclaration: ({node}) => {
            dependencies.push(node.source.value)
        }
    })
    const id = ID++
    const { code } = transformFromAst(ast, null, {
        presets: ['env']
    })
    return {
        id,
        fileName,
        dependencies,
        code
    }
}
```

# 2.根据入口文件生成模块依赖树
```javascript
function createGraph(entry) {
    const mainAsset = createAsset(entry)
    const queue = [mainAsset]
    for(const ast of queue) {
        const dirname = path.dirname(ast.fileName)
        ast.mapping = {}
        ast.dependencies.forEach(relationPath => {
            const absolutePath = path.join(dirname, relationPath)
            const child = createAsset(absolutePath)
            ast.mapping[relationPath] = child.id
            queue.push(child)
        })
    }
    return queue
}
```

**源代码**
```javascript
const fs = require('fs')
const path = require('path')
const babylon = require('babylon')
const traverse = require('babel-traverse').default
const { transformFromAst } = require('babel-core')

let ID = 0

function createAsset(fileName) {
    const content = fs.readFileSync(fileName, 'utf-8')

    const ast = babylon.parse(content, {
        sourceType: 'module'
    })
    const dependencies = []
    traverse(ast,{
        ImportDeclaration: ({node}) => {
            dependencies.push(node.source.value)
        }
    })
    const id = ID++
    const { code } = transformFromAst(ast, null, {
        presets: ['env']
    })
    return {
        id,
        fileName,
        dependencies,
        code
    }
}

function createGraph(entry) {
    const mainAsset = createAsset(entry)
    const queue = [mainAsset]
    for(const ast of queue) {
        const dirname = path.dirname(ast.fileName)
        ast.mapping = {}
        ast.dependencies.forEach(relationPath => {
            const absolutePath = path.join(dirname, relationPath)
            const child = createAsset(absolutePath)
            ast.mapping[relationPath] = child.id
            queue.push(child)
        })
    }
    return queue
}

function bundle(graph) {
    let modules = ''
    graph.forEach(mod => {
        modules += `${mod.id}: [
            function(require, module, exports) {
                ${mod.code}
            },
            ${JSON.stringify(mod.mapping)}
        ],`
    })
    const res = `
        (function(modules) {
            function require(id) {
                const [fn, mapping] = modules[id]

                function localRequire(relativePath) {
                    return require(mapping[relativePath])
                }

                const module = { exports: {} }

                fn(localRequire, module, module.exports)

                return module.exports
            }
            require(0)
        })({${modules}})
    `
    return res
}

const graph = createGraph('./example/entry.js')
const res = bundle(graph)
console.log(res)
```
