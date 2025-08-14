<template>
  <header class="header">
    <div class="header-container">
      <!-- 网站标题 -->
      <div class="site-title">
        <h1 class="main-title">青岚</h1>
        <p class="subtitle">寻青云涧，栖山水间</p>
      </div>
      
      <!-- 导航菜单 -->
      <nav class="navigation">
        <router-link to="/" class="nav-link">首页</router-link>
        <router-link to="/blog" class="nav-link">个人笔记</router-link>
        <router-link to="/recommendations" class="nav-link">推荐</router-link>
        <router-link to="/news" class="nav-link">资讯</router-link>
        <router-link to="/map" class="nav-link">世界地图</router-link>
      </nav>
      
      <!-- 搜索框 -->
      <div class="search-box">
        <el-autocomplete
          v-model="searchQuery"
          :fetch-suggestions="querySearchAsync"
          placeholder="站内搜索..."
          @select="handleSelect"
          @keyup.enter="handleSearch"
          class="search-input"
          clearable
        >
          <template #suffix>
            <el-icon @click="handleSearch" class="search-icon">
              <Search />
            </el-icon>
          </template>
          <template #default="{ item }">
            <div class="search-suggestion">
              <el-icon class="suggestion-icon">
                <component :is="item.icon" />
              </el-icon>
              <span class="suggestion-text">{{ item.value }}</span>
              <span class="suggestion-type">{{ item.type }}</span>
            </div>
          </template>
        </el-autocomplete>
      </div>
      
      <!-- 移动端菜单按钮 -->
      <div class="mobile-menu-btn" @click="toggleMobileMenu">
        <el-icon><Menu /></el-icon>
      </div>
    </div>
    
    <!-- 移动端菜单 -->
    <div class="mobile-menu" :class="{ active: showMobileMenu }">
      <router-link to="/" class="mobile-nav-link" @click="closeMobileMenu">首页</router-link>
      <router-link to="/blog" class="mobile-nav-link" @click="closeMobileMenu">个人笔记</router-link>
      <router-link to="/recommendations" class="mobile-nav-link" @click="closeMobileMenu">推荐</router-link>
      <router-link to="/news" class="mobile-nav-link" @click="closeMobileMenu">资讯</router-link>
      <router-link to="/map" class="mobile-nav-link" @click="closeMobileMenu">世界地图</router-link>
    </div>
  </header>
</template>

<script>
import { Search, Menu } from '@element-plus/icons-vue'

export default {
  name: 'HeaderComponent',
  components: {
    Search,
    Menu
  },
  data() {
    return {
      searchQuery: '',
      showMobileMenu: false,
      searchSuggestions: [
        { value: '音乐推荐', type: '推荐', icon: 'Headphone' },
        { value: '软件工具', type: '推荐', icon: 'Monitor' },
        { value: '网站收藏', type: '推荐', icon: 'Link' },
        { value: '个人笔记', type: '博客', icon: 'Document' },
        { value: '今日新鲜事', type: '资讯', icon: 'Notification' },
        { value: '网安事件', type: '资讯', icon: 'Lock' },
        { value: '历史上的今天', type: '资讯', icon: 'Calendar' },
        { value: 'AI工具', type: '推荐', icon: 'MagicStick' },
        { value: '电影推荐', type: '推荐', icon: 'VideoCamera' }
      ],
      searchHistory: JSON.parse(localStorage.getItem('searchHistory') || '[]')
    }
  },
  methods: {
    handleSearch() {
      if (this.searchQuery.trim()) {
        this.addToSearchHistory(this.searchQuery)
        this.performSearch(this.searchQuery)
      }
    },
    handleSelect(item) {
      this.searchQuery = item.value
      this.handleSearch()
    },
    querySearchAsync(queryString, callback) {
      const suggestions = this.searchSuggestions.filter(item =>
        item.value.toLowerCase().includes(queryString.toLowerCase())
      )

      // 添加搜索历史到建议中
      const historyItems = this.searchHistory
        .filter(item => item.toLowerCase().includes(queryString.toLowerCase()))
        .slice(0, 3)
        .map(item => ({ value: item, type: '历史', icon: 'Clock' }))

      const allSuggestions = [...historyItems, ...suggestions].slice(0, 8)
      callback(allSuggestions)
    },
    addToSearchHistory(query) {
      const history = this.searchHistory.filter(item => item !== query)
      history.unshift(query)
      this.searchHistory = history.slice(0, 10) // 保留最近10条
      localStorage.setItem('searchHistory', JSON.stringify(this.searchHistory))
    },
    performSearch(query) {
      // 根据搜索内容跳转到相应页面
      const lowerQuery = query.toLowerCase()

      if (lowerQuery.includes('音乐') || lowerQuery.includes('歌曲')) {
        this.$router.push({ path: '/recommendations', query: { category: 'music' } })
      } else if (lowerQuery.includes('软件') || lowerQuery.includes('工具')) {
        this.$router.push({ path: '/recommendations', query: { category: 'software' } })
      } else if (lowerQuery.includes('网站') || lowerQuery.includes('链接')) {
        this.$router.push({ path: '/recommendations', query: { category: 'websites' } })
      } else if (lowerQuery.includes('笔记') || lowerQuery.includes('博客') || lowerQuery.includes('文章')) {
        this.$router.push('/blog')
      } else if (lowerQuery.includes('新闻') || lowerQuery.includes('资讯') || lowerQuery.includes('事件')) {
        this.$router.push('/news')
      } else if (lowerQuery.includes('地图')) {
        this.$router.push('/map')
      } else if (lowerQuery.includes('ai') || lowerQuery.includes('人工智能')) {
        this.$router.push({ path: '/recommendations', query: { category: 'ai' } })
      } else if (lowerQuery.includes('电影') || lowerQuery.includes('电视')) {
        this.$router.push({ path: '/recommendations', query: { category: 'movies' } })
      } else {
        // 默认搜索，可以在这里实现全文搜索
        this.$router.push({ path: '/search', query: { q: query } })
      }

      this.$emit('search', query)
    },
    toggleMobileMenu() {
      this.showMobileMenu = !this.showMobileMenu
    },
    closeMobileMenu() {
      this.showMobileMenu = false
    }
  }
}
</script>

<style scoped>
.header {
  background: linear-gradient(135deg, #8B4513 0%, #A0522D 50%, #CD853F 100%);
  color: white;
  box-shadow: 0 3px 15px rgba(139, 69, 19, 0.3);
  position: sticky;
  top: 0;
  z-index: 1000;
}

.header-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0.8rem 1.5rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 1rem;
}

.site-title {
  text-align: left;
  flex-shrink: 0;
}

.main-title {
  font-size: 1.8rem;
  margin: 0;
  font-weight: 600;
  text-shadow: 2px 2px 6px rgba(0, 0, 0, 0.4);
  font-family: 'STKaiti', '楷体', serif;
  letter-spacing: 2px;
}

.subtitle {
  font-size: 0.75rem;
  margin: 0.2rem 0 0 0;
  opacity: 0.85;
  font-style: italic;
  font-family: 'STKaiti', '楷体', serif;
  letter-spacing: 1px;
}

.navigation {
  display: flex;
  gap: 1.2rem;
  flex-wrap: wrap;
}

.nav-link {
  color: white;
  text-decoration: none;
  font-weight: 400;
  transition: all 0.3s ease;
  padding: 0.4rem 0.8rem;
  border-radius: 15px;
  font-size: 0.9rem;
  white-space: nowrap;
}

.nav-link:hover,
.nav-link.router-link-active {
  background: rgba(255, 255, 255, 0.25);
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.search-box {
  width: 240px;
  flex-shrink: 0;
}

.search-input {
  border-radius: 25px;
}

.search-icon {
  cursor: pointer;
  color: #409eff;
}

.search-suggestion {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 0;
}

.suggestion-icon {
  color: #909399;
  font-size: 14px;
}

.suggestion-text {
  flex: 1;
  color: #303133;
}

.suggestion-type {
  font-size: 12px;
  color: #909399;
  background: #f5f7fa;
  padding: 2px 6px;
  border-radius: 10px;
}

.mobile-menu-btn {
  display: none;
  cursor: pointer;
  font-size: 1.5rem;
}

.mobile-menu {
  display: none;
  flex-direction: column;
  background: rgba(0, 0, 0, 0.9);
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  padding: 1rem;
}

.mobile-menu.active {
  display: flex;
}

.mobile-nav-link {
  color: white;
  text-decoration: none;
  padding: 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

@media (max-width: 768px) {
  .header-container {
    padding: 0.6rem 1rem;
    gap: 0.8rem;
  }

  .main-title {
    font-size: 1.5rem;
  }

  .subtitle {
    font-size: 0.7rem;
  }

  .navigation {
    display: none;
  }

  .search-box {
    width: 180px;
  }

  .mobile-menu-btn {
    display: block;
  }
}

@media (max-width: 480px) {
  .header-container {
    flex-direction: column;
    gap: 1rem;
  }
  
  .search-box {
    width: 100%;
  }
}
</style>
