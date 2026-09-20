<template>
  <div class="product-page">
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
            :class="{
              active: !compareMode && activeTab === product.id,
              'compare-selected': compareMode && compareSelection.includes(product.id),
              'compare-muted': compareMode && !comparableIds.includes(product.id)
            }"
            @click="handleTabClick(product)"
          >
            <el-icon :size="18"><component :is="product.icon" /></el-icon>
            {{ product.shortTitle }}
            <el-icon v-if="compareMode && compareSelection.includes(product.id)" :size="14"><Check /></el-icon>
          </button>
          <button
            class="tab-btn compare-toggle"
            :class="{ active: compareMode }"
            @click="toggleCompareMode"
          >
            <el-icon :size="18"><Switch /></el-icon>
            {{ compareMode ? '退出对照' : '方案对照' }}
          </button>
        </div>
        <p v-if="compareMode" class="compare-hint">
          请选择两项方案进行对照（已选 {{ compareSelection.length }}/{{ compareMax }}），支持：智慧仓储、运输管理、配送调度
        </p>
      </div>
    </section>
    
    <!-- 方案对照 -->
    <section v-if="compareMode" class="section section-light compare-section">
      <div class="container">
        <SectionTitle
          title="方案对照"
          subtitle="对照关键能力、技术优势与服务流程，选出最适合您的方案"
        />
        <div v-if="!isCompareReady" class="compare-empty">
          <el-icon :size="40"><Switch /></el-icon>
          <p>请从上方选择两项方案开始对照（已选 {{ compareSelection.length }}/{{ compareMax }}）</p>
          <p class="compare-empty-sub">支持对照的方案：智慧仓储、运输管理、配送调度</p>
        </div>
        <template v-else>
          <div class="compare-grid">
            <div class="compare-col" v-for="product in compareProducts" :key="product.id">
              <div class="compare-col-header" :style="{ background: product.gradient }">
                <el-icon :size="28"><component :is="product.icon" /></el-icon>
                <div class="compare-col-title">
                  <h3>{{ product.title }}</h3>
                  <span>{{ product.tag }}</span>
                </div>
              </div>
              <div class="compare-block">
                <h4 class="compare-block-title">关键能力</h4>
                <ul class="compare-list">
                  <li v-for="feature in product.features" :key="feature.title">
                    <strong>{{ feature.title }}</strong>
                    <span>{{ feature.desc }}</span>
                  </li>
                </ul>
              </div>
              <div class="compare-block">
                <h4 class="compare-block-title">技术优势</h4>
                <ul class="compare-list">
                  <li v-for="tech in product.technologies" :key="tech.title">
                    <strong>{{ tech.title }}</strong>
                    <span>{{ tech.description }}</span>
                  </li>
                </ul>
              </div>
              <div class="compare-block">
                <h4 class="compare-block-title">服务流程</h4>
                <ol class="compare-steps">
                  <li v-for="(step, index) in product.steps" :key="step.title">
                    <em>{{ index + 1 }}</em>
                    <div>
                      <strong>{{ step.title }}</strong>
                      <span>{{ step.description }}</span>
                    </div>
                  </li>
                </ol>
              </div>
            </div>
            <div class="compare-vs">VS</div>
          </div>
          <div class="compare-actions">
            <el-button type="primary" size="large" @click="goConsult">
              带着对照结果去咨询
              <el-icon class="el-icon--right"><ArrowRight /></el-icon>
            </el-button>
            <el-button size="large" @click="exitCompare">退出对照</el-button>
          </div>
        </template>
      </div>
    </section>

    <!-- 产品列表 -->
    <section v-if="!compareMode" class="section section-light">
      <div class="container">
        <div
          v-for="product in products"
          :key="product.id"
          :id="`product-${product.id}`"
          class="product-detail"
          :class="{ 'product-highlight': activeTab === product.id }"
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
            <el-button type="primary" size="large" @click="$router.push('/contact')">
              获取方案
              <el-icon class="el-icon--right"><ArrowRight /></el-icon>
            </el-button>
          </div>
          <div class="product-image">
            <div class="image-placeholder" :style="{ background: product.gradient }">
              <el-icon :size="80">
                <component :is="product.icon" />
              </el-icon>
            </div>
          </div>
        </div>
      </div>
    </section>
    
    <!-- 技术优势 -->
    <section v-if="!compareMode" class="section section-gray">
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
    <section v-if="!compareMode" class="section section-light">
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
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import SectionTitle from '@/components/SectionTitle.vue'
import { products, comparableIds, parseCompareIds, COMPARE_MAX } from '@/data/products'

const route = useRoute()
const router = useRouter()

const compareMax = COMPARE_MAX

const activeTab = ref('wms')

// 方案对照状态
const compareMode = ref(false)
const compareSelection = ref([])

const compareProducts = computed(() => {
  if (!compareMode.value || compareSelection.value.length !== COMPARE_MAX) return []
  return compareSelection.value
    .map(id => products.find(p => p.id === id))
    .filter(Boolean)
})

const isCompareReady = computed(() => compareProducts.value.length === COMPARE_MAX)

const activeProduct = computed(() => {
  return products.find(p => p.id === activeTab.value) || products[0]
})

const activeTechnologies = computed(() => {
  return activeProduct.value.technologies
})

const activeSteps = computed(() => {
  return activeProduct.value.steps
})

const scrollToProduct = (productId) => {
  nextTick(() => {
    const el = document.getElementById(`product-${productId}`)
    if (el) {
      const top = el.getBoundingClientRect().top + window.scrollY - 80
      window.scrollTo({ top, behavior: 'smooth' })
    }
  })
}

const switchTab = (productId) => {
  activeTab.value = productId
  router.replace({ query: { tab: productId }, hash: `#product-${productId}` })
  scrollToProduct(productId)
}

// 将对照选择同步到地址栏，刷新 / 前进后退可恢复
const syncCompareQuery = () => {
  const query = { ...route.query }
  if (compareSelection.value.length > 0) {
    query.compare = compareSelection.value.join(',')
  } else {
    delete query.compare
  }
  router.replace({ query })
}

const clearCompareState = () => {
  compareMode.value = false
  compareSelection.value = []
}

const removeCompareQuery = () => {
  if (route.query.compare === undefined) return
  const query = { ...route.query }
  delete query.compare
  router.replace({ query })
}

const enterCompare = () => {
  compareSelection.value = []
  compareMode.value = true
}

const exitCompare = () => {
  clearCompareState()
  removeCompareQuery()
}

const toggleCompareMode = () => {
  if (compareMode.value) {
    exitCompare()
  } else {
    enterCompare()
  }
}

// 选择对照项：再次点击取消；超出上限时替换最早的选择，保证连续切换不残留
const toggleCompareSelection = (productId) => {
  const index = compareSelection.value.indexOf(productId)
  if (index > -1) {
    compareSelection.value.splice(index, 1)
  } else {
    if (compareSelection.value.length >= COMPARE_MAX) {
      compareSelection.value.shift()
    }
    compareSelection.value.push(productId)
  }
  syncCompareQuery()
}

const handleTabClick = (product) => {
  if (!compareMode.value) {
    switchTab(product.id)
    return
  }
  if (comparableIds.includes(product.id)) {
    toggleCompareSelection(product.id)
  } else {
    // 点击不参与对照的产品：退出对照并切回单项
    exitCompare()
    switchTab(product.id)
  }
}

// 带着对照结果跳转咨询入口
const goConsult = () => {
  if (!isCompareReady.value) return
  router.push({
    path: '/contact',
    query: { compare: compareSelection.value.join(',') }
  })
}

onMounted(() => {
  const tabFromQuery = route.query.tab
  const hashFromUrl = route.hash ? route.hash.replace('#product-', '') : ''
  const initialTab = tabFromQuery || hashFromUrl
  if (initialTab && products.some(p => p.id === initialTab)) {
    activeTab.value = initialTab
    scrollToProduct(initialTab)
  }
})

watch(() => route.query.tab, (newTab) => {
  if (newTab && products.some(p => p.id === newTab) && newTab !== activeTab.value) {
    activeTab.value = newTab
    scrollToProduct(newTab)
  }
})

// 对照参数变化（含初始加载）：有效则恢复对照项，无效则清空状态不残留高亮
watch(() => route.query.compare, (raw) => {
  if (raw === undefined) {
    clearCompareState()
    return
  }
  const ids = parseCompareIds(raw)
  if (ids.length === 0) {
    clearCompareState()
    removeCompareQuery()
    return
  }
  compareMode.value = true
  compareSelection.value = ids
}, { immediate: true })
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

  &.compare-selected {
    background: $warning-color;
    border-color: $warning-color;
    color: #fff;
  }

  &.compare-muted {
    opacity: 0.45;
  }
}

.compare-toggle {
  border-style: dashed;
}

.compare-hint {
  margin-top: $spacing-md;
  font-size: $font-size-sm;
  color: rgba(255, 255, 255, 0.75);
}

.compare-section {
  min-height: 400px;
}

.compare-empty {
  text-align: center;
  color: $text-secondary;
  padding: $spacing-xxl 0;

  .el-icon {
    color: $text-placeholder;
    margin-bottom: $spacing-md;
  }

  p {
    font-size: $font-size-base;
  }
}

.compare-empty .compare-empty-sub {
  margin-top: $spacing-xs;
  font-size: $font-size-sm;
  color: $text-placeholder;
}

.compare-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: $spacing-xl;
  position: relative;
}

.compare-col {
  background: $bg-white;
  border-radius: $radius-lg;
  box-shadow: $shadow-md;
  overflow: hidden;
}

.compare-col-header {
  display: flex;
  align-items: center;
  gap: $spacing-md;
  padding: $spacing-lg $spacing-xl;
  color: #fff;

  h3 {
    font-size: $font-size-lg;
    font-weight: 600;
  }

  span {
    font-size: $font-size-xs;
    opacity: 0.85;
  }
}

.compare-block {
  padding: $spacing-lg $spacing-xl;
  border-bottom: 1px solid $border-light;

  &:last-child {
    border-bottom: none;
  }
}

.compare-block-title {
  font-size: $font-size-base;
  color: $primary-color;
  font-weight: 600;
  margin-bottom: $spacing-md;
  padding-left: $spacing-sm;
  border-left: 3px solid $primary-color;
}

.compare-list {
  list-style: none;

  li {
    display: flex;
    flex-direction: column;
    margin-bottom: $spacing-sm;

    &:last-child {
      margin-bottom: 0;
    }

    strong {
      font-size: $font-size-sm;
      color: $text-primary;
    }

    span {
      font-size: $font-size-xs;
      color: $text-secondary;
      line-height: $line-height-base;
    }
  }
}

.compare-steps {
  list-style: none;

  li {
    display: flex;
    gap: $spacing-sm;
    margin-bottom: $spacing-sm;

    &:last-child {
      margin-bottom: 0;
    }

    em {
      flex: 0 0 22px;
      width: 22px;
      height: 22px;
      background: rgba($primary-color, 0.1);
      color: $primary-color;
      border-radius: 50%;
      font-size: $font-size-xs;
      font-style: normal;
      font-weight: 600;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-top: 2px;
    }

    strong {
      display: block;
      font-size: $font-size-sm;
      color: $text-primary;
    }

    span {
      font-size: $font-size-xs;
      color: $text-secondary;
      line-height: $line-height-base;
    }
  }
}

.compare-vs {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 56px;
  height: 56px;
  background: linear-gradient(135deg, $warning-color, #fbb034);
  color: #fff;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: $font-size-lg;
  font-weight: 700;
  box-shadow: $shadow-lg;
  z-index: 1;
}

.compare-actions {
  display: flex;
  justify-content: center;
  gap: $spacing-md;
  margin-top: $spacing-xl;
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

.product-image {
  flex: 0 0 450px;
}

.image-placeholder {
  width: 100%;
  height: 350px;
  border-radius: $radius-lg;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
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

  .product-highlight {
    margin: 0;
  }

  .compare-grid {
    grid-template-columns: 1fr;
    gap: $spacing-lg;
  }

  .compare-vs {
    display: none;
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
}
</style>
