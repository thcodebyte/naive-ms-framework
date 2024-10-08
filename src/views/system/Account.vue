<script setup lang="ts">

import PageContainer from "@/components/PageContainer.vue";
import EditableTable from "@/components/editableTable/EditableTable.vue";
import {h, nextTick, reactive, ref} from "vue";
import type {Acc} from "@/interface/interface";
import {requestHandler} from "@/utils/requestHandler";
import type {DropdownOption, Pagination, TableColumn} from "@/type/common";
import ShowOrEdit from "@/components/editableTable/ShowOrEdit.vue";
import {NButton, NButtonGroup, NDropdown} from "naive-ui";
import ContextMenu from "@/components/ContextMenu.vue";
import {Stripe} from '@vicons/fa'
import {Refresh} from '@vicons/tabler'

const accList = ref<Acc[]>();
requestHandler<Acc[]>("GET", "/account")
    .then(res => accList.value = res.data);
const cols: TableColumn<Acc>[] = [
  {
    title: 'id',
    key: 'id',
  },
  {
    title: '用户名',
    key: 'username',
    r(row) {
      return h(ShowOrEdit, {
        rawValue: row.username,
        callback: (v) => {
          console.log(v);
          const index = accList.value!.findIndex(a => a.id === row.id);
          accList.value![index].username = v;
        }
      });
    },
  },
  {
    title: '角色',
    key: 'role',
    r(row) {
      return h(NDropdown, {
        value: row.role,
        trigger: 'click',
        options: [
          {label: '启用', key: 'ACTIVE'},
          {label: '禁用', key: 'DISABLE'},
          {label: '归档', key: 'ARCHIVED'},
        ],
      }, {
        default: () => h(NButton, {
          tertiary: true,
          type: "info",
          class: 'w-20'
        }, row.role)
      });
    }
  },
  {
    title: '绑定员工',
    key: 'staff',
  },
  {
    title: '创建时间',
    key: 'createdAt',
  },
  {
    title: '操作',
    key: 'operations',
    r(row) {
      return h(NButtonGroup, {
        size: 'small',
      }, {
        default: () => [
          h(NButton, {
            type: 'error',
            size: 'small',
            tertiary: true,
            onClick: () => console.log('edit', row)
          }, '删除'),
          h(NButton, {
            type: 'warning',
            size: 'small',
            tertiary: true,
            onClick: () => console.log('edit', row)
          }, '重置密码')
        ]
      })
    },
  }
]
const pagination = reactive<Pagination>({
  page: 1,
  pageSize: 2,
  pageCount: 10,
  pageCallback: (page) => console.log(page)
});

const options: DropdownOption[] = [
  {
    label: '编辑',
    key: 'edit'
  },
  {
    label: '删除',
    key: 'delete'
  }
];
// 右键菜单位置
const xRef = ref(0);
const yRef = ref(0);
// 右键菜单ref
const tableContextMenuRef = ref<InstanceType<typeof ContextMenu>>();
const striped = ref(false);

function handleTabsContextMenu(e: MouseEvent) {
  e.preventDefault();
  findClickedTab(e);
  nextTick().then(() => {
    tableContextMenuRef.value?.show();
    xRef.value = e.clientX;
    yRef.value = e.clientY;
  });
}

function findClickedTab(e: MouseEvent) {
  const clickedElement = e.target as HTMLElement;
  const tabElement = <HTMLElement>clickedElement.parentElement;
  if (tabElement) {
    const attribute = tabElement.getAttribute('data-name');
    return attribute || null;
  }
  return null;
}
</script>

<template>
  <page-container>
    <n-card class="flex-grow" content-class="w-full h-full flex">
      <template #header>
        <n-form inline label-placement="left">
          <n-form-item>
            <n-button type="info">
              新增用户
            </n-button>
          </n-form-item>
          <n-form-item label="">
            <n-input placeholder="请输入员工姓名" clearable></n-input>
          </n-form-item>
          <n-form-item>
            <n-button type="info" tertiary>搜索</n-button>
          </n-form-item>
          <div class="flex-grow"></div>
          <n-form-item>
            <n-flex size="large">
              <n-button text>
                <template #icon>
                  <n-icon :component="Refresh"></n-icon>
                </template>
              </n-button>
              <n-button text @click="striped = !striped">
                <template #icon>
                  <n-icon :component="Stripe" size="30" />
                </template>
              </n-button>
            </n-flex>
          </n-form-item>
        </n-form>
      </template>
      <editable-table
          :data="accList"
          :columns="cols"
          :key="(row: Acc) => row.id"
          :pagination="pagination"
          :striped="striped"
          @contextmenu="handleTabsContextMenu"
      />
      <context-menu ref="tableContextMenuRef" :options="options" :x="xRef" :y="yRef"/>
    </n-card>
  </page-container>
</template>

<style scoped>
</style>