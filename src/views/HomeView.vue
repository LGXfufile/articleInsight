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
  'Artificial Intelligence', 'Electric Vehicles', 'Pet Economy', 'Remote Work',
  'Metaverse', 'Cryptocurrency', 'Short Video', 'Live Commerce'
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
        <div class="logo-container">
          <div class="logo-icon">💡</div>
          <h1 class="main-title">OPPORTUNITY</h1>
          <div class="subtitle-english">Business Intelligence Platform</div>
        </div>
        <div class="divider-elegant"></div>
        <p class="subtitle">🔍 Discover business opportunities with any keyword</p>
        <p class="subtitle-chinese">输入任何关键词，发现商业机会</p>
      </div>

      <!-- 搜索区域 -->
      <div class="search-section">
        <div class="search-container-wrapper">
          <div class="search-input-container">
            <div class="search-icon">🔍</div>
            <input 
              v-model="searchKeyword"
              type="text" 
              placeholder="Try: Artificial Intelligence, Pet Economy..."
              class="search-input"
              @keyup.enter="handleSearch"
            >
            <button 
              @click="handleSearch" 
              :disabled="isLoading"
              class="search-button"
            >
              <span v-if="!isLoading" class="search-btn-text">Analyze</span>
              <span v-else class="loading-spinner">⚡</span>
            </button>
          </div>
        </div>
        
        <div class="suggested-keywords">
          <div class="suggestions-label">💡 Popular Keywords:</div>
          <div class="keyword-tags-container">
            <span 
              v-for="(keyword, index) in suggestedKeywords" 
              :key="keyword"
              @click="searchKeyword = keyword"
              class="keyword-tag"
              :style="{ animationDelay: `${index * 0.1}s` }"
            >
              {{ keyword }}
            </span>
          </div>
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
  position: relative;
  overflow: hidden;
}

.home-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  background-size: 300% 300%;
  animation: gradientShift 15s ease infinite;
}

@keyframes gradientShift {
  0%, 100% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
}

.main-content {
  max-width: 1000px;
  width: 100%;
  position: relative;
  z-index: 1;
}

/* 标题区域 */
.header-section {
  text-align: center;
  margin-bottom: 80px;
}

.logo-container {
  margin-bottom: 40px;
}

.logo-icon {
  font-size: 4rem;
  margin-bottom: 20px;
  animation: float 6s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-15px); }
}

.main-title {
  font-size: 4rem;
  font-weight: 200;
  color: white;
  letter-spacing: 12px;
  margin-bottom: 10px;
  text-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  animation: titleGlow 3s ease-in-out infinite alternate;
}

@keyframes titleGlow {
  0% { text-shadow: 0 4px 8px rgba(0, 0, 0, 0.3); }
  100% { text-shadow: 0 4px 20px rgba(255, 255, 255, 0.5); }
}

.subtitle-english {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.8);
  font-weight: 300;
  letter-spacing: 3px;
  text-transform: uppercase;
  margin-bottom: 30px;
}

.divider-elegant {
  width: 200px;
  height: 1px;
  background: linear-gradient(90deg, transparent 0%, white 50%, transparent 100%);
  margin: 0 auto 30px;
  position: relative;
}

.divider-elegant::before {
  content: '◇';
  position: absolute;
  top: -8px;
  left: 50%;
  transform: translateX(-50%);
  color: white;
  background: linear-gradient(135deg, #667eea, #764ba2);
  padding: 0 10px;
  font-size: 0.8rem;
}

.subtitle {
  font-size: 1.4rem;
  color: rgba(255, 255, 255, 0.95);
  font-weight: 300;
  margin-bottom: 10px;
}

.subtitle-chinese {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  font-weight: 300;
}

/* 搜索区域 */
.search-section {
  text-align: center;
  margin-bottom: 80px;
}

.search-container-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 40px;
}

.search-input-container {
  display: flex;
  align-items: center;
  max-width: 700px;
  width: 100%;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 60px;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  overflow: hidden;
  transition: all 0.3s ease;
}

.search-input-container:hover {
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.25);
  transform: translateY(-2px);
}

.search-icon {
  padding: 0 20px;
  color: #667eea;
  font-size: 1.2rem;
}

.search-input {
  flex: 1;
  padding: 25px 10px;
  border: none;
  font-size: 1.1rem;
  outline: none;
  background: transparent;
  color: #333;
  font-weight: 400;
}

.search-input::placeholder {
  color: #999;
  font-style: italic;
}

.search-button {
  padding: 25px 35px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  border: none;
  color: white;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border-radius: 0 60px 60px 0;
  position: relative;
  overflow: hidden;
}

.search-button::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s ease;
}

.search-button:hover::before {
  left: 100%;
}

.search-button:hover {
  background: linear-gradient(135deg, #5a67d8, #6b46c1);
  transform: scale(1.05);
}

.search-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

.loading-spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.suggested-keywords {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.suggestions-label {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1.1rem;
  font-weight: 300;
}

.keyword-tags-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 15px;
  max-width: 800px;
}

.keyword-tag {
  display: inline-block;
  padding: 12px 24px;
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  color: white;
  font-weight: 400;
  font-size: 0.9rem;
  animation: fadeInUp 0.8s ease forwards;
  opacity: 0;
  transform: translateY(20px);
}

@keyframes fadeInUp {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.keyword-tag:hover {
  background: rgba(255, 255, 255, 0.25);
  border-color: rgba(255, 255, 255, 0.5);
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
}

/* 结果区域 */
.results-section {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 30px;
  padding: 50px;
  box-shadow: 0 25px 70px rgba(0, 0, 0, 0.15);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.loading-state {
  text-align: center;
  padding: 80px 0;
}

.radar-animation {
  font-size: 5rem;
  animation: radarPulse 2s infinite;
  margin-bottom: 30px;
}

@keyframes radarPulse {
  0%, 100% { 
    transform: scale(1) rotate(0deg); 
    opacity: 1; 
  }
  50% { 
    transform: scale(1.2) rotate(180deg); 
    opacity: 0.7; 
  }
}

.loading-state p {
  font-size: 1.3rem;
  color: #667eea;
  font-weight: 300;
}

.analysis-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(380px, 1fr));
  gap: 30px;
  margin-bottom: 50px;
}

.analysis-card {
  background: white;
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  border-left: 5px solid #667eea;
  transition: all 0.3s ease;
}

.analysis-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
}

.card-header h3 {
  color: #333;
  margin-bottom: 25px;
  font-size: 1.4rem;
  font-weight: 600;
}

.card-content ul {
  list-style: none;
  padding: 0;
}

.card-content li {
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
  color: #666;
  line-height: 1.6;
}

.card-content li:last-child {
  border-bottom: none;
}

.market-info {
  background: linear-gradient(135deg, #f8f9ff, #e3f2fd);
  padding: 20px;
  border-radius: 15px;
  margin-bottom: 25px;
}

.market-info p {
  margin: 8px 0;
  color: #555;
  font-weight: 500;
}

/* 导出区域 */
.export-section {
  text-align: center;
  padding-top: 40px;
  border-top: 2px solid #f0f0f0;
}

.export-section h3 {
  color: #333;
  margin-bottom: 30px;
  font-size: 1.5rem;
}

.platform-buttons {
  margin-bottom: 30px;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 15px;
}

.platform-btn {
  padding: 12px 25px;
  background: linear-gradient(135deg, #f0f0f0, #e0e0e0);
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.9rem;
  font-weight: 500;
}

.platform-btn:hover {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  transform: translateY(-2px);
}

.export-button {
  padding: 20px 50px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border: none;
  border-radius: 30px;
  font-size: 1.2rem;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 10px 25px rgba(102, 126, 234, 0.3);
  transition: all 0.3s ease;
}

.export-button:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 15px 35px rgba(102, 126, 234, 0.4);
}

/* 移动端适配 */
@media (max-width: 768px) {
  .home-container {
    padding: 15px;
  }

  .main-title {
    font-size: 2.5rem;
    letter-spacing: 6px;
  }
  
  .analysis-cards {
    grid-template-columns: 1fr;
    gap: 25px;
  }
  
  .search-input-container {
    max-width: 100%;
    border-radius: 50px;
  }
  
  .search-input {
    padding: 20px 10px;
    font-size: 1rem;
  }
  
  .search-button {
    padding: 20px 25px;
  }
  
  .keyword-tags-container {
    gap: 10px;
  }
  
  .keyword-tag {
    padding: 10px 20px;
    font-size: 0.85rem;
  }

  .results-section {
    padding: 30px 20px;
    border-radius: 20px;
  }

  .analysis-card {
    padding: 25px 20px;
  }

  .platform-buttons {
    flex-direction: column;
    align-items: center;
  }

  .platform-btn {
    width: 200px;
  }

  .export-button {
    width: 100%;
    padding: 18px 40px;
    font-size: 1.1rem;
  }
}

@media (max-width: 480px) {
  .main-title {
    font-size: 2rem;
    letter-spacing: 3px;
  }

  .subtitle {
    font-size: 1.1rem;
  }

  .subtitle-chinese {
    font-size: 0.9rem;
  }

  .search-input {
    font-size: 0.95rem;
  }

  .keyword-tag {
    font-size: 0.8rem;
    padding: 8px 16px;
  }

  .logo-icon {
    font-size: 3rem;
  }
}

/* 错误状态样式 */
.error-section {
  display: flex;
  justify-content: center;
  margin: 60px 0;
}

.error-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 40px;
  text-align: center;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.15);
  border-left: 5px solid #f56565;
  max-width: 500px;
  backdrop-filter: blur(10px);
}

.error-card h3 {
  color: #f56565;
  margin-bottom: 20px;
  font-size: 1.3rem;
}

.error-card p {
  color: #666;
  margin-bottom: 25px;
  line-height: 1.6;
}

.retry-button {
  padding: 12px 35px;
  background: linear-gradient(135deg, #f56565, #e53e3e);
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  transition: all 0.3s ease;
}

.retry-button:hover {
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 10px 25px rgba(245, 101, 101, 0.3);
}
</style>
