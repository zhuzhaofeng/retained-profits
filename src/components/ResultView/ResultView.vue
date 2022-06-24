<template>
  <van-action-sheet @open="handleOpen" :show="visible" title="计算结果" @cancel="handleBeforeClose" :before-close="handleBeforeClose">
    <!-- {{ result }} -->
    <div style="height: calc(100vh * 0.8)">
      <van-collapse v-model="activeNames">
        <van-collapse-item title="每天净收益" :value="`${formatValue(result.profit.daily)}元`">
          <van-cell-group :border="false" title="公式">
            <span class="font-mono break-normal">{{ calculationMethod('profit_daily') }}</span>
          </van-cell-group>
          <van-cell-group :border="false" title="数据">
            <span class="font-mono break-normal">{{ calculationMethodValue('profit_daily') }}</span>
          </van-cell-group>
        </van-collapse-item>
        <van-collapse-item title="每月净收益" :value="`${formatValue(result.profit.monthly)}元`">
          <van-cell-group :border="false" title="公式">
            <span class="font-mono break-normal">{{ calculationMethod('profit_monthly') }}</span>
          </van-cell-group>
          <van-cell-group :border="false" title="数据">
            <span class="font-mono break-normal">{{ calculationMethodValue('profit_monthly') }}</span>
          </van-cell-group>
        </van-collapse-item>
        <van-collapse-item v-if="isAgent" title="代理每天分成" :value="`${formatValue(result.sharing.daily)}元`">
          <van-cell-group :border="false" title="公式">
            <span class="font-mono break-normal">{{ calculationMethod('sharing_daily') }}</span>
          </van-cell-group>
          <van-cell-group :border="false" title="数据">
            <span class="font-mono break-normal">{{ calculationMethodValue('sharing_daily') }}</span>
          </van-cell-group>
        </van-collapse-item>
        <van-collapse-item v-if="isAgent" title="代理每月的分成" :value="`${formatValue(result.sharing.monthly)}元`">
          <van-cell-group :border="false" title="公式">
            <span class="font-mono break-normal">{{ calculationMethod('sharing_monthly') }}</span>
          </van-cell-group>
          <van-cell-group :border="false" title="数据">
            <span class="font-mono break-normal">{{ calculationMethodValue('sharing_monthly') }}</span>
          </van-cell-group>
        </van-collapse-item>
        <van-collapse-item title="预计回本时间" :value="`${formatValue(result.estimatedReturnTime)}天`">
          <div class="whitespace-pre-wrap">
            <van-cell-group :border="false" title="公式">
              <span class="font-mono break-normal">{{ calculationMethod('estimatedReturnTime') }}</span>
            </van-cell-group>
            <van-cell-group :border="false" title="数据">
              <span class="font-mono break-normal">{{ calculationMethodValue('estimatedReturnTime') }}</span>
            </van-cell-group>
          </div>
        </van-collapse-item>
      </van-collapse>
    </div>
  </van-action-sheet>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import { noAgent, withAgent } from './methods'

const emit = defineEmits(['update:visible'])
const props = defineProps({
  visible: { type: Boolean, default: false },
  record: { type: Object, required: true }
})

const calculationMethod = computed(() => (type) => {
  const res = isAgent.value ? withAgent[type] : noAgent[type]

  return res.replace(new RegExp('{{', 'gm'), '').replace(new RegExp('}}', 'gm'), '')
})

const calculationMethodValue = computed(() => (type) => {
  const res = isAgent.value ? withAgent[type] : noAgent[type]

  const { quantity, deposit, charge, sharing, rent, utilizationRate } = props.record
  // (柜子 x 16 x 使用率) x (押金-代理分成) x 60% + (柜子 x 16 x 使用率) x (收费价格 -代理分成) - 租金 / 30

  //  总成本
  const totalCost = (quantity % 5) * 2500 + (quantity - (quantity % 5)) * 1500
  return res
    .replace(new RegExp('{{柜子}}', 'gm'), quantity)
    .replace(new RegExp('{{押金}}', 'gm'), deposit)
    .replace(new RegExp('{{收费价格}}', 'gm'), charge)
    .replace(new RegExp('{{使用率}}', 'gm'), `${utilizationRate * 100}%`)
    .replace(new RegExp('{{代理分成}}', 'gm'), sharing)
    .replace(new RegExp('{{租金}}', 'gm'), rent)
    .replace(new RegExp('{{总成本}}', 'gm'), formatValue.value(totalCost))
    .replace(new RegExp('{{每天净利润}}', 'gm'), formatValue.value(result.profit.daily))
})

const activeNames = ref([])

const result = reactive({
  // 我们收益
  profit: {
    // 每天
    daily: 0,
    // 每月
    monthly: 0
  },
  // 分成
  sharing: {
    // 每天
    daily: 0,
    // 每月
    monthly: 0
  },
  // 预计回本时间
  estimatedReturnTime: 0
})

const formatValue = computed(() => (val) => {
  const res = Number(val).toFixed(2)
  return res.toString().replace('.00', '')
})

const isAgent = computed(() => props.record?.sharing && props.record?.sharing > 0)

// const formState = reactive({
//   // 柜子数量
//   // quantity: undefined,
//   quantity: 6,
//   // 押金金额
//   // deposit: undefined,
//   deposit: 15,
//   // 收费价格
//   // charge: undefined,
//   charge: 15,
//   // 分成金额
//   // sharing: undefined,
//   sharing: 1.5,
//   // 租金
//   // rent: undefined,
//   rent: 500,
//   // 使用率
//   utilizationRate: 0.5
// })

/**
 * 如果代理分成
  我们每天的净收益为：
      计算方式： (柜子 x 16 x 使用率) x (押金-代理分成) x 60% + (柜子 x 16 x 使用率) x (收费价格 -代理分成) - 租金 / 30
  我们每月的净收益为：
      计算方式： ((柜子 x 16 x 使用率) x (押金-代理分成) x 60% + (柜子 x 16 x 使用率) x (收费价格 -代理分成) - 租金 / 30) x 30
  代理每天的分成：
      计算方式： (柜子 x 16 x 使用率) x 代理分成 x 60% + (柜子 x 16 x 使用率) x 代理分成 - 租金 / 30
  代理每月的分成：
      ((柜子 x 16 x 使用率) x 代理分成 x 60% + (柜子 x 16 x 使用率) x 代理分成 - 租金 / 30) x 30
  我们预计回本时间：
    计算方式：
      总成本： 进一取整(柜子 % 5) x 2500 + (柜子 - 进一取整(柜子 % 5)) x 1500
      每天净利润：(柜子 x 16 x 使用率) x (押金-代理分成) x 60% + (柜子 x 16 x 使用率) x (收费价格 -代理分成) - 租金 / 30
      回本时间： 总成本 / 每天净利润
 *
 * 如果代理分成未填写
  我们每天的净收益为：
      计算方式： (柜子 x 16 x 使用率) x押金 x 60% + (柜子 x 16 x 使用率) x 收费价格 - 租金 / 30
  我们每月的净收益为：
      计算方式： ((柜子 x 16 x 使用率) x押金 x 60% + (柜子 x 16 x 使用率) x 收费价格 - 租金 / 30) x 30
  我们预计回本时间(天)：
      计算方式：
        总成本： 进一取整(柜子 % 5) x 2500 + (柜子 - 进一取整(柜子 % 5)) x 1500
        每天净利润：(柜子 x 16 x 使用率) x 押金 x 60% + (柜子 x 16 x 使用率) x 收费价格 - 租金 / 30
        回本时间： 总成本 / 每天净利润
 */
const computedResult = () => {
  const { quantity, deposit, charge, sharing, rent, utilizationRate } = props.record

  //  总成本
  const totalCost = Math.ceil(quantity % 5) * 2500 + (quantity - Math.ceil(quantity % 5)) * 1500
  if (isAgent.value) {
    // 我们每日净利润
    const profitDaily = quantity * 16 * utilizationRate * (deposit - sharing) * 0.6 + quantity * 16 * utilizationRate * (charge - sharing) - rent / 30
    result.profit.daily = profitDaily
    // 我们每月净利润
    result.profit.monthly = profitDaily * 30
    // 代理每天分成
    const agentDaily = quantity * 16 * utilizationRate * sharing * 0.6 + quantity * 16 * utilizationRate * sharing - rent / 30
    result.sharing.daily = agentDaily
    // 代理每月分成
    result.sharing.monthly = agentDaily * 30
    // 我们预计回本时间
    result.estimatedReturnTime = totalCost / profitDaily
    return
  }

  const profitDaily = quantity * 16 * utilizationRate * deposit * 0.6 + quantity * 16 * utilizationRate * charge - rent / 30
  result.profit.daily = profitDaily
  result.profit.monthly = profitDaily * 30

  result.estimatedReturnTime = totalCost / profitDaily
}

const handleBeforeClose = () => {
  emit('update:visible', false)
}

const handleOpen = () => {
  computedResult()
}
</script>

<style lang="less" scoped>
::v-deep(.van-collapse-item__content) {
  @apply flex flex-col;
  .van-cell-group__title {
    @apply pt-2 px-2 text-gray-600;
  }
  .van-cell-group {
    @apply px-2 bg-gray-100 py-1 rounded-md;
  }
}
</style>
