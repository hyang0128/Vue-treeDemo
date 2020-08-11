<!--
 * @Author: Hyang
 * @Date: 2020-04-14 10:34:30
 * @LastEditors: Hyang
 * @LastEditTime: 2020-08-11 19:01:26
 * @Description: file content
 -->
<template>
<li :class="liClass">
    <div class="tree-node-el">
        <span @click="expandNode(item)" v-if="hasChildren" class="tree-expand" :class="item.expanded ? 'tree-open' : 'tree-close'"></span>
        <div class="tree_item_content" :iskey="item.key" :class="item.checked?'item_checked':''" @click="handlecheckedChange(item,$event)">
            <span v-if="!item.nocheck" class='inputCheck' :class="spanInputClass">
                <input :disabled="item.chkDisabled" :class="['check', item.chkDisabled ? 'discheck' : '']" type="checkbox" :checked="item.checked">
            </span>
            <span :title="item.title" class="node-title">{{ item.title }}</span>
        </div>
    </div>
    <template>
        <collapse-transition>
            <ul class="tree-ul" v-show="item.expanded" v-if="hasChildren">
                <tree-item v-for="(child, index) in item.children" :key="child.id ? child.id : index" @handlecheckedChange="handlecheckedChange" :item="child"></tree-item>
            </ul>
        </collapse-transition>
    </template>
</li>
</template>

<script>
import CollapseTransition from "./collapse-transition";
export default {
    name: "tree-item",
    props: {
        item: {
            type: Object,
            default: () => {}
        },
    },
    components: {
        CollapseTransition
    },
    computed: {
        liClass() {
            return {
                leaf: this.hasChildren && this.item.level > 1
            };
        },
        hasChildren() {
            const item = this.item;
            return item.children && item.children.length > 0;
        },
        spanInputClass() {
            const {
                checked,
                chkDisabled
            } = this.item;
            let result = '';
            result += checked ? 'box-checked' : 'box-unchecked';
            result += chkDisabled ? ' chkDisabled' : '';
            return result;
        }
    },
    methods: {
        expandNode(node) {
            const expended = !node.expanded;
            this.$set(node, "expanded", expended);
        },
        handlecheckedChange(node, e) {
            this.$emit('handlecheckedChange', node)
        },
    }
}
</script>

<style lang="less" scoped>

</style>
