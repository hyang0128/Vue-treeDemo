<template>
<ul class="tree-ul">
    <tree-item class="first-node only-node" v-for="(child, index) in treeSource" :key="child.id ? child.id : index" @handlecheckedChange="handlecheckedChange" :item="child"></tree-item>
</ul>
</template>

<script>
import treeItem from "./tree-node";
export default {
    props: {
        treeData: {
            type: Array,
            default: () => []
        }
    },
    data() {
        return {
            //处理过（遍历添加Key和level属性）的树数据，用于显示
            treeSource: [],
            //展开后的平级map树
            roots: new Map()
        }
    },
    components: {
        treeItem: treeItem
    },
    watch: {
        "treeData": {
            handler() {
                this.setTreeSource();
            },
            deep: true
        }
    },
    created() {
        this.setTreeSource();
    },
    methods: {
        setTreeSource() {
            if (this.treeData && Object.prototype.toString.call(this.treeData) == '[object Array]') {
                this.treeSource = (this.reduceTreeFunc(this.treeData, '0') || []).slice(0);
                this.roots = new Map();
                const _traverseNodes = (root) => {
                    for (let node of root) {
                        this.roots.set(node.key, node)
                        if (node.children && node.children.length > 0) _traverseNodes(node.children)
                    }
                }
                _traverseNodes(this.treeSource)
                // console.log('root',this.roots)
            }
        },
        reduceTreeFunc(data = [], key, level = 1, parentKey) {
            let copyData = data.slice(0);
            let treeData = copyData.map((item, i) => {
                item.nodeKey = key + '-' + i.toString();
                if (!item.hasOwnProperty('key')) {
                    item.key = item.nodeKey;
                }
                if (!item.hasOwnProperty('level')) {
                    item.level = level;
                }
                if (parentKey && level > 1) {
                    item.parentKey = parentKey;
                }
                if (item.hasOwnProperty('children') && item.children.length > 0) {
                    this.reduceTreeFunc(item.children, item.nodeKey, level + 1, item.key);
                }
                return item;
            })
            return treeData;
        },
        handlecheckedChange(node) {
            if (node.chkDisabled) return;
            this.$set(node, 'checked', !node.checked)
            this.changeCheckStatus(node)
            this.$emit('node-change', node, this.getCheckedNode())
        },
        setCheckedKeys(keys) {
            if (Object.prototype.toString.call(keys) == '[object Array]') {
                this.clearChecked();
                // console.log('keys',keys)
                let leng = keys.length;
                for (let i = 0; i < leng; i++) {
                    let node = this.getNode(keys[i]);
                    // console.log(this.roots)
                    if (node) {
                        this.$set(node, 'checked', true);
                        this.changeCheckStatus(node);
                    }
                }
            }
        },
        setCheckedAndExpand(key) {
            if (key) {
                this.setCheckedKeys([key]);
                const _traverseUp_expand = (node) => {
                    //往上寻找，没有父节点停止
                    if (!node.parentKey) {
                        return false
                    }
                    let key = this.__parseKey(node);
                    let parent = this.getNode(key.parent);
                    if (parent) {
                        this.$set(parent, 'expanded', true);
                        _traverseUp_expand(parent);
                    }
                };
                let node = this.getNode(key);
                if (node) {
                    this.$emit('node-change', node, this.getCheckedNode())
                    _traverseUp_expand(node);
                }

            }
        },
        changeCheckStatus(node) {
            const _traverseUp = (node) => {
                //往上寻找，level=1停止
                if (node.level === 1) {
                    return false
                }

                let key = this.__parseKey(node)
                if (node.checked) {
                    let parent = this.getNode(key.parent)
                    this.$set(parent, 'checked', this.sameSilibingChecked(key.parent, key.current))
                    _traverseUp(parent)
                } else {
                    if (!node.checked) {
                        let upparent = this.getNode(key.parent)
                        this.$set(upparent, 'checked', false)
                        if (upparent.key) {
                            _traverseUp(upparent)
                        }
                    }
                }
            }

            const _traverseDown = (node) => {
                if (node.children && node.children.length > 0) {
                    for (let child of node.children) {
                        this.$set(child, 'checked', node.checked)
                        _traverseDown(child)
                    }
                }
            }
            _traverseUp(node)
            _traverseDown(node)
        },
        expandLevel1() {
            this.roots.forEach(item => {
                if (item.level === 1)
                    this.$set(item, 'expanded', true);
            });
        },
        getNode(key) {
            return this.roots.get(key)
        },
        __parseKey(node) {
            let parent = node.parentKey ? node.parentKey : null;
            return {
                current: node.key,
                parent: parent
            }
        },
        getCheckedNode() {
            let arr = [];
            this.roots.forEach(item => {
                if (item.checked)
                    arr.push(item);
            });
            return arr;
        },
        clearChecked() {
            this.roots.forEach(item => this.$set(item, 'checked', false));
        },
        clearExpand() {
            this.roots.forEach(item => this.$set(item, 'expanded', false));
        },
        sameSilibingChecked(parentKey, currentKey) {

            let parent = this.roots.get(parentKey)
             console.log(this.roots,parent)
            let sbIds = []
            parent.children.forEach(x => {
                if (x.key !== currentKey) sbIds.push(x.key)
            })
            for (let key of sbIds) {
                let node = this.getNode(key)
                if (!node.checked) return false
            }
            return true
        }
    }
}
</script>

<style lang="less">
.tree-ul {
    font-size: 14px;
    -webkit-transition: 0.3s height ease-in-out, 0.3s padding-top ease-in-out,
        0.3s padding-bottom ease-in-out;
    transition: 0.3s height ease-in-out, 0.3s padding-top ease-in-out,
        0.3s padding-bottom ease-in-out;
    user-select: none;
    margin: 0;

    .first-node {
        cursor: pointer;
        padding: 5px;
    }

    ul {
        box-sizing: border-box;
    }

    ul,
    li {
        list-style-type: none;
        text-align: left;
    }

    li span:hover {
        background-color: transparent;
        /* padding: 6px 0; */
    }

    .fade-enter-active,
    .fade-leave-active {
        transition: opacity 0.2s;
    }

    .fade-enter,
    .fade-leave-to

    /* .fade-leave-active in below version 2.1.8 */
        {
        opacity: 0;
    }

    .expand-enter-active {
        transition: all 3s ease;
        height: 50px;
        overflow: hidden;
    }

    .expand-leave-active {
        transition: all 3s ease;
        height: 0px;
        overflow: hidden;
    }

    .expand-enter,
    .expand-leave {
        height: 0;
        opacity: 0;
    }

    .inputCheck {
        display: inline-block;
        position: relative;
        width: 14px;
        height: 14px;
        border: 1px solid #fffefb;
        border-radius: 2px;
        top: 6px;
        text-align: center;
        font-size: 14px;
        line-height: 14px;
    }

    .inputCheck.notAllNodes:before {
        content: "\2713";
        display: block;
        position: absolute;
        width: 100%;
        height: 100%;
        background-color: #888888;
        z-index: 1;
        color: #ffffff;
    }

    .inputCheck.box-checked:after {
        content: "\2713";
        display: block;
        position: absolute;
        z-index: 1;
        width: 100%;
        text-align: center;
        color: #4abded;
    }

    .box-checked {
        border-color: #4eccfc;
    }

    .box-halfchecked {
        background-color: #888888;
    }

    .box-halfchecked:after {
        content: "\2713";
        display: block;
        position: absolute;
        z-index: 1;
        width: 100%;
        text-align: center;
        color: #ffffff;
    }

    .check {
        display: block;
        position: absolute;
        font-size: 14px;
        width: 14px;
        height: 14px;
        /* left: -5px;
        top: -4px; */
        border: 1px solid #000000;
        opacity: 0;
        cursor: pointer;
        -ms-filter: "progid:DXImageTransform.Microsoft.Alpha(Opacity=0)";
        filter: alpha(opacity=0);
        z-index: 2;
    }

    .chkDisabled {
        border-color: #999999;
    }

    .chkDisabled:after {
        color: #999999 !important;
    }

    .discheck {
        cursor: not-allowed;
    }

    li {
        margin: 0;
        padding: 5px 5px 5px 15px;
        position: relative;
        list-style: none;
    }

    li:after,
    li:before {
        content: "";
        left: -8px;
        position: absolute;
        right: auto;
        border-width: 1px;
    }

    li:before {
        border-left: 1px solid #cccccf;
        bottom: 50px;
        height: 100%;
        top: -9px;
        width: 1px;
    }

    li:after {
        border-bottom: 1px solid #cccccf;
        height: 6px;
        top: 12px;
        left: -6px;
        width: 19px;
        transform: skew(30deg);
        border-left: 1px solid #cccccf;
        border-bottom-left-radius: 5px;
    }

    li:last-child::before {
        height: 21px;
    }

    li:first-child::before {
        height: calc(100% + 4px);
        top: -13px;
    }

    li.first-node:before {
        top: 17px;
    }

    li.second-node:before {
        top: 4px;
    }

    li.first-node.only-node::before {
        border-left: none;
    }

    li.only-node:after {
        border-bottom: none;
        border-left: none;
    }

    ul {
        padding-left: 0;
    }

    ul {
        padding-left: 17px;
        padding-top: 10px;
    }

    .tree-expand {
        display: inline-block;
        padding: 4px 8px;
        text-align: center;
        // line-height: 13px;
        // border: 1px solid #888888;
        // border-radius: 2px;
        // background: #ffffff;
        font-style: normal;
        position: relative;
        top: 6px;
    }

    .tree-close:after {
        // content: "+";
        content: '';
        width: 8px;
        height: 8px;
        position: absolute;
        border-right: 2px solid #cccccf;
        border-bottom: 2px solid #cccccf;
        transform: rotate(-45deg);
        left: 1px;
        top: 3px;
    }

    .tree-open:after {
        // content: "\2013";
        content: '';
        width: 8px;
        height: 8px;
        position: absolute;
        border-right: 2px solid #cccccf;
        border-bottom: 2px solid #cccccf;
        transform: rotate(45deg);
        left: 3px;
        top: 1px;
    }

    .tree-empty:after {
        content: "";
    }

    .tree-node-el {
        // background-color: #ffffff;
        padding-left: 2px;
        position: relative;
        z-index: 3;
        display: flex;
    }

    .tree_item_content {
        padding: 0px 5px;
        display: flex;
    }

    .item_checked {
        background-color: #2c7cc6;
    }

    li.leaf {
        padding-left: 15px;
    }

    .node-title {
        padding: 3px 3px;
        border-radius: 3px;
        cursor: pointer;
        margin: 0 2px;
        color: #fffefb;
        white-space: nowrap;
    }

    .tree_item_content:hover {
        background-color: #2c7cc6;
    }

    .node-title-disabled {
        padding: 3px 3px;
        border-radius: 3px;
        background-color: #f5f5f5;
        opacity: 1;
        cursor: not-allowed;
        margin: 0 2px;
    }

    .node-title-disabled:hover {
        background-color: #f5f5f5;
    }

    .node-selected {
        border: 1px solid #dddddd;
        background-color: #dddddd;
    }

    .node-title.node-searched {
        border: 1px solid #ff8247;
    }
}
</style>
