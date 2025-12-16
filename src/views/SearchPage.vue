<template>
  <div class="min-h-screen flex flex-col">
    <Header />
    
    <!-- 主内容容器 -->
    <div class="flex-1 bg-gray-50">
      <div class="container mx-auto p-6 md:p-8">
        <!-- 标题区域 -->
        <div class="mb-8">
          <h1 class="text-2xl font-semibold text-dark mb-2">搜索课程</h1>
          <p v-if="searchQuery" class="text-gray-600">搜索关键词：<span class="text-primary font-medium">{{ searchQuery }}</span></p>
        </div>

        <!-- 筛选区域 -->
        <div class="mb-8">
          <div class="flex flex-wrap items-center justify-between gap-4 mb-4">
            <h2 class="text-lg font-semibold text-dark">搜索结果：<span id="resultCount">{{ filteredCourses.length }}</span> 个相关课程</h2>
            <div class="flex items-center gap-4">
              <select v-model="sortBy" class="px-4 py-2 border border-gray-300 rounded-md focus:border-primary focus:outline-none text-sm">
                <option value="default">综合排序</option>
                <option value="hot">按热度排序</option>
                <option value="time">按时间排序</option>
                <option value="rating">按评分排序</option>
              </select>
              <select v-model="selectedCategory" class="px-4 py-2 border border-gray-300 rounded-md focus:border-primary focus:outline-none text-sm">
                <option value="all">全部分类</option>
                <option v-for="category in categories" :key="category.value" :value="category.value">
                  {{ category.label }}
                </option>
              </select>
            </div>
          </div>
        </div>

        <!-- 搜索结果区域 -->
        <div class="grid grid-cols-1 gap-6">
          <!-- 课程列表 -->
          <div>
            <!-- 空状态 -->
            <div v-if="filteredCourses.length === 0" class="text-center py-12">
              <i class="fa fa-search text-4xl text-gray-400 mb-4"></i>
              <p class="text-gray-500">没有找到相关课程</p>
              <p class="text-sm text-gray-400 mt-2">试试其他关键词吧</p>
            </div>

            <!-- 搜索结果列表 -->
            <div v-else class="space-y-6">
              <!-- 搜索结果卡片 -->
              <div
                v-for="course in paginatedCourses"
                :key="course.id"
                class="card p-4 hover:shadow-md transition-shadow duration-200 cursor-pointer"
                @click="goToCourseDetail(course.id)"
              >
                <div class="flex flex-col md:flex-row gap-4">
                  <div class="relative w-full md:w-48 h-32 rounded-lg overflow-hidden flex-shrink-0">
                    <img :src="course.image" :alt="course.title" class="w-full h-full object-cover">
                    <span :class="course.badgeClass">{{ course.badgeText }}</span>
                    <span class="video-time">{{ formatDuration(course.duration) }}</span>
                  </div>
                  <div class="flex-1">
                    <h3 class="text-lg font-semibold text-dark mb-2 hover:text-primary transition-colors">{{ course.title }}</h3>
                    <p class="text-sm text-gray-600 mb-3 line-clamp-2">
                      {{ course.description }}
                    </p>
                    <div class="flex flex-wrap items-center gap-4 text-sm text-gray-500">
                      <span class="flex items-center gap-1"><i class="fa fa-user text-gray-400"></i> {{ course.teacher }}</span>
                      <span class="flex items-center gap-1"><i class="fa fa-eye text-gray-400"></i> {{ course.views }}</span>
                      <span class="flex items-center gap-1"><i class="fa fa-thumbs-up text-gray-400"></i> {{ course.rating }}</span>
                      <span class="text-xs px-2 py-1 bg-gray-100 rounded">{{ course.category }}</span>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- 分页 -->
            <div v-if="filteredCourses.length > 0" class="flex justify-center items-center gap-2 mt-8">
              <button 
                @click="prevPage" 
                :disabled="currentPage === 1"
                class="w-10 h-10 flex items-center justify-center rounded-md border border-gray-200 hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed"
              >
                <i class="fa fa-angle-left text-gray-500"></i>
              </button>
              
              <button 
                v-for="page in visiblePages" 
                :key="page"
                @click="goToPage(page)"
                class="w-10 h-10 flex items-center justify-center rounded-md border"
                :class="currentPage === page ? 'bg-primary text-white border-primary' : 'border-gray-200 hover:bg-gray-50'"
              >
                {{ page }}
              </button>
              
              <button 
                @click="nextPage" 
                :disabled="currentPage === totalPages"
                class="w-10 h-10 flex items-center justify-center rounded-md border border-gray-200 hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed"
              >
                <i class="fa fa-angle-right text-gray-500"></i>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <Footer />
  </div>
</template>

<script>
import Header from '@/components/Header.vue'
import Footer from '@/components/Footer.vue'
import { allCourses } from '@/data/coursesData.js'

export default {
  name: 'SearchPage',
  components: {
    Header,
    Footer
  },
  data() {
    return {
      searchQuery: '',
      allCourses: allCourses,
      categories: [
        { value: 'programming', label: '编程开发' },
        { value: 'ai', label: '人工智能' },
        { value: 'data-science', label: '数据科学' },
        { value: 'business', label: '商业管理' },
        { value: 'medical', label: '医学健康' }
      ],
      selectedCategory: 'all',
      sortBy: 'default',
      difficultyLevels: [
        { value: 'beginner', label: '入门' },
        { value: 'intermediate', label: '中级' },
        { value: 'advanced', label: '高级' }
      ],
      selectedDifficulty: [],
      durationFilters: [
        { value: 'short', label: '0-5小时' },
        { value: 'medium', label: '5-15小时' },
        { value: 'long', label: '15小时以上' }
      ],
      selectedDuration: [],
      isFree: true,
      isPaid: false,
      currentPage: 1,
      pageSize: 10
    }
  },
  computed: {
    filteredCourses() {
      let filtered = [...this.allCourses]

      // 搜索关键词过滤（模糊搜索）
      if (this.searchQuery.trim()) {
        const query = this.searchQuery.toLowerCase()
        filtered = filtered.filter(course =>
          course.title.toLowerCase().includes(query) ||
          course.description.toLowerCase().includes(query) ||
          course.teacher.toLowerCase().includes(query) ||
          course.category.toLowerCase().includes(query)
        )
      }
      
      // 分类过滤
      if (this.selectedCategory !== 'all') {
        const categoryMap = {
          'programming': '编程开发',
          'ai': '人工智能',
          'data-science': '数据科学',
          'business': '商业管理',
          'medical': '医学健康'
        }
        const targetCategory = categoryMap[this.selectedCategory] || this.selectedCategory
        filtered = filtered.filter(course => course.category === targetCategory)
      }
      
      // 难度过滤
      if (this.selectedDifficulty.length > 0) {
        filtered = filtered.filter(course => this.selectedDifficulty.includes(course.difficulty))
      }
      
      // 时长过滤
      if (this.selectedDuration.length > 0) {
        filtered = filtered.filter(course => {
          const hours = course.duration / 3600
          if (this.selectedDuration.includes('short')) return hours <= 5
          if (this.selectedDuration.includes('medium')) return hours > 5 && hours <= 15
          if (this.selectedDuration.includes('long')) return hours > 15
          return true
        })
      }
      
      // 价格过滤
      const priceFilters = []
      if (this.isFree) priceFilters.push('free')
      if (this.isPaid) priceFilters.push('paid')
      if (priceFilters.length > 0) {
        filtered = filtered.filter(course => priceFilters.includes(course.price))
      }
      
      // 排序
      filtered.sort((a, b) => {
        switch (this.sortBy) {
          case 'hot':
            return parseInt(b.views.replace(/[^\d]/g, '')) - parseInt(a.views.replace(/[^\d]/g, ''))
          case 'time':
            return b.id - a.id // 假设ID越大越新
          case 'rating':
            return parseFloat(b.rating) - parseFloat(a.rating)
          default:
            return 0
        }
      })
      
      return filtered
    },
    totalPages() {
      return Math.ceil(this.filteredCourses.length / this.pageSize)
    },
    paginatedCourses() {
      const start = (this.currentPage - 1) * this.pageSize
      const end = start + this.pageSize
      return this.filteredCourses.slice(start, end)
    },
    visiblePages() {
      const pages = []
      const maxVisible = 5
      
      if (this.totalPages <= maxVisible) {
        for (let i = 1; i <= this.totalPages; i++) {
          pages.push(i)
        }
      } else {
        let start = Math.max(1, this.currentPage - 2)
        let end = Math.min(this.totalPages, start + maxVisible - 1)
        
        if (end - start + 1 < maxVisible) {
          start = Math.max(1, end - maxVisible + 1)
        }
        
        for (let i = start; i <= end; i++) {
          pages.push(i)
        }
      }
      
      return pages
    }
  },
  mounted() {
    if (this.$route.query.q) {
      this.searchQuery = this.$route.query.q
    }
  },
  watch: {
    '$route.query.q'(newQuery) {
      if (newQuery) {
        this.searchQuery = newQuery
        this.currentPage = 1
      }
    }
  },
  methods: {
    setSearchQuery(query) {
      this.searchQuery = query
      this.currentPage = 1
    },

    performSearch() {
      this.currentPage = 1
    },
    
    formatDuration(seconds) {
      const hours = Math.floor(seconds / 3600)
      const minutes = Math.floor((seconds % 3600) / 60)
      const secs = seconds % 60
      return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
    },
    
    applyFilters() {
      this.currentPage = 1 // 重置到第一页
    },
    
    goToCourseDetail(courseId) {
      this.$router.push({
        name: 'VideoPlayer',
        params: { id: courseId },
        query: { from: 'search' }
      })
    },
    
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--
      }
    },
    
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++
      }
    },
    
    goToPage(page) {
      this.currentPage = page
    }
  }
}
</script>

<style scoped>
/* 引入Tailwind中的自定义工具类 */
.hide-scrollbar::-webkit-scrollbar {
  display: none;
}
.hide-scrollbar {
  -ms-overflow-style: none;
  scrollbar-width: none;
}

/* 卡片hover效果优化 */
.card {
  transition: all 0.3s ease;
}
.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.05);
}

/* 限制行数 */
.line-clamp-1 {
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 1;
}
.line-clamp-2 {
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
}
.line-clamp-3 {
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
}

/* 视频相关工具类 */
.badge-hot {
  @apply absolute top-2 left-2 bg-red-500 text-white text-xs px-2 py-0.5 rounded z-10;
}
.badge-blue {
  @apply absolute top-2 left-2 bg-primary text-white text-xs px-2 py-0.5 rounded z-10;
}
.badge-green {
  @apply absolute top-2 left-2 bg-success text-white text-xs px-2 py-0.5 rounded z-10;
}
.badge-orange {
  @apply absolute top-2 left-2 bg-orange-500 text-white text-xs px-2 py-0.5 rounded z-10;
}
.video-time {
  @apply absolute bottom-2 right-2 bg-black/70 text-white text-xs px-1.5 py-0.5 rounded z-10;
}
.video-card-hover {
  @apply hover:shadow-md transition-shadow duration-200 cursor-pointer;
}

/* 确保所有课程卡片都有手型光标 */
.cursor-pointer {
  cursor: pointer;
}
</style>