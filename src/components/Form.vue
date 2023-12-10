<template>
  

  <el-form ref="form" class="form" :model="formData" label-position="top">
    <header style="margin-bottom: 20px;">
      <div class="title">{{ $t('title') }}</div>
      <div class="title-desc">{{ $t('description') }}</div>
      <div style="margin-top: 10px;margin-bottom: 40px;" class="title-desc">
        {{ $t('targetFileds') }}
      </div>
    </header>


    <el-form-item :label="$t('funcLabel')" size="large" required>
      <el-select v-model="formulaFieldId" :placeholder="$t('funcPlaceholder')" style="width: 100%">
        <el-option v-for="meta in formulaFieldList" :key="meta.id" :label="meta.name" :value="meta.id" />
      </el-select>
    </el-form-item>

    
    <!-- 箭头图片 -->
    <!-- <div style="margin: 20px 0;width: 100%;text-align: center;">
      <svg width="24" height="24" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M24 44C35.0457 44 44 35.0457 44 24C44 12.9543 35.0457 4 24 4C12.9543 4 4 12.9543 4 24C4 35.0457 12.9543 44 24 44Z" fill="none" stroke="#3370ff" stroke-width="4" stroke-linejoin="round"/><path d="M24 15V33" stroke="#3370ff" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/><path d="M33 24L24 33L15 24" stroke="#3370ff" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div> -->

    <el-form-item :label="$t('dateLabel')" size="large" required>
      
      
      <!-- 选择目标字段列 -->
      <el-select v-model="otherFieldId" :placeholder="$t('otherPlaceholder')" style="width: 100%;">
        <el-option v-for="meta in otherFieldList" :key="meta.id" :label="meta.name" :value="meta.id" />
      </el-select>
      <!-- 选择目标字段列 -->

    </el-form-item>

    <!-- 交互式记录选择 -->
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

    <!-- 执行按钮 -->
    <el-button v-show="isConverted" style="width: 96px;height: 40px;margin-top: 12px;" color="#3370ff" type="primary" @click="convert">
      {{ $t('convertBtn') }}
    </el-button>
    <el-button v-show="!isConverted" :icon="Loading" disabled style="width: 96px;height: 40px;margin-top: 12px;" color="#3370ff" type="primary" @click="convert">
      {{ $t('convertBtn') }}
    </el-button>

    <el-alert type></el-alert>

  </el-form>
  
</template>

<script setup>
  import { bitable, FieldType  } from '@lark-base-open/js-sdk';
  import { ref, onMounted, computed  } from 'vue';
  import { Loading } from '@element-plus/icons-vue';

  // -- 数据区域
  const formData = ref({ table: '' });

  const formulaFieldId = ref('')  // 公式字段ID
  const formulaFieldList = ref([]);
  const otherFieldId = ref('')  // 其他类型字段ID
  const otherFieldList = ref([]);

  const isSelectPartRecords = ref(false)  // 是否是 “交互式选择记录”模式
  const targetFieldType = ref(1)  // 目标转换字段
  // --== 辅助数字区域
  const supportFieldsList = ref([
    'datetime', 'number', 'text', 'email', 'rating', 'price'
  ])
  const isConverted = ref(true)

  // -- 辅助算法区域
  // --== 弹出异常提示
  const showErrorToast = async (content) => {
    await bitable.ui.showToast({
      toastType: 'error',
      message: content
    })
    
    isConverted.value = true
  }

  // -- 核心算法区域
  // --001== 开始字段转换
  const convert = async () => {
    isConverted.value = false

    if (formulaFieldId.value.length == 0) {
      await bitable.ui.showToast({
        toastType: 'warning',
        message: '请选择需要映射的公式类型字段'
      })
      isConverted.value = true
      return
    } else if (otherFieldId.value.length == 0) {
      await bitable.ui.showToast({
        toastType: 'warning',
        message: '请选择要映射到的其他类型字段'
      })
      isConverted.value = true
      return
    }
  
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

    const targetFieldMetaList = await table.getFieldMetaById(otherFieldId.value)
    targetFieldType.value = targetFieldMetaList.type

    for (let recordId of RecordList) {
      console.log("recordId:", recordId)

      const formulaField = await table.getFieldById(formulaFieldId.value);
      const formulaValue = await formulaField.getValue(recordId);
      console.log("formulaValue:", formulaValue)
      
      if ( formulaValue !== null && typeof formulaValue === 'object' ) {
        await coreConvertAndWrite(recordId, formulaValue[0].text)
      }
      else 
        await coreConvertAndWrite(recordId, formulaValue)
      
      

    } // recordIdList  for循环结束

    isConverted.value = true
    
  }

  // --002==字段映射 switch-case函数
  const coreConvertAndWrite = async (recordId, formulaValue) => {
    const { tableId, viewId } = await bitable.base.getSelection();
    const table = await bitable.base.getActiveTable();
    const view = await table.getViewById(viewId);
    
    try {
        switch(targetFieldType.value) {
          case 5: 
            const datetimeField = await table.getFieldById(otherFieldId.value);
            const date = new Date(formulaValue);
            const datetimeValue = date.getTime();
            await datetimeField.setValue(recordId, datetimeValue);
            break;
          case 2: 
            const numberField = await table.getFieldById(otherFieldId.value);
            const numberValue = Number(formulaValue)
            if (Number.isNaN(numberValue)) {
              await showErrorToast(`⌈ ${formulaValue} ⌋ 为非法数字值`)
              return 
            }
            await numberField.setValue(recordId, numberValue);
            break;
          case 1: 
            const textField = await table.getFieldById(otherFieldId.value);
            const textValue = formulaValue
            await textField.setValue(recordId, textValue);
            break;
          case 99005: 
            const emailField = await table.getFieldById(otherFieldId.value);
            const emailValue = formulaValue
            if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(emailValue)) {
              await showErrorToast(`⌈ ${formulaValue} ⌋ 为非法邮箱`)
              return 
            }
            await emailField.setValue(recordId, emailValue);
            break;
          case 99004: 
            const ratingField = await table.getFieldById(otherFieldId.value);
            const ratingValue = formulaValue
            if (!Number.isInteger(ratingValue) || ratingValue < 0 || ratingValue > 10) {
              await showErrorToast(`⌈ ${ratingValue} ⌋ 为非法评分值，应为0~10之间的整数`)
              return 
            }
            await ratingField.setValue(recordId, ratingValue);
            break;
          case 99003: 
            const priceField = await table.getFieldById(otherFieldId.value);
            const priceValue = formulaValue
            if (isNaN(priceValue)) {
              await showErrorToast(`⌈ ${priceValue} ⌋ 为非法数值`)
              return 
            }
            await priceField.setValue(recordId, priceValue);
            break;
        }
      } catch(e) {
        await showErrorToast('错误，请检查配置')
      }
  }


  onMounted(async () => {
    const selection = await bitable.base.getSelection()
    const table = await bitable.base.getTableById(selection.tableId)
    const view = await table.getViewById(selection.viewId)
    formulaFieldList.value = await table.getFieldMetaListByType(FieldType.Formula)
    otherFieldList.value = await view.getFieldMetaList()
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



:deep(.el-alert .el-alert__icon) {
  color: #3370ff;
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
