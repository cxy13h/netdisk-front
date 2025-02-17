<template>
  <div class="custom-tree-container">
    <el-tree
        style="max-width: 600px"
        :data="dataSource"
        show-checkbox
        node-key="id"
        default-expand-all
        :expand-on-click-node="false"
    >
      <template #default="{ node, data }">
        <span class="custom-tree-node">
          <span>{{ node.label }}</span>
          <span>
            <a @click="append(data)"> Append </a>
            <a style="margin-left: 8px" @click="remove(node, data)"> Delete </a>
          </span>
        </span>
      </template>
    </el-tree>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import type Node from 'element-plus/es/components/tree/src/model/node'

interface Tree {
  id: number
  type: string
  label: string
  children?: Tree[]
}
let id = 1000

const append = (data: Tree) => {
  console.log(data)
}

const remove = (node: Node, data: Tree) => {
  console.log(data)

  console.log(node)
}


const dataSource = ref<Tree[]>([
  {
    id: 2,
    type: "directory",
    label: "一级目录-1",
    children: [
      {
        id: 4,
        type: "directory",
        label: "二级目录-1",
        children: [
          {
            id: 5,
            type: "directory",
            label: "三级目录-1",
            children: []
          },
          {
            id: 3,
            type: "file",
            label: "文件3",
            children: null
          }
        ]
      },
      {
        id: 1,
        type: "file",
        label: "文件1",
        children: null
      }
    ]
  }
])
</script>

<style>
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>
