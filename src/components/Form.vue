<template>
  

  <el-form ref="form" class="form" :model="formData" label-position="left">
    <header style="margin-bottom: 20px;">
      <div class="title">{{ $t('title') }}</div>
      <div class="title-desc">{{ $t('description') }}</div>
    </header>


    <el-form-item :label="$t('funcLabel')" size="large" required>
      <el-select v-model="formulaFieldId" :placeholder="$t('funcPlaceholder')" style="width: 100%">
        <el-option v-for="meta in formulaFieldList" :key="meta.id" :label="meta.name" :value="meta.id" />
      </el-select>
    </el-form-item>

    <!-- <div style="margin: 20px 0;width: 100%;text-align: center;">
      <svg width="24" height="24" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M24 44C35.0457 44 44 35.0457 44 24C44 12.9543 35.0457 4 24 4C12.9543 4 4 12.9543 4 24C4 35.0457 12.9543 44 24 44Z" fill="none" stroke="#3370ff" stroke-width="4" stroke-linejoin="round"/><path d="M24 15V33" stroke="#3370ff" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/><path d="M33 24L24 33L15 24" stroke="#3370ff" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div> -->

    <el-form-item :label="$t('dateLabel')" size="large" required>
      <el-select v-model="datetimeFieldId" :placeholder="$t('datePlaceholder')" style="width: 100%">
        <el-option v-for="meta in datetimeFieldList" :key="meta.id" :label="meta.name" :value="meta.id" />
      </el-select>
    </el-form-item>

    <div style="display: flex; flex-direction: row;align-items: center;">
      <el-checkbox style="margin-right: 20px;" v-model="isSelectPartRecords" :label="$t('convertMode')" size="large" />

      <el-popover
        placement="top-start"
        :width="200"
        trigger="hover"
        :content="$t('convertModeInfo')"
      >
        <template #reference>
          <svg width="18" height="18" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M24 44C29.5228 44 34.5228 41.7614 38.1421 38.1421C41.7614 34.5228 44 29.5228 44 24C44 18.4772 41.7614 13.4772 38.1421 9.85786C34.5228 6.23858 29.5228 4 24 4C18.4772 4 13.4772 6.23858 9.85786 9.85786C6.23858 13.4772 4 18.4772 4 24C4 29.5228 6.23858 34.5228 9.85786 38.1421C13.4772 41.7614 18.4772 44 24 44Z" fill="none" stroke="#3370ff" stroke-width="4" stroke-linejoin="round"/><path d="M24 28.6248V24.6248C27.3137 24.6248 30 21.9385 30 18.6248C30 15.3111 27.3137 12.6248 24 12.6248C20.6863 12.6248 18 15.3111 18 18.6248" stroke="#3370ff" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/><path fill-rule="evenodd" clip-rule="evenodd" d="M24 37.6248C25.3807 37.6248 26.5 36.5055 26.5 35.1248C26.5 33.7441 25.3807 32.6248 24 32.6248C22.6193 32.6248 21.5 33.7441 21.5 35.1248C21.5 36.5055 22.6193 37.6248 24 37.6248Z" fill="#3370ff"/></svg>
        </template>
      </el-popover>

    </div>

    <el-button style="width: 96px;height: 40px;margin-top: 12px;" color="#3370ff" type="primary" @click="convert">
      {{ $t('convertBtn') }}
    </el-button>
  </el-form>
  
</template>

<script setup>
  import { bitable, FieldType  } from '@lark-base-open/js-sdk';
  import { ref, onMounted, computed  } from 'vue';


  // -- 数据区域
  const formData = ref({ table: '' });

  const formulaFieldId = ref('')  // 公式字段ID
  const formulaFieldList = ref([]);
  const datetimeFieldId = ref('')  // 日期字段ID
  const datetimeFieldList = ref([]);

  const isSelectPartRecords = ref(false)

  

  // -- 算法区域
  const convert = async () => {
    const { tableId, viewId } = await bitable.base.getSelection();
    const table = await bitable.base.getActiveTable();
    const view = await table.getViewById(viewId);

    let RecordList = []

    if (isSelectPartRecords.value) {
      // 选择记录会话框
      RecordList = await bitable.ui.selectRecordIdList(tableId, viewId);
    } else {
      // 全部记录
      RecordList = await view.getVisibleRecordIdList()
    }

    for (let recordId of RecordList) {
      console.log("recordId:", recordId)

      const formulaField = await table.getFieldById(formulaFieldId.value);
      const formulaValue = await formulaField.getValue(recordId);
      console.log("formulaValue:", formulaValue)

      const datetimeField = await table.getFieldById(datetimeFieldId.value);
      const date = new Date(formulaValue);
      const timestamp = date.getTime();
      // 写入数据
      await datetimeField.setValue(recordId, timestamp);
    }

    
  }


  onMounted(async () => {
    const selection = await bitable.base.getSelection()
    const table = await bitable.base.getTableById(selection.tableId)
    const view = await table.getViewById(selection.viewId)
    formulaFieldList.value = await table.getFieldMetaListByType(FieldType.Formula)
    datetimeFieldList.value = await table.getFieldMetaListByType(FieldType.DateTime)
  });

  
</script>


<style scoped>

header {
  margin-bottom: 20px;
}

.title {
  color: #1f2329;
  font-size: 18px;
  padding-left: 10px;
  border-left: 4px solid #3370ff;
  margin-bottom: 20px;
  padding-top: 5px;
}

.title-desc {
  color: #8f959e;
  font-size: 14px;
  line-height: 1.5;
}





:deep(.el-radio-button__original-radio:checked+.el-radio-button__inner) {
  background-color: #3370ff;
}

:deep(.el-radio-button__inner:hover) {
  color: #3e75f5;
}

:deep(.el-radio-button__original-radio:checked+.el-radio-button__inner:hover) {
  color: #fff;
}
</style>
