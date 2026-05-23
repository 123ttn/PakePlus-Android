<template>
  <view class="page">
    <!-- 头部 -->
    <view class="header safe-area-top">
      <text class="title">模块管理</text>
    </view>

    <!-- Tab栏 -->
    <view class="tab-bar">
      <view 
        :class="['tab-item', activeTab === 'market' ? 'active' : '']"
        @click="switchTab('market')"
      >
        <text>模板市场</text>
      </view>
      <view 
        :class="['tab-item', activeTab === 'my' ? 'active' : '']"
        @click="switchTab('my')"
      >
        <text>我的模块</text>
      </view>
    </view>

    <!-- 模板市场 -->
    <view class="tab-content" v-if="activeTab === 'market'">
      <view class="market-list">
        <view 
          v-for="(template, index) in marketTemplates" 
          :key="index"
          class="template-item"
          @click="selectTemplate(template)"
        >
          <view class="template-icon" :style="{ background: template.color }">
            <text>{{ getIcon(template.name) }}</text>
          </view>
          <view class="template-info">
            <text class="template-name">{{ template.name }}</text>
            <text class="template-count">{{ template.heroCount }} 个英雄</text>
          </view>
          <view class="template-action">
            <text class="action-text">选择</text>
          </view>
        </view>
      </view>

      <!-- 加载按钮 -->
      <view class="load-more" v-if="marketTemplates.length === 0">
        <button class="btn-load" @click="loadFromApi">
          从服务器加载模板
        </button>
      </view>
    </view>

    <!-- 我的模块 -->
    <view class="tab-content" v-if="activeTab === 'my'">
      <view class="empty-state" v-if="myModules.length === 0">
        <text class="empty-icon">📦</text>
        <text class="empty-text">还没有创建模块</text>
        <text class="empty-hint">从模板市场选择一个开始</text>
      </view>

      <view class="my-list" v-else>
        <view 
          v-for="(module, index) in myModules" 
          :key="index"
          class="module-item"
        >
          <view class="module-icon" :style="{ background: module.color || '#667eea' }">
            <text>{{ getIcon(module.name) }}</text>
          </view>
          <view class="module-info">
            <text class="module-name">{{ module.name }}</text>
            <text class="module-count">{{ module.heroes ? module.heroes.length : 0 }} 个英雄</text>
          </view>
          <view class="module-actions">
            <view class="action-btn" @click="useModule(module)">
              <text>使用</text>
            </view>
            <view class="action-btn delete" @click="deleteModule(index)">
              <text>删除</text>
            </view>
          </view>
        </view>
      </view>
    </view>

    <!-- 导入按钮 -->
    <view class="fab safe-area-bottom" @click="showImportModal">
      <text class="fab-icon">+</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      activeTab: 'market',
      marketTemplates: [],
      myModules: [],
      showImport: false
    }
  },
  onLoad() {
    this.loadMyModules()
  },
  onShow() {
    this.loadMyModules()
  },
  methods: {
    switchTab(tab) {
      this.activeTab = tab
    },
    getIcon(name) {
      return name && name.length > 0 ? name.charAt(0) : '?'
    },
    loadMyModules() {
      const modules = uni.getStorageSync('modules') || []
      this.myModules = modules
    },
    loadFromApi() {
      uni.showLoading({ title: '加载中...' })
      uni.request({
        url: 'http://localhost:9091/api/v1/templates',
        success: (res) => {
          uni.hideLoading()
          if (res.data && res.data.data) {
            this.marketTemplates = res.data.data.map(t => ({
              name: t.name,
              heroCount: t.heroes ? t.heroes.length : 0,
              heroes: t.heroes,
              color: this.getRandomColor()
            }))
            uni.showToast({
              title: '加载成功 ' + this.marketTemplates.length + ' 个',
              icon: 'success'
            })
          }
        },
        fail: () => {
          uni.hideLoading()
          uni.showToast({
            title: '加载失败，请检查网络',
            icon: 'none'
          })
        }
      })
    },
    getRandomColor() {
      const colors = ['#FF6B6B', '#4ECDC4', '#FFE66D', '#95E1D3', '#F38181', '#AA96DA']
      return colors[Math.floor(Math.random() * colors.length)]
    },
    selectTemplate(template) {
      uni.setStorageSync('currentModule', template)
      
      // 保存到我的模块
      let modules = uni.getStorageSync('modules') || []
      const exists = modules.find(m => m.name === template.name)
      if (!exists) {
        modules.push(template)
        uni.setStorageSync('modules', modules)
        this.myModules = modules
      }
      
      uni.showToast({
        title: '已选择: ' + template.name,
        icon: 'success'
      })
      
      setTimeout(() => {
        uni.switchTab({
          url: '/pages/home/index'
        })
      }, 1000)
    },
    useModule(module) {
      uni.setStorageSync('currentModule', module)
      uni.showToast({
        title: '已使用: ' + module.name,
        icon: 'success'
      })
      setTimeout(() => {
        uni.switchTab({
          url: '/pages/home/index'
        })
      }, 1000)
    },
    deleteModule(index) {
      uni.showModal({
        title: '确认删除',
        content: '确定要删除这个模块吗？',
        success: (res) => {
          if (res.confirm) {
            let modules = uni.getStorageSync('modules') || []
            modules.splice(index, 1)
            uni.setStorageSync('modules', modules)
            this.myModules = modules
            uni.showToast({
              title: '已删除',
              icon: 'success'
            })
          }
        }
      })
    },
    showImportModal() {
      uni.showModal({
        title: '导入方式',
        content: '请选择导入方式',
        cancelText: '从API导入',
        confirmText: '创建自定义',
        success: (res) => {
          if (res.confirm) {
            this.createCustom()
          } else if (res.cancel) {
            this.loadFromApi()
          }
        }
      })
    },
    createCustom() {
      uni.showToast({
        title: '请在设置中添加自定义模块',
        icon: 'none'
      })
    }
  }
}
</script>

<style scoped>
.page {
  min-height: 100vh;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
  padding-bottom: 150rpx;
}

.header {
  padding: 40rpx 30rpx;
  text-align: center;
}

.title {
  font-size: 40rpx;
  font-weight: bold;
  color: #ffffff;
}

.tab-bar {
  display: flex;
  padding: 0 30rpx;
  gap: 20rpx;
  margin-bottom: 30rpx;
}

.tab-item {
  flex: 1;
  padding: 25rpx;
  text-align: center;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 20rpx;
  border: 2px solid transparent;
}

.tab-item.active {
  background: linear-gradient(135deg, rgba(255, 215, 0, 0.2), rgba(255, 140, 0, 0.2));
  border-color: rgba(255, 215, 0, 0.5);
}

.tab-item text {
  font-size: 28rpx;
  color: #a0a0a0;
}

.tab-item.active text {
  color: #ffd700;
  font-weight: bold;
}

.tab-content {
  padding: 0 30rpx;
}

.market-list,
.my-list {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
}

.template-item,
.module-item {
  display: flex;
  align-items: center;
  padding: 25rpx;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 20rpx;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.template-icon,
.module-icon {
  width: 100rpx;
  height: 100rpx;
  border-radius: 20rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 25rpx;
}

.template-icon text,
.module-icon text {
  font-size: 36rpx;
  color: #ffffff;
  font-weight: bold;
}

.template-info,
.module-info {
  flex: 1;
}

.template-name,
.module-name {
  display: block;
  font-size: 30rpx;
  color: #ffffff;
  font-weight: bold;
  margin-bottom: 8rpx;
}

.template-count,
.module-count {
  display: block;
  font-size: 24rpx;
  color: #a0a0a0;
}

.template-action text,
.module-actions {
  color: #ffd700;
  font-size: 26rpx;
}

.module-actions {
  display: flex;
  gap: 15rpx;
}

.action-btn {
  padding: 10rpx 25rpx;
  background: rgba(255, 215, 0, 0.2);
  border-radius: 20rpx;
}

.action-btn text {
  font-size: 24rpx;
  color: #ffd700;
}

.action-btn.delete {
  background: rgba(255, 100, 100, 0.2);
}

.action-btn.delete text {
  color: #ff6666;
}

.empty-state {
  text-align: center;
  padding: 100rpx 40rpx;
}

.empty-icon {
  font-size: 100rpx;
  display: block;
  margin-bottom: 30rpx;
}

.empty-text {
  display: block;
  font-size: 32rpx;
  color: #ffffff;
  margin-bottom: 15rpx;
}

.empty-hint {
  display: block;
  font-size: 26rpx;
  color: #a0a0a0;
}

.load-more {
  padding: 40rpx;
}

.btn-load {
  width: 100%;
  height: 90rpx;
  background: linear-gradient(135deg, #667eea, #764ba2);
  border: none;
  border-radius: 45rpx;
  font-size: 30rpx;
  color: #ffffff;
}

.fab {
  position: fixed;
  right: 40rpx;
  bottom: 150rpx;
  width: 120rpx;
  height: 120rpx;
  background: linear-gradient(135deg, #ffd700, #ff8c00);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 10rpx 30rpx rgba(255, 215, 0, 0.4);
}

.fab-icon {
  font-size: 60rpx;
  color: #1a1a2e;
  font-weight: bold;
}

.safe-area-bottom {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}
</style>
