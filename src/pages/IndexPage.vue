<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input
          ref="nameRef"
          v-model="tempData.name"
          label="姓名"
          :rules="[(val) => !!val || '姓名未填寫']"
        />
        <q-input
          ref="ageRef"
          v-model="tempData.age"
          label="年齡"
          :rules="ageRules"
        />
        <q-btn
          v-if="isEdit"
          color="primary"
          class="q-mt-md"
          @click="updateTempData"
          >更新</q-btn
        >
        <q-btn
          v-else
          color="primary"
          class="q-mt-md"
          @click="addTempData"
          :disabled="!tempData.name || !tempData.age"
          >新增</q-btn
        >
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);
const isEdit = ref<boolean>(false);
const tempData = ref<{ name: string; age: number }>({
  name: '',
  age: '',
});
const nameRef = ref(null);
const ageRef = ref(null);
const ageRules = [
  (val) => (val !== null && val !== '') || '年齡未填寫',
  (val) => (val > 0 && val < 100) || '請輸入真實年齡',
];
// add
async function addTempData(): Promise<void> {
  await axios.post(
    'https://dahua.metcfire.com.tw/api/CRUDTest/',
    tempData.value
  );
  getTempData();
}
// get
async function getTempData(): Promise<void> {
  const { data }: { data: { id: string; name: string; age: string }[] } =
    await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
  blockData.value = data;
}
// patch
async function updateTempData(): Promise<void> {
  try {
    await axios.patch(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      tempData.value
    );
    tempData.value = {
      name: '',
      age: '',
    };
    getTempData();
    isEdit.value = false;
    ageRef.value.resetValidation();
    nameRef.value.resetValidation();
  } catch (error) {
    console.log(error);
  }
}

// Delete
async function removeTempData(id: string): Promise<void> {
  await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`);

  getTempData();
}
getTempData();

function handleClickOption(btn, data) {
  if (btn.status === 'edit') {
    tempData.value = data;
    isEdit.value = true;
  } else if (btn.status === 'delete') {
    removeTempData(data.id);
  }
}
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
