<template>
  <div class="product-page" :class="{ 'has-compare-bar': compareIds.length > 0 }">
    <!-- 页面头部 -->
    <section class="page-header">
      <div class="container">
        <h1 class="page-title">产品服务</h1>
        <p class="page-subtitle">全方位智慧物流解决方案，助力企业数字化转型</p>
        <div class="product-tabs">
          <button
            v-for="product in products"
            :key="product.id"
            class="tab-btn"
            :class="{ active: activeTab === product.id }"
            @click="switchTab(product.id)"
          >
            <el-icon :size="18"><component :is="product.icon" /></el-icon>
            {{ product.shortTitle }}
            <span v-if="isComparing(product.id)" class="tab-compare-badge">对照中</span>
          </button>
        </div>
      </div>
    </section>

    <!-- 产品列表 -->
    <section class="section section-light">
      <div class="container">
        <div
          v-for="product in products"
          :key="product.id"
          :id="`product-${product.id}`"
          class="product-detail"
          :class="{
            'product-highlight': activeTab === product.id,
            'compare-selected': isComparing(product.id)
          }"
        >
          <div class="product-content" :class="{ 'order-2': product.reverse }">
            <div class="product-tag">{{ product.tag }}</div>
            <h2 class="product-title">{{ product.title }}</h2>
            <p class="product-desc">{{ product.description }}</p>
            <div class="product-features">
              <div class="feature-item" v-for="feature in product.features" :key="feature.title">
                <el-icon :size="20"><Check /></el-icon>
                <div>
                  <h4>{{ feature.title }}</h4>
                  <p>{{ feature.desc }}</p>
                </div>
              </div>
            </div>
            <div class="product-actions">
              <el-button type="primary" size="large" @click="$router.push('/contact')">
                获取方案
                <el-icon class="el-icon--right"><ArrowRight /></el-icon>
              </el-button>
              <el-button
                v-if="product.comparable"
                size="large"
                plain
                class="compare-btn"
                :class="{ 'is-selected': isComparing(product.id) }"
                :disabled="!isComparing(product.id) && compareIds.length >= 2"
                @click="toggleCompare(product.id)"
              >
                <el-icon class="el-icon--left">
                  <CircleCheck v-if="isComparing(product.id)" />
                  <ScaleToOriginal v-else />
                </el-icon>
                {{ isComparing(product.id) ? '已加入对照' : '加入方案对照' }}
              </el-button>
            </div>
          </div>
          <div class="product-image">
            <div
              class="image-placeholder"
              :class="{ 'compare-image-flag': isComparing(product.id) }"
              :style="{ background: product.gradient }"
            >
              <el-icon :size="80">
                <component :is="product.icon" />
              </el-icon>
              <span v-if="isComparing(product.id)" class="image-compare-tag">
                <el-icon><CircleCheckFilled /></el-icon>
                对照方案 {{ compareIndex(product.id) + 1 }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- 方案对照 -->
    <section id="compare-section" class="section section-gray compare-section">
      <div class="container">
        <SectionTitle
          title="方案对照"
          subtitle="任选两项核心方案，对照关键能力、技术优势与服务流程，选择最适合的组合"
        />

        <!-- 未选满两项：选择面板 -->
        <div v-if="compareIds.length < 2" class="compare-picker card-panel">
          <div class="picker-header">
            <el-icon :size="22"><ScaleToOriginal /></el-icon>
            <h3>选择要对照的方案（{{ compareIds.length }}/2）</h3>
          </div>
          <div class="picker-chips">
            <button
              v-for="product in comparableProducts"
              :key="product.id"
              class="picker-chip"
              :class="{ selected: isComparing(product.id) }"
              @click="toggleCompare(product.id)"
            >
              <span class="chip-icon" :style="{ background: product.gradient }">
                <el-icon :size="20"><component :is="product.icon" /></el-icon>
              </span>
              <span class="chip-text">
                <strong>{{ product.shortTitle }}</strong>
                <em>{{ product.title }}</em>
              </span>
              <span class="chip-state">
                <el-icon><CircleCheck v-if="isComparing(product.id)" /><Plus v-else /></el-icon>
                {{ isComparing(product.id) ? '已选择' : '选择' }}
              </span>
            </button>
          </div>
          <p v-if="compareIds.length === 1" class="picker-tip">
            还需再选择 1 项方案，即可查看完整对照
          </p>
          <p v-else class="picker-tip picker-tip-empty">
            支持「智慧仓储」「运输管理」「配送调度」三项核心方案两两对照
          </p>
        </div>

        <!-- 已选满两项：对照内容（桌面端表格） -->
        <template v-else>
          <div class="compare-table">
            <!-- 方案头 -->
            <div class="compare-row compare-head-row">
              <div class="compare-cell compare-label-cell"></div>
              <div
                v-for="(id, slot) in compareIds"
                :key="`head-${id}`"
                class="compare-cell compare-head-cell"
              >
                <div class="head-icon" :style="{ background: compareProducts[slot].gradient }">
                  <el-icon :size="26"><component :is="compareProducts[slot].icon" /></el-icon>
                </div>
                <h3>{{ compareProducts[slot].shortTitle }}</h3>
                <p>{{ compareProducts[slot].title }}</p>
                <el-select
                  :model-value="id"
                  class="head-select"
                  @update:model-value="(val) => switchCompareSlot(slot, val)"
                >
                  <el-option
                    v-for="opt in slotOptions(slot)"
                    :key="opt.id"
                    :label="opt.shortTitle"
                    :value="opt.id"
                    :disabled="opt.disabled"
                  />
                </el-select>
              </div>
            </div>

            <!-- 方案简介 -->
            <div class="compare-row">
              <div class="compare-cell compare-label-cell">方案简介</div>
              <div v-for="product in compareProducts" :key="`desc-${product.id}`" class="compare-cell">
                {{ product.description }}
              </div>
            </div>

            <!-- 关键能力 -->
            <template v-for="group in compareRows" :key="group.title">
              <div class="compare-row compare-group-row">
                <div class="compare-cell compare-group-cell">
                  <el-icon><component :is="group.icon" /></el-icon>
                  {{ group.title }}
                </div>
              </div>
              <div v-for="item in group.items" :key="group.title + item.label" class="compare-row">
                <div class="compare-cell compare-label-cell">{{ item.label }}</div>
                <div v-for="(id, slot) in compareIds" :key="group.title + item.label + id" class="compare-cell">
                  {{ item.render(compareProducts[slot]) }}
                </div>
              </div>
            </template>
          </div>

          <!-- 已选满两项：对照内容（移动端卡片） -->
          <div class="compare-cards">
            <div v-for="product in compareProducts" :key="`card-${product.id}`" class="compare-card">
              <div class="compare-card-head">
                <div class="head-icon" :style="{ background: product.gradient }">
                  <el-icon :size="24"><component :is="product.icon" /></el-icon>
                </div>
                <div>
                  <h3>{{ product.shortTitle }}</h3>
                  <p>{{ product.title }}</p>
                </div>
              </div>
              <div class="compare-card-body">
                <p class="card-desc">{{ product.description }}</p>
                <template v-for="group in compareRows" :key="'card-' + group.title">
                  <h4 class="card-group-title">
                    <el-icon><component :is="group.icon" /></el-icon>
                    {{ group.title }}
                  </h4>
                  <div v-for="item in group.items" :key="'card-' + group.title + item.label" class="card-row">
                    <span class="card-row-label">{{ item.label }}</span>
                    <span class="card-row-value">{{ item.render(product) }}</span>
                  </div>
                </template>
              </div>
            </div>
          </div>

          <!-- 带对照结果去咨询 -->
          <div class="compare-cta">
            <el-button size="large" @click="clearCompare">
              清空对照
            </el-button>
            <el-button type="primary" size="large" @click="goContactWithCompare">
              带着对照结果去咨询
              <el-icon class="el-icon--right"><ArrowRight /></el-icon>
            </el-button>
          </div>
        </template>
      </div>
    </section>

    <!-- 技术优势 -->
    <section class="section section-light">
      <div class="container">
        <SectionTitle
          title="技术优势"
          subtitle="领先的技术架构，保障系统稳定高效运行"
        />
        <div class="tech-grid">
          <div class="tech-card" v-for="tech in activeTechnologies" :key="tech.title">
            <div class="tech-icon">
              <el-icon :size="32">
                <component :is="tech.icon" />
              </el-icon>
            </div>
            <h3>{{ tech.title }}</h3>
            <p>{{ tech.description }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- 服务流程 -->
    <section class="section section-gray">
      <div class="container">
        <SectionTitle
          title="服务流程"
          subtitle="专业规范的服务流程，确保项目顺利交付"
        />
        <div class="process-steps">
          <div class="step-item" v-for="(step, index) in activeSteps" :key="step.title">
            <div class="step-number">{{ index + 1 }}</div>
            <div class="step-content">
              <h4>{{ step.title }}</h4>
              <p>{{ step.description }}</p>
            </div>
            <div class="step-arrow" v-if="index < activeSteps.length - 1">
              <el-icon><ArrowRight /></el-icon>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- CTA -->
    <section class="section cta-section">
      <div class="container text-center">
        <h2 class="cta-title">需要定制化解决方案？</h2>
        <p class="cta-desc">我们的专家团队将为您提供一对一咨询服务</p>
        <el-button type="primary" size="large" round @click="$router.push('/contact')">
          立即咨询
          <el-icon class="el-icon--right"><ArrowRight /></el-icon>
        </el-button>
      </div>
    </section>

    <!-- 底部对照悬浮条 -->
    <transition name="bar-slide">
      <div v-if="compareIds.length > 0" class="compare-bar">
        <div class="container compare-bar-inner">
          <div class="bar-products">
            <span class="bar-label">方案对照（{{ compareIds.length }}/2）：</span>
            <template v-if="compareIds.length">
              <span
                v-for="(id, index) in compareIds"
                :key="`bar-${id}`"
                class="bar-product"
              >
                <el-icon class="bar-product-icon"><component :is="getProductById(id).icon" /></el-icon>
                {{ getProductById(id).shortTitle }}
                <el-icon class="bar-product-remove" title="移除" @click="toggleCompare(id)">
                  <Close />
                </el-icon>
                <span v-if="index < compareIds.length - 1" class="bar-vs">VS</span>
              </span>
            </template>
          </div>
          <div class="bar-actions">
            <el-button size="default" @click="clearCompare">清空</el-button>
            <el-button
              size="default"
              :disabled="compareIds.length < 2"
              @click="scrollToCompare"
            >
              查看对照
            </el-button>
            <el-button
              type="primary"
              size="default"
              :disabled="compareIds.length < 2"
              @click="goContactWithCompare"
            >
              带对照结果咨询
            </el-button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import SectionTitle from '@/components/SectionTitle.vue'
import {
  products,
  comparableProducts,
  getProductById,
  parseCompareIds,
  buildContactQuery
} from '@/data/products.js'

const route = useRoute()
const router = useRouter()

const activeTab = ref('wms')

const activeProduct = computed(() => {
  return products.find(p => p.id === activeTab.value) || products[0]
})

const activeTechnologies = computed(() => {
  return activeProduct.value.technologies
})

const activeSteps = computed(() => {
  return activeProduct.value.steps
})

// 对照项始终从 URL 派生（单一数据源：刷新 / 前进后退天然恢复，状态切换无残留）
const compareIds = computed(() => parseCompareIds(route.query.compare))
const compareProducts = computed(() => compareIds.value.map(getProductById))

const isComparing = (id) => compareIds.value.includes(id)
const compareIndex = (id) => compareIds.value.indexOf(id)

// 对照维度：关键能力 / 技术优势 / 服务流程
const compareRows = computed(() => [
  {
    title: '关键能力',
    icon: 'Star',
    items: [
      { label: '核心能力一', render: (p) => p.features[0].title },
      { label: '核心能力二', render: (p) => p.features[1].title },
      { label: '核心能力三', render: (p) => p.features[2].title },
      { label: '核心能力四', render: (p) => p.features[3].title },
      { label: '能力说明', render: (p) => p.features.map(f => f.desc).join('；') }
    ]
  },
  {
    title: '技术优势',
    icon: 'Cpu',
    items: [
      { label: '智能引擎', render: (p) => p.technologies[0].title },
      { label: '引擎说明', render: (p) => p.technologies[0].description },
      { label: '集成互联', render: (p) => `${p.technologies[1].title}：${p.technologies[1].description}` },
      { label: '安全保障', render: (p) => `${p.technologies[2].title}：${p.technologies[2].description}` },
      { label: '监控能力', render: (p) => `${p.technologies[3].title}：${p.technologies[3].description}` }
    ]
  },
  {
    title: '服务流程',
    icon: 'List',
    items: [
      { label: '第一步', render: (p) => `${p.steps[0].title}（${p.steps[0].description}）` },
      { label: '第二步', render: (p) => `${p.steps[1].title}（${p.steps[1].description}）` },
      { label: '第三步', render: (p) => `${p.steps[2].title}（${p.steps[2].description}）` },
      { label: '第四步', render: (p) => `${p.steps[3].title}（${p.steps[3].description}）` },
      { label: '第五步', render: (p) => `${p.steps[4].title}（${p.steps[4].description}）` }
    ]
  }
])

const scrollToProduct = (productId) => {
  nextTick(() => {
    const el = document.getElementById(`product-${productId}`)
    if (el) {
      const top = el.getBoundingClientRect().top + window.scrollY - 80
      window.scrollTo({ top, behavior: 'smooth' })
    }
  })
}

const scrollToCompare = () => {
  nextTick(() => {
    const el = document.getElementById('compare-section')
    if (el) {
      const top = el.getBoundingClientRect().top + window.scrollY - 80
      window.scrollTo({ top, behavior: 'smooth' })
    }
  })
}

// 仅更新对照参数，保留 tab / hash
const updateCompareQuery = async (ids) => {
  const query = { ...route.query }
  if (ids.length) {
    query.compare = ids.join(',')
  } else {
    delete query.compare
  }
  await router.push({ query, hash: route.hash || undefined })
}

const toggleCompare = async (productId) => {
  if (!comparableProducts.some(p => p.id === productId)) return
  const ids = [...compareIds.value]
  const index = ids.indexOf(productId)
  if (index > -1) {
    ids.splice(index, 1)
  } else {
    if (ids.length >= 2) {
      ElMessage.warning('最多只能选择两项方案进行对照，请先取消一项')
      return
    }
    ids.push(productId)
  }
  await updateCompareQuery(ids)
  // 选满两项后自动定位到对照区
  if (ids.length === 2) {
    scrollToCompare()
  }
}

// 对照表内下拉切换某一列的方案
const switchCompareSlot = async (slot, newId) => {
  if (compareIds.value[slot] === newId) return
  if (compareIds.value.includes(newId)) return
  const ids = [...compareIds.value]
  ids[slot] = newId
  await updateCompareQuery(ids)
}

// 下拉候选项：另一列已选的方案禁用，避免重复
const slotOptions = (slot) =>
  comparableProducts.map(p => ({
    ...p,
    disabled: compareIds.value.some((id, i) => id === p.id && i !== slot)
  }))

const clearCompare = async () => {
  await updateCompareQuery([])
}

const goContactWithCompare = () => {
  if (compareIds.value.length !== 2) return
  router.push({ path: '/contact', query: buildContactQuery(compareIds.value) })
}

const switchTab = (productId) => {
  activeTab.value = productId
  // 保留已有对照参数，避免切标签清空选择
  router.replace({
    query: { ...route.query, tab: productId },
    hash: `#product-${productId}`
  })
  scrollToProduct(productId)
}

onMounted(() => {
  const tabFromQuery = route.query.tab
  const hashFromUrl = route.hash ? route.hash.replace('#product-', '') : ''
  const initialTab = tabFromQuery || hashFromUrl
  if (initialTab && products.some(p => p.id === initialTab)) {
    activeTab.value = initialTab
    scrollToProduct(initialTab)
  }

  // 从咨询页「返回调整对照」时定位到对照区
  if (route.hash === '#compare-section') {
    scrollToCompare()
  }

  // 规整无效对照参数（含非法 id、重复、超过两项等情况）
  const normalized = parseCompareIds(route.query.compare)
  if (String(route.query.compare || '') !== normalized.join(',')) {
    const query = { ...route.query }
    if (normalized.length) query.compare = normalized.join(',')
    else delete query.compare
    router.replace({ query, hash: route.hash || undefined })
  }
})

watch(() => route.query.tab, (newTab) => {
  if (newTab && products.some(p => p.id === newTab) && newTab !== activeTab.value) {
    activeTab.value = newTab
    scrollToProduct(newTab)
  }
})

// 前进 / 后退或外部改 URL 后，规整无效对照参数
watch(() => route.query.compare, (raw) => {
  const normalized = parseCompareIds(raw)
  if (String(raw || '') !== normalized.join(',')) {
    const query = { ...route.query }
    if (normalized.length) query.compare = normalized.join(',')
    else delete query.compare
    router.replace({ query, hash: route.hash || undefined })
  }
})
</script>

<style lang="scss" scoped>
@use '@/assets/styles/variables.scss' as *;

.page-header {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  padding: $spacing-xxl 0;
  text-align: center;
  color: #fff;
}

.page-title {
  font-size: $font-size-xxxl;
  font-weight: 700;
  margin-bottom: $spacing-sm;
}

.page-subtitle {
  font-size: $font-size-lg;
  opacity: 0.75;
  margin-bottom: $spacing-xl;
}

.product-tabs {
  display: flex;
  justify-content: center;
  gap: $spacing-md;
  flex-wrap: wrap;
}

.tab-btn {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 10px 24px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: $radius-lg;
  background: transparent;
  color: rgba(255, 255, 255, 0.7);
  font-size: $font-size-base;
  cursor: pointer;
  transition: all 0.3s;

  &:hover {
    border-color: rgba(255, 255, 255, 0.6);
    color: #fff;
    background: rgba(255, 255, 255, 0.08);
  }

  &.active {
    background: $primary-color;
    border-color: $primary-color;
    color: #fff;
  }
}

.tab-compare-badge {
  margin-left: 4px;
  padding: 1px 8px;
  border-radius: $radius-sm;
  background: $warning-color;
  color: #fff;
  font-size: $font-size-xs;
  line-height: 18px;
}

.product-detail {
  display: flex;
  gap: $spacing-xxl;
  align-items: center;
  padding: $spacing-xxl 0;
  border-bottom: 1px solid $border-light;
  transition: background 0.3s;

  &:last-child {
    border-bottom: none;
  }
}

.product-highlight {
  background: rgba($primary-color, 0.03);
  border-radius: $radius-lg;
  padding: $spacing-xxl;
  margin: 0 (-$spacing-xl);
}

// 被选中对照的方案区块（与标签高亮相互独立，状态移除即消失，无残留）
.compare-selected {
  background: rgba($warning-color, 0.06);
  border-radius: $radius-lg;
  padding: $spacing-xxl;
  margin: 0 (-$spacing-xl);
  box-shadow: inset 4px 0 0 $warning-color;
}

.product-highlight.compare-selected {
  background: linear-gradient(90deg, rgba($primary-color, 0.04) 0%, rgba($warning-color, 0.06) 100%);
}

.product-content {
  flex: 1;

  &.order-2 {
    order: 2;
  }
}

.product-tag {
  display: inline-block;
  background: rgba($primary-color, 0.1);
  color: $primary-color;
  padding: $spacing-xs $spacing-sm;
  border-radius: $radius-sm;
  font-size: $font-size-xs;
  font-weight: 500;
  margin-bottom: $spacing-md;
}

.product-title {
  font-size: $font-size-xxl;
  font-weight: 700;
  color: $text-primary;
  margin-bottom: $spacing-md;
}

.product-desc {
  font-size: $font-size-base;
  color: $text-secondary;
  line-height: $line-height-loose;
  margin-bottom: $spacing-lg;
}

.product-features {
  margin-bottom: $spacing-xl;
}

.feature-item {
  display: flex;
  gap: $spacing-sm;
  margin-bottom: $spacing-md;

  .el-icon {
    color: $success-color;
    margin-top: 4px;
  }

  h4 {
    font-size: $font-size-base;
    color: $text-primary;
    margin-bottom: 2px;
  }

  p {
    font-size: $font-size-sm;
    color: $text-secondary;
  }
}

.product-actions {
  display: flex;
  gap: $spacing-md;
  flex-wrap: wrap;
}

.compare-btn {
  &.is-selected {
    color: $warning-color;
    background: rgba($warning-color, 0.1);
    border-color: $warning-color;

    &:hover,
    &:focus {
      color: #fff;
      background: $warning-color;
      border-color: $warning-color;
    }
  }

  &.is-disabled,
  &.is-disabled:hover {
    color: $text-placeholder;
    border-color: $border-light;
    background: $bg-color;
  }
}

.product-image {
  flex: 0 0 450px;
}

.image-placeholder {
  position: relative;
  width: 100%;
  height: 350px;
  border-radius: $radius-lg;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  transition: box-shadow 0.3s;

  &.compare-image-flag {
    box-shadow: 0 0 0 3px $warning-color;
  }
}

.image-compare-tag {
  position: absolute;
  top: $spacing-md;
  right: $spacing-md;
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 12px;
  border-radius: $radius-md;
  background: rgba(0, 0, 0, 0.35);
  backdrop-filter: blur(4px);
  font-size: $font-size-sm;
}

/* 方案对照区 */
.compare-section {
  scroll-margin-top: 80px;
}

.card-panel {
  background: $bg-white;
  border-radius: $radius-lg;
  box-shadow: $shadow-md;
  padding: $spacing-xl;
}

.picker-header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: $spacing-sm;
  margin-bottom: $spacing-lg;
  color: $text-primary;

  h3 {
    font-size: $font-size-lg;
    font-weight: 600;
  }
}

.picker-chips {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: $spacing-md;
}

.picker-chip {
  display: flex;
  align-items: center;
  gap: $spacing-sm;
  padding: $spacing-md;
  border: 2px solid $border-light;
  border-radius: $radius-lg;
  background: $bg-white;
  cursor: pointer;
  text-align: left;
  transition: all 0.25s;

  &:hover {
    border-color: $primary-light;
    transform: translateY(-2px);
  }

  &.selected {
    border-color: $warning-color;
    background: rgba($warning-color, 0.05);
  }
}

.chip-icon {
  flex: 0 0 44px;
  width: 44px;
  height: 44px;
  border-radius: $radius-md;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
}

.chip-text {
  flex: 1;
  display: flex;
  flex-direction: column;

  strong {
    font-size: $font-size-base;
    color: $text-primary;
  }

  em {
    font-style: normal;
    font-size: $font-size-xs;
    color: $text-secondary;
  }
}

.chip-state {
  display: inline-flex;
  align-items: center;
  gap: 2px;
  font-size: $font-size-xs;
  color: $text-secondary;

  .selected & {
    color: $warning-color;
  }
}

.picker-tip {
  margin-top: $spacing-md;
  text-align: center;
  font-size: $font-size-sm;
  color: $warning-color;

  &.picker-tip-empty {
    color: $text-secondary;
  }
}

/* 对照表（桌面端） */
.compare-table {
  display: grid;
  background: $bg-white;
  border-radius: $radius-lg;
  box-shadow: $shadow-md;
  overflow: hidden;
}

.compare-row {
  display: grid;
  grid-template-columns: 160px 1fr 1fr;
}

.compare-cell {
  padding: $spacing-md $spacing-lg;
  font-size: $font-size-sm;
  color: $text-regular;
  line-height: $line-height-loose;
  border-bottom: 1px solid $border-light;
  border-right: 1px solid $border-light;

  &:last-child {
    border-right: none;
  }
}

.compare-label-cell {
  background: $bg-color;
  color: $text-primary;
  font-weight: 600;
  display: flex;
  align-items: center;
}

.compare-head-row .compare-cell {
  border-bottom: 2px solid $primary-color;
}

.compare-head-cell {
  text-align: center;

  h3 {
    font-size: $font-size-lg;
    color: $text-primary;
    margin: $spacing-sm 0 2px;
  }

  p {
    font-size: $font-size-xs;
    color: $text-secondary;
    margin-bottom: $spacing-sm;
  }
}

.head-icon {
  width: 56px;
  height: 56px;
  margin: 0 auto;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
}

.head-select {
  width: 160px;
}

.compare-group-row {
  grid-template-columns: 1fr;

  .compare-group-cell {
    grid-column: 1 / -1;
    display: flex;
    align-items: center;
    gap: $spacing-xs;
    background: rgba($primary-color, 0.06);
    color: $primary-color;
    font-weight: 600;
    padding: $spacing-sm $spacing-lg;
    border-bottom: 1px solid $border-light;
    border-right: none;
  }
}

.compare-cards {
  display: none;
}

.compare-cta {
  display: flex;
  justify-content: center;
  gap: $spacing-md;
  margin-top: $spacing-xl;
}

/* 底部对照悬浮条 */
.compare-bar {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 100;
  background: rgba(26, 26, 46, 0.96);
  backdrop-filter: blur(8px);
  box-shadow: 0 -4px 20px rgba(0, 0, 0, 0.15);
  color: #fff;
}

.compare-bar-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: $spacing-md;
  padding-top: $spacing-md;
  padding-bottom: $spacing-md;
  flex-wrap: wrap;
}

.bar-products {
  display: flex;
  align-items: center;
  gap: $spacing-sm;
  flex-wrap: wrap;
}

.bar-label {
  font-size: $font-size-sm;
  color: rgba(255, 255, 255, 0.75);
}

.bar-product {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 14px;
  border-radius: $radius-md;
  background: rgba(255, 255, 255, 0.1);
  font-size: $font-size-sm;
}

.bar-product-icon {
  color: $warning-color;
}

.bar-product-remove {
  cursor: pointer;
  color: rgba(255, 255, 255, 0.55);
  transition: color 0.2s;

  &:hover {
    color: $danger-color;
  }
}

.bar-vs {
  position: absolute;
  right: -20px;
  font-size: $font-size-xs;
  font-weight: 700;
  color: $warning-color;
}

.bar-actions {
  display: flex;
  gap: $spacing-sm;
}

.bar-slide-enter-active,
.bar-slide-leave-active {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.bar-slide-enter-from,
.bar-slide-leave-to {
  transform: translateY(100%);
  opacity: 0;
}

.has-compare-bar {
  padding-bottom: 76px;
}

.tech-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: $spacing-lg;
}

.tech-card {
  background: $bg-white;
  padding: $spacing-xl;
  border-radius: $radius-lg;
  text-align: center;
  box-shadow: $shadow-md;
  transition: all 0.3s;

  &:hover {
    transform: translateY(-8px);
    box-shadow: $shadow-lg;
  }
}

.tech-icon {
  width: 64px;
  height: 64px;
  background: rgba($primary-color, 0.1);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto $spacing-md;
  color: $primary-color;
}

.tech-card h3 {
  font-size: $font-size-lg;
  color: $text-primary;
  margin-bottom: $spacing-sm;
}

.tech-card p {
  font-size: $font-size-sm;
  color: $text-secondary;
}

.process-steps {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.step-item {
  flex: 1;
  text-align: center;
  position: relative;
}

.step-number {
  width: 48px;
  height: 48px;
  background: $primary-color;
  color: #fff;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: $font-size-lg;
  font-weight: 700;
  margin: 0 auto $spacing-md;
}

.step-content {
  h4 {
    font-size: $font-size-base;
    color: $text-primary;
    margin-bottom: $spacing-xs;
  }

  p {
    font-size: $font-size-sm;
    color: $text-secondary;
    padding: 0 $spacing-sm;
  }
}

.step-arrow {
  position: absolute;
  right: -10px;
  top: 20px;
  color: $border-color;
}

.cta-section {
  background: linear-gradient(135deg, $primary-color, $primary-dark);
  color: #fff;
}

.cta-title {
  font-size: $font-size-xxl;
  font-weight: 700;
  margin-bottom: $spacing-md;
}

.cta-desc {
  font-size: $font-size-lg;
  opacity: 0.85;
  margin-bottom: $spacing-xl;
}

@media (max-width: $breakpoint-lg) {
  .product-detail {
    flex-direction: column;
  }

  .product-content.order-2 {
    order: 0;
  }

  .product-image {
    flex: none;
    width: 100%;
  }

  .tech-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .process-steps {
    flex-wrap: wrap;
    gap: $spacing-lg;
  }

  .step-item {
    flex: 0 0 calc(33.333% - 16px);
  }

  .step-arrow {
    display: none;
  }

  .product-highlight,
  .compare-selected {
    margin: 0;
    padding: $spacing-lg;
  }

  .picker-chips {
    grid-template-columns: 1fr;
  }

  /* 平板/移动端切换为卡片式对照 */
  .compare-table {
    display: none;
  }

  .compare-cards {
    display: block;
  }

  .compare-card {
    background: $bg-white;
    border-radius: $radius-lg;
    box-shadow: $shadow-md;
    overflow: hidden;
    margin-bottom: $spacing-md;
  }

  .compare-card-head {
    display: flex;
    align-items: center;
    gap: $spacing-md;
    padding: $spacing-md $spacing-lg;
    border-bottom: 2px solid $primary-color;

    h3 {
      font-size: $font-size-lg;
      color: $text-primary;
    }

    p {
      font-size: $font-size-xs;
      color: $text-secondary;
    }

    .head-icon {
      width: 48px;
      height: 48px;
      margin: 0;
    }
  }

  .compare-card-body {
    padding: $spacing-md $spacing-lg;
  }

  .card-desc {
    font-size: $font-size-sm;
    color: $text-secondary;
    line-height: $line-height-loose;
    margin-bottom: $spacing-md;
  }

  .card-group-title {
    display: flex;
    align-items: center;
    gap: 6px;
    font-size: $font-size-base;
    color: $primary-color;
    margin: $spacing-md 0 $spacing-sm;
  }

  .card-row {
    display: flex;
    gap: $spacing-md;
    padding: $spacing-xs 0;
    font-size: $font-size-sm;
    line-height: $line-height-base;
    border-bottom: 1px dashed $border-light;
  }

  .card-row-label {
    flex: 0 0 72px;
    color: $text-primary;
    font-weight: 600;
  }

  .card-row-value {
    flex: 1;
    color: $text-regular;
  }
}

@media (max-width: $breakpoint-md) {
  .page-title {
    font-size: $font-size-xxl;
  }

  .product-title {
    font-size: $font-size-xl;
  }

  .tech-grid {
    grid-template-columns: 1fr;
  }

  .step-item {
    flex: 0 0 100%;
  }

  .tab-btn {
    padding: 8px 16px;
    font-size: $font-size-sm;
  }

  .compare-bar-inner {
    justify-content: center;
  }

  .has-compare-bar {
    padding-bottom: 130px;
  }
}
</style>
