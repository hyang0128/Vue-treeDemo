<!--
 * @Author: Hyang
 * @Date: 2020-08-11 18:55:24
 * @LastEditors: Hyang
 * @LastEditTime: 2020-08-11 19:23:38
 * @Description: file content
-->
<template>
<div class="hello">
    <Tree ref="tree" :treeData="pointList" @node-change="itemChange"></Tree>
</div>
</template>

<script>
import Tree from "./tree/tree"
export default {
    name: 'HelloWorld',
    data() {
        return {
            pointList: [{
                title: 'node 1',
                children: [{
                        title: 'node 1-1',
                        children: [{
                                title: 'node 1-1-1'
                            },
                            {
                                title: 'node 1-1-2'
                            },
                            {
                                title: 'node 1-1-3'
                            },
                            {
                                title: 'node 1-1-4'
                            }
                        ]
                    },
                    {
                        title: 'node 1-2',
                        children: [{
                                title: 'node 1-2-1'
                            },
                            {
                                title: 'node 1-2-2',
                                children: [{
                                    title: 'node 1-2-2-1'
                                }]
                            },
                            {
                                title: 'node 1-2-3'
                            },
                            {
                                title: 'node 1-2-4'
                            }
                        ]
                    }
                ]
            }]
        }
    },
    components: {
        Tree
    },
    methods: {
        itemChange(node, checkedlist) {
            console.log(node, checkedlist);
        },
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style lang="less" scoped>
.hello {
    background-image: linear-gradient(rgba(0, 129, 220, 0.2) 0%, rgba(0, 129, 220, 0) 50%, rgba(0, 129, 220, 0.2) 100%);
    background-color: rgba(0, 129, 220, 1);
}
</style>
