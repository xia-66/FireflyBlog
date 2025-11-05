---
title: "Vue3实现复制到剪贴板"
published: 2024-09-29
updated: 2024-09-29
description: ""
tags: ["Vite","Vue3"]
category: "前端相关"
draft: false
---

**安装[vue-clipboard3][1]库**

    npm install --save vue-clipboard3

**测试用法**

    <template lang=\"html\">
      <button @click=\"copy\">Copy!</button>
    </template>
    
    <script lang=\"ts\" setup>
      import useClipboard from \'vue-clipboard3\'
      const { toClipboard } = useClipboard()
      const copy = async () => {
        try {
          await toClipboard(\'Any text you like\')
          console.log(\'Copied to clipboard\')
        } catch (e) {
          console.error(e)
        }
      }
    </script>

**进阶用法**

  在utils目录下新建一个  copy.ts  文件，并封装复制功能

    import useClipboard from \"vue-clipboard3\";
    import { ElMessage } from \"element-plus\";
    export async function copy(text) {
      const { toClipboard } = useClipboard();
      try {
        await toClipboard(text);
        ElMessage({
          message: \"复制成功\",
          type: \"success\",
        });
      } catch (e) {
        ElMessage({
          message: \"复制失败，请手动复制\",
          type: \"error\",
        });
      }
    }
*局部使用*

    <template>
      <el-table :data=\"tableData\" style=\"width: 100%\">
        <el-table-column prop=\"date\" label=\"Date\" width=\"180\">
          <template #default=\"{ row }\">
            <div>{{ row.date }}<el-icon  @click=\"copy(row.date)\"><CopyDocument /></el-icon></div>
          </template>
        </el-table-column>
        <el-table-column prop=\"name\" label=\"Name\" width=\"180\" />
        <el-table-column prop=\"address\" label=\"Address\" width=\"260\"/>
      </el-table>
    </template>
    
    <script lang=\"ts\" setup>
    import { copy } from \'../../src/utils/copy\'
    import { CopyDocument } from \'@element-plus/icons-vue\';
    const tableData = [
      {
        date: \'2016-05-03\',
        name: \'Tom\',
        address: \'No. 189, Grove St, Los Angeles\'
      },
      {
        date: \'2016-05-02\',
        name: \'Tom\',
        address: \'No. 189, Grove St, Los Angeles\'
      },
      {
        date: \'2016-05-04\',
        name: \'Tom\',
        address: \'No. 189, Grove St, Los Angeles\'
      },
      {
        date: \'2016-05-01\',
        name: \'Tom\',
        address: \'No. 189, Grove St, Los Angeles\'
      }
    ]
    </script>

*全局注册*
    //main.ts
    import { copy } from \"../src/utils/copy\";
    app.config.globalProperties.$copy = copy;

    ----------------------------------
    //页面使用
    <template>
      <el-table :data=\"tableData\" style=\"width: 100%\">
        <el-table-column prop=\"date\" label=\"Date\" width=\"180\">
          <template #default=\"{ row }\">
            <div>{{ row.date }}<el-icon  @click=\"$copy(row.date)\"><CopyDocument /></el-icon></div>
          </template>
        </el-table-column>
        <el-table-column prop=\"name\" label=\"Name\" width=\"180\" />
        <el-table-column prop=\"address\" label=\"Address\" width=\"260\"/>
      </el-table>
    </template>
    
    <script lang=\"ts\" setup>
    import { CopyDocument } from \'@element-plus/icons-vue\';
    const tableData = [
      {
        date: \'2016-05-03\',
        name: \'Tom\',
        address: \'No. 189, Grove St, Los Angeles\'
      },
      {
        date: \'2016-05-02\',
        name: \'Tom\',
        address: \'No. 189, Grove St, Los Angeles\'
      },
      {
        date: \'2016-05-04\',
        name: \'Tom\',
        address: \'No. 189, Grove St, Los Angeles\'
      },
      {
        date: \'2016-05-01\',
        name: \'Tom\',
        address: \'No. 189, Grove St, Los Angeles\'
      }
    ]
    </script>

**效果图如下**
![copy.png][2]


  [1]: https://github.com/JamieCurnow/vue-clipboard3
  [2]: https://blog.heiyu.fun/usr/uploads/2024/09/576536961.png