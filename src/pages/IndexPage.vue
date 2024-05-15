<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref } from 'vue';
import { useQuasar } from 'quasar';
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

const tempData = ref({
  name: '',
  age: '',
});

const btnStatus = ref(1);
const chooseId = ref(null);
// const deletModal = ref(false);

axios
  .get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
  .then(function (response) {
    console.log(response.data);
    blockData.value = response.data;
  });

async function handleClickOption(btn, data) {
  console.log(btn.status);
  console.log(data);
  if (btn.status === 'delete') {
    confirm();
    chooseId.value = data.id;
  } else if (btn.status === 'edit') {
    tempData.value.name = data.name;
    tempData.value.age = data.age;
    chooseId.value = data.id;
    btnStatus.value = 2;
  }
}

//新增
const create = async () => {
  if (tempData.value.name === '' || tempData.value.age === '') {
    alert('輸入不得為空');
  } else if (
    !/^\d+$/.test(tempData.value.age) ||
    parseInt(tempData.value.age) === 0
  ) {
    alert('年齡輸入有誤');
  } else {
    try {
      await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', {
        name: tempData.value.name,
        age: tempData.value.age,
      });
      const response = await axios.get(
        'https://dahua.metcfire.com.tw/api/CRUDTest/a'
      );
      blockData.value = response.data;
      tempData.value.name = '';
      tempData.value.age = '';
    } catch (error) {
      console.error('Error occurred:', error);
    }
  }
};

//編輯
const edit = async () => {
  if (tempData.value.name === '' || tempData.value.age === '') {
    alert('輸入不得為空');
  } else if (
    !/^\d+$/.test(tempData.value.age) ||
    parseInt(tempData.value.age) === 0
  ) {
    alert('年齡輸入有誤');
  } else {
    try {
      await axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', {
        id: chooseId.value,
        name: tempData.value.name,
        age: tempData.value.age,
      });
      const response = await axios.get(
        'https://dahua.metcfire.com.tw/api/CRUDTest/a'
      );
      blockData.value = response.data;
      tempData.value.name = '';
      tempData.value.age = '';
      chooseId.value = null;
      btnStatus.value = 1;
    } catch (error) {
      console.error('Error occurred:', error);
    }
  }
};

//刪除
const deleteData = async () => {
  try {
    // 發起 DELETE 請求刪除數據
    await axios.delete(
      `https://dahua.metcfire.com.tw/api/CRUDTest/${chooseId.value}`
    );
    console.log('Deletion successful');

    // 發起 GET 請求獲取更新後的數據列表
    const response = await axios.get(
      'https://dahua.metcfire.com.tw/api/CRUDTest/a'
    );
    console.log('Updated data fetched successfully');
    blockData.value = response.data;
    chooseId.value = null;
    return blockData.value; // 返回更新後的數據列表
  } catch (error) {
    console.error('Error occurred during the operation:', error);
  }
};

const $q = useQuasar();

const confirm = () => {
  $q.dialog({
    title: '提示',
    message: '是否確定要刪除此筆資料嗎?',
    cancel: true,
    persistent: true,
  })
    .onOk(() => {
      // console.log('>>>> OK');
      deleteData();
    })
    .onOk(() => {
      // console.log('>>>> second OK catcher')
    })
    .onCancel(() => {
      // console.log('>>>> Cancel')
    })
    .onDismiss(() => {
      // console.log('I am triggered on both OK and Cancel')
    });
};

//按年齡排序
const sort = ref(false);

const sortDataByAge = (val) => {
  if (val === 1) {
    sort.value = true;
    blockData.value.sort((a, b) => a.age - b.age);
  } else if (val === 2) {
    sort.value = false;
    axios
      .get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
      .then(function (response) {
        console.log(response.data);
        blockData.value = response.data;
      });
  }
};
</script>

<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn
          v-if="btnStatus === 1"
          color="primary"
          class="q-mt-md"
          @click="create"
          >新增</q-btn
        >
        <q-btn
          v-if="btnStatus === 2"
          color="primary"
          class="q-mt-md"
          @click="edit"
          >更新</q-btn
        >
      </div>
      <q-btn v-if="!sort" class="q-mt-md" @click="sortDataByAge(1)"
        >按年齡排序</q-btn
      >
      <q-btn v-if="sort" class="q-mt-md" @click="sortDataByAge(2)"
        >取消按年齡排序</q-btn
      >
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
    <!-- <div v-if="deletModal" class="q-pa-md q-gutter-sm">
      <q-btn label="Confirm" color="primary" @click="confirm" />
    </div> -->
  </q-page>
</template>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
