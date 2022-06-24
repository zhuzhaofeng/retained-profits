<script setup>
import { computed, reactive, ref } from 'vue'
import ResultView from '@/components/ResultView/ResultView.vue'

const utilizationRatePicker = ref(false)

const resultVisible = ref(false)

const columns = [0.1, 0.2, 0.5, 0.7, 0.9]
const formState = reactive({
  // 柜子数量
  quantity: undefined,
  // quantity: 6,
  // 押金金额
  deposit: undefined,
  // deposit: 15,
  // 收费价格
  charge: undefined,
  // charge: 15,
  // 分成金额
  sharing: undefined,
  // sharing: 1.5,
  // 租金
  rent: undefined,
  // rent: 500,
  // 使用率
  utilizationRate: 0.5
})

const compUtilizationRate = computed(() => `${formState.utilizationRate * 100}%`)

const defaultIndex = computed(() => columns.findIndex((item) => item === formState.utilizationRate))

const validatorMessage = (val, field) => (formState[field] || val ? true : '押金与收费任一可为空 不能同时为空')

const onSubmit = () => {
  resultVisible.value = true
}
const onConfirm = (value) => {
  formState.utilizationRate = value
  utilizationRatePicker.value = false
}
</script>

<template>
  <van-form @submit="onSubmit" class="py-4 flex flex-col gap-y-4">
    <van-cell-group inset>
      <van-field
        v-model="formState.quantity"
        input-align="right"
        :rules="[{ required: true, message: '投放柜子数量不能为空' }]"
        clearable
        type="digit"
        name="quantity"
        label="投放柜子数量"
        placeholder="请输入投放柜子数量(个)"
      >
        <template #right-icon>个</template>
      </van-field>
    </van-cell-group>
    <van-cell-group inset>
      <van-field
        v-model="formState.deposit"
        :rules="[{ validator: (v) => validatorMessage(v, 'charge') }]"
        input-align="right"
        clearable
        type="number"
        name="deposit"
        label="押金金额"
        placeholder="请输入押金金额(元)"
      >
        <template #right-icon>元</template>
      </van-field>
      <van-field
        :border="false"
        :rules="[{ validator: (v) => validatorMessage(v, 'deposit'), message: '此项必填' }]"
        v-model="formState.charge"
        input-align="right"
        clearable
        type="number"
        name="charge"
        label="收费价格"
        placeholder="请输入收费价格(元)"
      >
        <template #right-icon>元</template>
      </van-field>
      <div class="text-xs text-orange-500 px-4 pb-2">押金与收费任一可为空 不能同时为空</div>
    </van-cell-group>
    <van-cell-group inset>
      <van-field v-model="formState.sharing" input-align="right" clearable type="number" name="sharing" label="代理每单分成" placeholder="请输入代理每单分成金额">
        <template #right-icon>元</template>
      </van-field>
      <van-field v-model="formState.rent" input-align="right" clearable type="number" name="rent" label="租金" placeholder="请输入租金(每月多少元)">
        <template #right-icon>元</template>
      </van-field>
    </van-cell-group>
    <van-cell-group inset>
      <van-field @click="utilizationRatePicker = true" input-align="right" is-link readonly :model-value="compUtilizationRate" name="utilizationRate" label="使用率" placeholder="请选择(默认50%)" />
    </van-cell-group>
    <van-button round class="mx-4 box-border" type="primary" native-type="submit"> 计算 </van-button>
  </van-form>
  <van-popup v-model:show="utilizationRatePicker" position="bottom">
    <van-picker title="使用率" :defaultIndex="defaultIndex" :columns="columns" @confirm="onConfirm" @cancel="utilizationRatePicker = false">
      <template #option="record"> {{ record * 100 }} % </template>
    </van-picker>
  </van-popup>

  <ResultView v-model:visible="resultVisible" :record="formState" />
</template>
