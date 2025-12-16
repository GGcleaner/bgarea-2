<template>
  <div class="min-h-screen bg-gray-100">
    <Header />
    
    <div class="container mx-auto px-4 py-8">
      <div class="bg-white rounded-lg shadow-md p-6">
        <h1 class="text-2xl font-bold mb-6">老师空间</h1>
        
        <!-- 教师信息头部 -->
        <div class="flex items-center mb-8">
          <div class="relative">
            <div class="w-24 h-24 rounded-full overflow-hidden border-4 border-white shadow-sm">
              <img :src="teacher.avatar" :alt="teacher.name" class="w-full h-full object-cover">
            </div>
          </div>
          <div class="ml-6">
            <h2 class="text-2xl font-bold text-dark mb-2">{{ teacher.name }}</h2>
            <div class="flex flex-wrap items-center gap-x-6 gap-y-2 text-sm text-secondary">
              <div class="flex items-center gap-1">
                <i class="fa fa-graduation-cap text-primary"></i>
                <span>{{ teacher.department }}</span>
              </div>
              <div class="flex items-center gap-1">
                <i class="fa fa-map-marker text-primary"></i>
                <span>{{ teacher.location || '未设置位置' }}</span>
              </div>
              <div class="flex items-center gap-1">
                <i class="fa fa-star text-yellow-500"></i>
                <span>粉丝数: <span>{{ teacher.followers || 0 }}</span></span>
              </div>
              <div class="flex items-center gap-1">
                <i class="fa fa-book text-primary"></i>
                <span>发布课程: <span>{{ totalCourses }}</span></span>
              </div>
            </div>
          </div>
        </div>
        
        <div class="text-gray-700 mb-8 p-4 bg-gray-50 rounded-lg">
          <h3 class="font-semibold text-dark mb-2">教师简介</h3>
          <p>{{ teacher.description || '这位老师还没有填写个人简介' }}</p>
        </div>
        
        <!-- 课程区域 -->
        <div class="mt-8">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-lg font-semibold text-dark">教师的课程 ({{ totalCourses }})</h3>
            <!-- 只有一个查看更多/收起按钮 -->
            <button 
              v-if="teacherCourses.length > initialCourseCount"
              class="text-link text-sm hover:underline flex items-center gap-1 px-3 py-1.5 rounded-md hover:bg-blue-50 transition-colors"
              @click="toggleExpandCourses"
            >
              {{ showAllCourses ? '收起课程' : `查看更多 (${totalCourses - initialCourseCount}个课程)` }}
              <i class="fa" :class="showAllCourses ? 'fa-angle-up' : 'fa-angle-down'"></i>
            </button>
          </div>
          
          <!-- 课程网格 -->
          <div 
            class="grid gap-4 transition-all duration-300"
            :class="gridColsClass"
          >
            <!-- 课程卡片 -->
            <div 
              v-for="course in displayedCourses" 
              :key="course.id"
              class="bg-white rounded-lg overflow-hidden cursor-pointer hover:shadow-lg transition-all duration-300 group video-card"
              @click="goToCourse(course)"
            >
              <!-- 视频封面 -->
              <div class="relative" style="aspect-ratio: 16/9;">
                <img 
                  :src="course.image" 
                  :alt="course.title" 
                  class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
                >
                <!-- 播放按钮 -->
                <div class="absolute inset-0 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity duration-300 bg-black/20">
                  <div class="w-12 h-12 rounded-full bg-white/20 backdrop-blur-sm flex items-center justify-center">
                    <i class="fa fa-play text-white text-lg"></i>
                  </div>
                </div>
              </div>
              <!-- 课程信息 -->
              <div class="p-3">
                <h4 class="text-sm font-medium line-clamp-2 group-hover:text-primary transition-colors mb-1">
                  {{ course.title }}
                </h4>
                <div class="text-xs text-gray-500">{{ course.subject }}</div>
                <div class="text-xs text-gray-400 mt-1">{{ course.createdAt }}</div>
              </div>
            </div>
          </div>
          
          <!-- 空状态 -->
          <div v-if="teacherCourses.length === 0" class="text-center py-12">
            <i class="fa fa-book-open text-4xl text-gray-300 mb-4"></i>
            <p class="text-gray-500">该老师还没有发布任何课程</p>
          </div>
        </div>
        
        <!-- 返回按钮 -->
        <div class="mt-8 pt-6 border-t border-gray-200">
          <button @click="goBack" class="px-6 py-2.5 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors duration-200">
            <i class="fa fa-arrow-left mr-2"></i>返回上一页
          </button>
        </div>
      </div>
    </div>
    
    <Footer />
  </div>
</template>

<script>
import Header from '@/components/Header.vue'
import Footer from '@/components/Footer.vue'
import { useRoute, useRouter } from 'vue-router'
import { isTestAccount, getTestAccountData } from '@/data/mockData.js'

export default {
  name: 'TeacherSpace',
  components: {
    Header,
    Footer
  },
  setup() {
    const route = useRoute()
    const router = useRouter()
    
    const teacher = JSON.parse(localStorage.getItem('currentTeacherInfo') || '{}')
    
    const goBack = () => {
      router.go(-1)
    }
    
    return {
      teacher,
      goBack
    }
  },
  data() {
    return {
      showAllCourses: false,
      initialCourseCount: 8,
      teacherCourses: []
    }
  },
  computed: {
    displayedCourses() {
      if (this.showAllCourses) {
        // 点击"查看更多"后显示全部课程
        return this.teacherCourses
      }
      // 默认只显示前8个课程
      return this.teacherCourses.slice(0, this.initialCourseCount)
    },
    
    totalCourses() {
      return this.teacherCourses.length
    },
    
    gridColsClass() {
      // 基础网格布局
      const baseClass = 'grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4'
      if (this.showAllCourses) {
        // 展开时显示完整网格
        return baseClass
      }
      // 收起时限制高度并隐藏溢出
      return `${baseClass} max-h-[500px] overflow-hidden`
    }
  },
  mounted() {
    if (isTestAccount(this.teacher.email)) {
      const data = getTestAccountData()
      this.teacherCourses = data.courses.map(course => ({
        id: course.id,
        title: course.name,
        image: course.avatar,
        subject: course.tag,
        createdAt: course.createdAt
      }))
    } else {
      this.teacherCourses = []
    }

    const teacherId = this.$route.query.teacherId
    if (teacherId) {
      this.loadTeacherCourses(teacherId)
    }
  },
  methods: {
    toggleExpandCourses() {
      this.showAllCourses = !this.showAllCourses
    },
    
    goToCourse(course) {
      this.$router.push({
        name: 'VideoPlayer',
        params: { courseId: course.id }
      })
    },
    
    loadTeacherCourses(teacherId) {
      // 这里可以根据teacherId从API加载对应老师的课程
      console.log('加载老师课程:', teacherId)
    }
  }
}
</script>

<style scoped>
/* 视频卡片样式 */
.video-card {
  transition: all 0.3s ease;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

.video-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
}

.line-clamp-2 {
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
}

/* 查看更多按钮样式 */
.text-link {
  @apply text-blue-600 hover:text-blue-800 transition-colors duration-200;
}
</style>