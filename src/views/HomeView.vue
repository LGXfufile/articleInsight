<script setup lang="ts">
import { ref, reactive } from 'vue'
import { AnalysisService, exportToMarkdown as generateMarkdown } from '@/services/api'
import type { AnalysisData } from '@/services/api'

const searchKeyword = ref('')
const isLoading = ref(false)
const error = ref('')
const analysisResult = reactive<AnalysisData>({
  painPoints: [],
  competitors: [],
  opportunities: [],
  difficulty: '',
  marketSize: '',
  suggestions: []
})

const suggestedKeywords = [
  '人工智能', '新能源汽车', '宠物经济', '居家办公', 
  '元宇宙', '数字货币', '短视频', '直播带货'
]

const handleSearch = async () => {
  if (!searchKeyword.value.trim()) return
  
  isLoading.value = true
  error.value = ''
  
  try {
    const result = await AnalysisService.performComprehensiveAnalysis(searchKeyword.value)
    
    // 更新响应式数据
    Object.assign(analysisResult, result)
  } catch (err) {
    error.value = err instanceof Error ? err.message : '分析失败，请稍后重试'
    console.error('Analysis error:', err)
  } finally {
    isLoading.value = false
  }
}

const exportToMarkdown = () => {
  const markdown = generateMarkdown(searchKeyword.value, analysisResult)
  
  navigator.clipboard.writeText(markdown).then(() => {
    alert('Markdown内容已复制到剪贴板！可以直接粘贴到各大自媒体平台发布。')
  }).catch(() => {
    // 降级方案
    const textArea = document.createElement('textarea')
    textArea.value = markdown
    document.body.appendChild(textArea)
    textArea.select()
    document.execCommand('copy')
    document.body.removeChild(textArea)
    alert('内容已复制到剪贴板！')
  })
}
</script>

<template>
  <div class="home-container">
    <div class="main-content">
      <!-- 标题区域 -->
      <div class="header-section">
        <h1 class="main-title">OPPORTUNITY</h1>
        <div class="divider"></div>
        <p class="subtitle">🔍 输入任何关键词，发现商业机会</p>
      </div>

      <!-- 搜索区域 -->
      <div class="search-section">
        <div class="search-input-container">
          <input 
            v-model="searchKeyword"
            type="text" 
            placeholder="人工智能"
            class="search-input"
            @keyup.enter="handleSearch"
          >
          <button 
            @click="handleSearch" 
            :disabled="isLoading"
            class="search-button"
          >
            <span v-if="!isLoading">🔍</span>
            <span v-else class="loading-spinner">🔄</span>
          </button>
        </div>
        
        <div class="suggested-keywords">
          💡 试试：
          <span 
            v-for="keyword in suggestedKeywords" 
            :key="keyword"
            @click="searchKeyword = keyword"
            class="keyword-tag"
          >
            {{ keyword }}
          </span>
        </div>
      </div>

      <!-- 分析结果 -->
      <div v-if="error" class="error-section">
        <div class="error-card">
          <h3>⚠️ 分析遇到问题</h3>
          <p>{{ error }}</p>
          <button @click="handleSearch" class="retry-button">重新分析</button>
        </div>
      </div>

      <div v-else-if="isLoading || analysisResult.painPoints.length > 0" class="results-section">
        <div v-if="isLoading" class="loading-state">
          <div class="radar-animation">🎯</div>
          <p>正在全网挖掘商业机会...</p>
        </div>
        
        <div v-else class="analysis-cards">
          <!-- 痛点热力图 -->
          <div class="analysis-card">
            <div class="card-header">
              <h3>🔥 痛点热力图</h3>
            </div>
            <div class="card-content">
              <ul>
                <li v-for="point in analysisResult.painPoints" :key="point">
                  {{ point }}
                </li>
              </ul>
            </div>
          </div>

          <!-- 赚钱机会 -->
          <div class="analysis-card">
            <div class="card-header">
              <h3>💰 赚钱机会</h3>
            </div>
            <div class="card-content">
              <div class="market-info">
                <p><strong>市场规模:</strong> {{ analysisResult.marketSize }}</p>
                <p><strong>难度评估:</strong> {{ analysisResult.difficulty }}</p>
              </div>
              <ul>
                <li v-for="opportunity in analysisResult.opportunities" :key="opportunity">
                  {{ opportunity }}
                </li>
              </ul>
            </div>
          </div>

          <!-- 竞品雷达 -->
          <div class="analysis-card">
            <div class="card-header">
              <h3>⚡ 竞品雷达</h3>
            </div>
            <div class="card-content">
              <ul>
                <li v-for="competitor in analysisResult.competitors" :key="competitor">
                  {{ competitor }}
                </li>
              </ul>
            </div>
          </div>
        </div>

        <!-- 导出按钮 -->
        <div v-if="!isLoading && analysisResult.painPoints.length > 0" class="export-section">
          <h3>📝 自媒体文章就绪</h3>
          <div class="platform-buttons">
            <button class="platform-btn">小红书</button>
            <button class="platform-btn">公众号</button>
            <button class="platform-btn">知乎</button>
            <button class="platform-btn">今日头条</button>
          </div>
          <button @click="exportToMarkdown" class="export-button">
            📋 复制Markdown
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.home-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.main-content {
  max-width: 1200px;
  width: 100%;
}

/* 标题区域 */
.header-section {
  text-align: center;
  margin-bottom: 60px;
}

.main-title {
  font-size: 3rem;
  font-weight: 300;
  color: white;
  letter-spacing: 8px;
  margin-bottom: 20px;
}

.divider {
  width: 100px;
  height: 2px;
  background: white;
  margin: 0 auto 30px;
}

.subtitle {
  font-size: 1.2rem;
  color: rgba(255, 255, 255, 0.9);
  font-weight: 300;
}

/* 搜索区域 */
.search-section {
  text-align: center;
  margin-bottom: 60px;
}

.search-input-container {
  display: flex;
  max-width: 600px;
  margin: 0 auto 30px;
  background: white;
  border-radius: 50px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  overflow: hidden;
}

.search-input {
  flex: 1;
  padding: 20px 30px;
  border: none;
  font-size: 1.1rem;
  outline: none;
  background: transparent;
}

.search-input::placeholder {
  color: #ccc;
}

.search-button {
  padding: 20px 30px;
  background: #667eea;
  border: none;
  color: white;
  font-size: 1.2rem;
  cursor: pointer;
  transition: background 0.3s;
}

.search-button:hover {
  background: #5a67d8;
}

.search-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.loading-spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.suggested-keywords {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
}

.keyword-tag {
  display: inline-block;
  margin: 0 10px;
  padding: 5px 15px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s;
}

.keyword-tag:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

/* 结果区域 */
.results-section {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 40px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.1);
}

.loading-state {
  text-align: center;
  padding: 60px 0;
}

.radar-animation {
  font-size: 4rem;
  animation: pulse 2s infinite;
  margin-bottom: 20px;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.1); opacity: 0.7; }
}

.analysis-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 30px;
  margin-bottom: 40px;
}

.analysis-card {
  background: white;
  border-radius: 15px;
  padding: 25px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.08);
  border-left: 4px solid #667eea;
}

.card-header h3 {
  color: #333;
  margin-bottom: 20px;
  font-size: 1.2rem;
}

.card-content ul {
  list-style: none;
  padding: 0;
}

.card-content li {
  padding: 10px 0;
  border-bottom: 1px solid #f0f0f0;
  color: #666;
  line-height: 1.5;
}

.card-content li:last-child {
  border-bottom: none;
}

.market-info {
  background: #f8f9ff;
  padding: 15px;
  border-radius: 10px;
  margin-bottom: 20px;
}

.market-info p {
  margin: 5px 0;
  color: #555;
}

/* 导出区域 */
.export-section {
  text-align: center;
  padding-top: 30px;
  border-top: 1px solid #e0e0e0;
}

.export-section h3 {
  color: #333;
  margin-bottom: 20px;
}

.platform-buttons {
  margin-bottom: 20px;
}

.platform-btn {
  margin: 0 10px;
  padding: 8px 20px;
  background: #f0f0f0;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 0.9rem;
}

.platform-btn:hover {
  background: #667eea;
  color: white;
}

.export-button {
  padding: 15px 40px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 25px;
  font-size: 1.1rem;
  cursor: pointer;
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.3);
  transition: all 0.3s;
}

.export-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 7px 20px rgba(102, 126, 234, 0.4);
}

/* 移动端适配 */
@media (max-width: 768px) {
  .home-container {
    padding: 10px;
  }

  .main-title {
    font-size: 2rem;
    letter-spacing: 4px;
  }
  
  .analysis-cards {
    grid-template-columns: 1fr;
    gap: 20px;
  }
  
  .search-input-container {
    flex-direction: column;
    border-radius: 15px;
    max-width: 100%;
  }
  
  .search-input {
    padding: 15px 20px;
    font-size: 1rem;
  }
  
  .search-button {
    padding: 15px 20px;
    border-radius: 0 0 15px 15px;
  }
  
  .suggested-keywords {
    margin-top: 20px;
  }
  
  .keyword-tag {
    display: inline-block;
    margin: 5px 5px;
    padding: 8px 12px;
    font-size: 0.9rem;
  }

  .results-section {
    padding: 20px;
    margin: 20px 0;
  }

  .analysis-card {
    padding: 20px;
  }

  .platform-buttons {
    flex-wrap: wrap;
    gap: 10px;
  }

  .platform-btn {
    flex: 1;
    min-width: 120px;
  }

  .export-button {
    width: 100%;
    margin-top: 10px;
  }
}

@media (max-width: 480px) {
  .main-title {
    font-size: 1.8rem;
    letter-spacing: 2px;
  }

  .subtitle {
    font-size: 1rem;
  }

  .search-input {
    font-size: 0.9rem;
  }

  .keyword-tag {
    font-size: 0.8rem;
    padding: 6px 10px;
  }
}

/* 错误状态样式 */
.error-section {
  display: flex;
  justify-content: center;
  margin: 40px 0;
}

.error-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 15px;
  padding: 30px;
  text-align: center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  border-left: 4px solid #f56565;
  max-width: 500px;
}

.error-card h3 {
  color: #f56565;
  margin-bottom: 15px;
  font-size: 1.2rem;
}

.error-card p {
  color: #666;
  margin-bottom: 20px;
  line-height: 1.5;
}

.retry-button {
  padding: 10px 30px;
  background: #f56565;
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.3s;
}

.retry-button:hover {
  background: #e53e3e;
  transform: translateY(-2px);
}
</style>
