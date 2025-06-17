<template>
  <div class="app-container">
    <div class="main-content">
      <h1 class="app-title">疾病预测系统</h1>
      
      <!-- 功能切换标签 -->
      <div class="tab-container">
        <button 
          @click="activeTab = 'diabetes'" 
          :class="['tab-btn', { active: activeTab === 'diabetes' }]"
        >
          糖尿病预测
        </button>
        <button 
          @click="activeTab = 'cirrhosis'" 
          :class="['tab-btn', { active: activeTab === 'cirrhosis' }]"
        >
          肝硬化预测
        </button>
        <button 
          @click="activeTab = 'hypertension'" 
          :class="['tab-btn', { active: activeTab === 'hypertension' }]"
        >
          高血压预测
        </button>
      </div>

      <!-- 糖尿病预测模块 -->
      <div v-if="activeTab === 'diabetes'" class="prediction-module">
        <!-- 操作面板 -->
        <div class="control-panel">
          <div class="upload-section">
            <input 
              ref="diabetesFileInput" 
              type="file" 
              accept=".csv" 
              @change="handleDiabetesFileUpload" 
              style="display: none"
            >
            <button @click="$refs.diabetesFileInput.click()" class="btn btn-secondary">
              上传CSV文件
            </button>
            <button @click="addDiabetesRow" class="btn btn-secondary">
              添加行
            </button>
            <button @click="clearDiabetesData" class="btn btn-danger">
              清空数据
            </button>
          </div>
        </div>

        <!-- 糖尿病输入数据表格 -->
        <div class="table-section">
          <h3>糖尿病预测输入数据</h3>
          <div class="table-container">
            <table class="data-table">
              <thead>
                <tr>
                  <th>序号</th>
                  <th>性别</th>
                  <th>年龄</th>
                  <th>高血压</th>
                  <th>心脏病</th>
                  <th>吸烟史</th>
                  <th>BMI</th>
                  <th>糖化血红蛋白</th>
                  <th>血糖水平</th>
                  <th>操作</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, index) in diabetesInputData" :key="index">
                  <td>{{ index + 1 }}</td>
                  <td>
                    <select v-model="row.gender" class="form-input">
                      <option value="Male">Male</option>
                      <option value="Female">Female</option>
                    </select>
                  </td>
                  <td>
                    <input 
                      v-model.number="row.age" 
                      type="number" 
                      class="form-input" 
                      min="0" 
                      max="120"
                    >
                  </td>
                  <td>
                    <select v-model.number="row.hypertension" class="form-input">
                      <option :value="0">无</option>
                      <option :value="1">有</option>
                    </select>
                  </td>
                  <td>
                    <select v-model.number="row.heart_disease" class="form-input">
                      <option :value="0">无</option>
                      <option :value="1">有</option>
                    </select>
                  </td>
                  <td>
                    <select v-model="row.smoking_history" class="form-input">
                      <option value="never">never</option>
                      <option value="former">former</option>
                      <option value="current">current</option>
                      <option value="not current">not current</option>
                      <option value="ever">ever</option>
                      <option value="No Info">No Info</option>
                    </select>
                  </td>
                  <td>
                    <input 
                      v-model.number="row.bmi" 
                      type="number" 
                      class="form-input" 
                      step="0.1" 
                      min="0"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.HbA1c_level" 
                      type="number" 
                      class="form-input" 
                      step="0.1" 
                      min="0"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.blood_glucose_level" 
                      type="number" 
                      class="form-input" 
                      min="0"
                    >
                  </td>
                  <td>
                    <button @click="removeDiabetesRow(index)" class="btn btn-danger-small">
                      删除
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- 糖尿病预测按钮 -->
        <div class="predict-section">
          <button 
            @click="predictDiabetes" 
            :disabled="diabetesLoading || diabetesInputData.length === 0" 
            class="btn btn-primary predict-btn"
          >
            {{ diabetesLoading ? '预测中...' : '开始糖尿病预测' }}
          </button>
        </div>

        <!-- 糖尿病预测结果 -->
        <div v-if="diabetesPredictions.length > 0" class="table-section">
          <h3>糖尿病预测结果</h3>
          <div class="result-summary">
            <span class="summary-item">
              总数: {{ diabetesSummary.total_count }}
            </span>
            <span class="summary-item positive">
              阳性: {{ diabetesSummary.positive_count }}
            </span>
            <span class="summary-item negative">
              阴性: {{ diabetesSummary.negative_count }}
            </span>
          </div>
          <div class="table-container">
            <table class="data-table">
              <thead>
                <tr>
                  <th>序号</th>
                  <th>性别</th>
                  <th>年龄</th>
                  <th>高血压</th>
                  <th>心脏病</th>
                  <th>吸烟史</th>
                  <th>BMI</th>
                  <th>糖化血红蛋白</th>
                  <th>血糖水平</th>
                  <th>预测结果</th>
                  <th>患病概率</th>
                  <th>置信度</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(result, index) in diabetesPredictions" :key="index">
                  <td>{{ index + 1 }}</td>
                  <td>{{ result.gender }}</td>
                  <td>{{ result.age }}</td>
                  <td>{{ result.hypertension ? '有' : '无' }}</td>
                  <td>{{ result.heart_disease ? '有' : '无' }}</td>
                  <td>{{ result.smoking_history }}</td>
                  <td>{{ result.bmi }}</td>
                  <td>{{ result.HbA1c_level }}</td>
                  <td>{{ result.blood_glucose_level }}</td>
                  <td>
                    <span :class="['prediction-result', result.diabetes_prediction ? 'positive' : 'negative']">
                      {{ result.diabetes_prediction ? '阳性' : '阴性' }}
                    </span>
                  </td>
                  <td>{{ (result.diabetes_probability * 100).toFixed(2) }}%</td>
                  <td>{{ (result.prediction_confidence * 100).toFixed(2) }}%</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- 肝硬化预测模块 -->
      <div v-if="activeTab === 'cirrhosis'" class="prediction-module">
        <!-- 操作面板 -->
        <div class="control-panel">
          <div class="upload-section">
            <input 
              ref="cirrhosisFileInput" 
              type="file" 
              accept=".csv" 
              @change="handleCirrhosisFileUpload" 
              style="display: none"
            >
            <button @click="$refs.cirrhosisFileInput.click()" class="btn btn-secondary">
              上传CSV文件
            </button>
            <button @click="addCirrhosisRow" class="btn btn-secondary">
              添加行
            </button>
            <button @click="clearCirrhosisData" class="btn btn-danger">
              清空数据
            </button>
          </div>
        </div>

        <!-- 肝硬化输入数据表格 -->
        <div class="table-section">
          <h3>肝硬化预测输入数据</h3>
          <div class="table-container">
            <table class="data-table cirrhosis-table">
              <thead>
                <tr>
                  <th>序号</th>
                  <th>年龄</th>
                  <th>药物</th>
                  <th>性别</th>
                  <th>腹水</th>
                  <th>肝肿大</th>
                  <th>蜘蛛痣</th>
                  <th>水肿</th>
                  <th>胆红素</th>
                  <th>胆固醇</th>
                  <th>白蛋白</th>
                  <th>铜</th>
                  <th>碱性磷酸酶</th>
                  <th>SGOT</th>
                  <th>甘油三酯</th>
                  <th>血小板</th>
                  <th>凝血酶原</th>
                  <th>观察天数</th>
                  <th>操作</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, index) in cirrhosisInputData" :key="index">
                  <td>{{ index + 1 }}</td>
                  <td>
                    <input 
                      v-model.number="row.Age" 
                      type="number" 
                      class="form-input" 
                      min="0" 
                      max="120"
                    >
                  </td>
                  <td>
                    <select v-model="row.Drug" class="form-input">
                      <option value="D-penicillamine">D-penicillamine</option>
                      <option value="Placebo">Placebo</option>
                    </select>
                  </td>
                  <td>
                    <select v-model="row.Sex" class="form-input">
                      <option value="M">男</option>
                      <option value="F">女</option>
                    </select>
                  </td>
                  <td>
                    <select v-model="row.Ascites" class="form-input">
                      <option value="N">无</option>
                      <option value="Y">有</option>
                    </select>
                  </td>
                  <td>
                    <select v-model="row.Hepatomegaly" class="form-input">
                      <option value="N">无</option>
                      <option value="Y">有</option>
                    </select>
                  </td>
                  <td>
                    <select v-model="row.Spiders" class="form-input">
                      <option value="N">无</option>
                      <option value="Y">有</option>
                    </select>
                  </td>
                  <td>
                    <select v-model="row.Edema" class="form-input">
                      <option value="N">无</option>
                      <option value="S">轻度</option>
                      <option value="Y">严重</option>
                    </select>
                  </td>
                  <td>
                    <input 
                      v-model.number="row.Bilirubin" 
                      type="number" 
                      class="form-input" 
                      step="0.1" 
                      min="0"
                      placeholder="mg/dl"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.Cholesterol" 
                      type="number" 
                      class="form-input" 
                      min="0"
                      placeholder="mg/dl"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.Albumin" 
                      type="number" 
                      class="form-input" 
                      step="0.1" 
                      min="0"
                      placeholder="g/dl"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.Copper" 
                      type="number" 
                      class="form-input" 
                      min="0"
                      placeholder="μg/day"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.Alk_Phos" 
                      type="number" 
                      class="form-input" 
                      step="0.1" 
                      min="0"
                      placeholder="U/L"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.SGOT" 
                      type="number" 
                      class="form-input" 
                      step="0.1" 
                      min="0"
                      placeholder="U/ml"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.Tryglicerides" 
                      type="number" 
                      class="form-input" 
                      min="0"
                      placeholder="mg/dl"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.Platelets" 
                      type="number" 
                      class="form-input" 
                      min="0"
                      placeholder="cubic ml/1000"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.Prothrombin" 
                      type="number" 
                      class="form-input" 
                      step="0.1" 
                      min="0"
                      placeholder="seconds"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.N_Days" 
                      type="number" 
                      class="form-input" 
                      min="0"
                      placeholder="天数"
                    >
                  </td>
                  <td>
                    <button @click="removeCirrhosisRow(index)" class="btn btn-danger-small">
                      删除
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- 肝硬化预测按钮 -->
        <div class="predict-section">
          <button 
            @click="predictCirrhosis" 
            :disabled="cirrhosisLoading || cirrhosisInputData.length === 0" 
            class="btn btn-primary predict-btn"
          >
            {{ cirrhosisLoading ? '预测中...' : '开始肝硬化预测' }}
          </button>
        </div>

        <!-- 肝硬化预测结果 -->
        <div v-if="cirrhosisPredictions.length > 0" class="table-section">
          <h3>肝硬化预测结果</h3>
          <div class="result-summary">
            <span class="summary-item">
              总数: {{ cirrhosisSummary.total_count }}
            </span>
            <span v-for="(count, stage) in cirrhosisSummary.stage_distribution" :key="stage" class="summary-item stage-item">
              {{ stage }}期: {{ count }}
            </span>
          </div>
          <div class="table-container">
            <table class="data-table cirrhosis-result-table">
              <thead>
                <tr>
                  <th>序号</th>
                  <th>年龄</th>
                  <th>性别</th>
                  <th>药物</th>
                  <th>胆红素</th>
                  <th>白蛋白</th>
                  <th>预测分期</th>
                  <th>置信度</th>
                  <th>各期概率</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(result, index) in cirrhosisPredictions" :key="index">
                  <td>{{ index + 1 }}</td>
                  <td>{{ result.Age }}</td>
                  <td>{{ result.Sex === 'M' ? '男' : '女' }}</td>
                  <td>{{ result.Drug }}</td>
                  <td>{{ result.Bilirubin }}</td>
                  <td>{{ result.Albumin }}</td>
                  <td>
                    <span :class="['stage-result', `stage-${result.cirrhosis_stage}`]">
                      {{ result.cirrhosis_stage }}期
                    </span>
                  </td>
                  <td>{{ (result.prediction_confidence * 100).toFixed(2) }}%</td>
                  <td class="probabilities-cell">
                    <div v-for="(prob, stage) in result.stage_probabilities" :key="stage" class="prob-item">
                      {{ stage }}: {{ (prob * 100).toFixed(1) }}%
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- 高血压预测模块 -->
      <div v-if="activeTab === 'hypertension'" class="prediction-module">
        <!-- 操作面板 -->
        <div class="control-panel">
          <div class="upload-section">
            <input 
              ref="hypertensionFileInput" 
              type="file" 
              accept=".csv" 
              @change="handleHypertensionFileUpload" 
              style="display: none"
            >
            <button @click="$refs.hypertensionFileInput.click()" class="btn btn-secondary">
              上传CSV文件
            </button>
            <button @click="addHypertensionRow" class="btn btn-secondary">
              添加行
            </button>
            <button @click="clearHypertensionData" class="btn btn-danger">
              清空数据
            </button>
          </div>
        </div>

        <!-- 高血压输入数据表格 -->
        <div class="table-section">
          <h3>高血压预测输入数据</h3>
          <div class="table-container">
            <table class="data-table">
              <thead>
                <tr>
                  <th>序号</th>
                  <th>性别</th>
                  <th>年龄</th>
                  <th>糖尿病</th>
                  <th>心脏病</th>
                  <th>吸烟史</th>
                  <th>BMI</th>
                  <th>糖化血红蛋白</th>
                  <th>血糖水平</th>
                  <th>操作</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, index) in hypertensionInputData" :key="index">
                  <td>{{ index + 1 }}</td>
                  <td>
                    <select v-model="row.gender" class="form-input">
                      <option value="Male">Male</option>
                      <option value="Female">Female</option>
                    </select>
                  </td>
                  <td>
                    <input 
                      v-model.number="row.age" 
                      type="number" 
                      class="form-input" 
                      min="0" 
                      max="120"
                    >
                  </td>
                  <td>
                    <select v-model.number="row.diabetes" class="form-input">
                      <option :value="0">无</option>
                      <option :value="1">有</option>
                    </select>
                  </td>
                  <td>
                    <select v-model.number="row.heart_disease" class="form-input">
                      <option :value="0">无</option>
                      <option :value="1">有</option>
                    </select>
                  </td>
                  <td>
                    <select v-model="row.smoking_history" class="form-input">
                      <option value="never">never</option>
                      <option value="former">former</option>
                      <option value="current">current</option>
                      <option value="not current">not current</option>
                      <option value="ever">ever</option>
                      <option value="No Info">No Info</option>
                    </select>
                  </td>
                  <td>
                    <input 
                      v-model.number="row.bmi" 
                      type="number" 
                      class="form-input" 
                      step="0.1" 
                      min="0"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.HbA1c_level" 
                      type="number" 
                      class="form-input" 
                      step="0.1" 
                      min="0"
                    >
                  </td>
                  <td>
                    <input 
                      v-model.number="row.blood_glucose_level" 
                      type="number" 
                      class="form-input" 
                      min="0"
                    >
                  </td>
                  <td>
                    <button @click="removeHypertensionRow(index)" class="btn btn-danger-small">
                      删除
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- 高血压预测按钮 -->
        <div class="predict-section">
          <button 
            @click="predictHypertension" 
            :disabled="hypertensionLoading || hypertensionInputData.length === 0" 
            class="btn btn-primary predict-btn"
          >
            {{ hypertensionLoading ? '预测中...' : '开始高血压预测' }}
          </button>
        </div>

        <!-- 高血压预测结果 -->
        <div v-if="hypertensionPredictions.length > 0" class="table-section">
          <h3>高血压预测结果</h3>
          <div class="result-summary">
            <span class="summary-item">
              总数: {{ hypertensionSummary.total_count }}
            </span>
            <span class="summary-item positive">
              阳性: {{ hypertensionSummary.positive_count }}
            </span>
            <span class="summary-item negative">
              阴性: {{ hypertensionSummary.negative_count }}
            </span>
          </div>
          <div class="table-container">
            <table class="data-table">
              <thead>
                <tr>
                  <th>序号</th>
                  <th>性别</th>
                  <th>年龄</th>
                  <th>糖尿病</th>
                  <th>心脏病</th>
                  <th>吸烟史</th>
                  <th>BMI</th>
                  <th>糖化血红蛋白</th>
                  <th>血糖水平</th>
                  <th>预测结果</th>
                  <th>患病概率</th>
                  <th>置信度</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(result, index) in hypertensionPredictions" :key="index">
                  <td>{{ index + 1 }}</td>
                  <td>{{ result.gender }}</td>
                  <td>{{ result.age }}</td>
                  <td>{{ result.diabetes ? '有' : '无' }}</td>
                  <td>{{ result.heart_disease ? '有' : '无' }}</td>
                  <td>{{ result.smoking_history }}</td>
                  <td>{{ result.bmi }}</td>
                  <td>{{ result.HbA1c_level }}</td>
                  <td>{{ result.blood_glucose_level }}</td>
                  <td>
                    <span :class="['prediction-result', result.hypertension_prediction ? 'positive' : 'negative']">
                      {{ result.hypertension_prediction ? '阳性' : '阴性' }}
                    </span>
                  </td>
                  <td>{{ (result.hypertension_probability * 100).toFixed(2) }}%</td>
                  <td>{{ (result.prediction_confidence * 100).toFixed(2) }}%</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- 错误信息 -->
      <div v-if="error" class="error-message">
        {{ error }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

// 当前激活的标签页
const activeTab = ref('diabetes')

// 错误信息
const error = ref('')

// API基础URL
const DIABETES_API_BASE_URL = 'http://localhost:5000/api'
const CIRRHOSIS_API_BASE_URL = 'http://localhost:5001/api/cirrhosis'
const HYPERTENSION_API_BASE_URL = 'http://localhost:5002/api/hypertension'

// ============= 糖尿病相关数据 =============
const diabetesInputData = ref([])
const diabetesPredictions = ref([])
const diabetesSummary = reactive({
  total_count: 0,
  positive_count: 0,
  negative_count: 0
})
const diabetesLoading = ref(false)
const diabetesFileInput = ref(null)

// ============= 肝硬化相关数据 =============
const cirrhosisInputData = ref([])
const cirrhosisPredictions = ref([])
const cirrhosisSummary = reactive({
  total_count: 0,
  stage_distribution: {}
})
const cirrhosisLoading = ref(false)
const cirrhosisFileInput = ref(null)

// ============= 高血压相关数据 =============
const hypertensionInputData = ref([])
const hypertensionPredictions = ref([])
const hypertensionSummary = reactive({
  total_count: 0,
  positive_count: 0,
  negative_count: 0
})
const hypertensionLoading = ref(false)
const hypertensionFileInput = ref(null)

// ============= 糖尿病相关方法 =============
const getDefaultDiabetesRow = () => ({
  gender: 'Female',
  age: 30,
  hypertension: 0,
  heart_disease: 0,
  smoking_history: 'never',
  bmi: 25.0,
  HbA1c_level: 5.5,
  blood_glucose_level: 100
})

const initializeDiabetesData = () => {
  diabetesInputData.value = [getDefaultDiabetesRow()]
}

const addDiabetesRow = () => {
  diabetesInputData.value.push(getDefaultDiabetesRow())
}

const removeDiabetesRow = (index) => {
  if (diabetesInputData.value.length > 1) {
    diabetesInputData.value.splice(index, 1)
  }
}

const clearDiabetesData = () => {
  diabetesInputData.value = [getDefaultDiabetesRow()]
  diabetesPredictions.value = []
  error.value = ''
  Object.assign(diabetesSummary, {
    total_count: 0,
    positive_count: 0,
    negative_count: 0
  })
}

const handleDiabetesFileUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return

  const formData = new FormData()
  formData.append('file', file)

  try {
    diabetesLoading.value = true
    error.value = ''
    
    const response = await fetch(`${DIABETES_API_BASE_URL}/upload`, {
      method: 'POST',
      body: formData
    })

    if (!response.ok) {
      throw new Error('文件上传失败')
    }

    const result = await response.json()
    if (result.success) {
      diabetesInputData.value = result.data
      diabetesPredictions.value = []
      Object.assign(diabetesSummary, {
        total_count: 0,
        positive_count: 0,
        negative_count: 0
      })
    } else {
      throw new Error(result.error || '文件处理失败')
    }
  } catch (err) {
    error.value = `糖尿病文件上传失败: ${err.message}`
  } finally {
    diabetesLoading.value = false
    if (diabetesFileInput.value) {
      diabetesFileInput.value.value = ''
    }
  }
}

const validateDiabetesInputData = () => {
  for (let i = 0; i < diabetesInputData.value.length; i++) {
    const row = diabetesInputData.value[i]
    if (!row.gender || row.age === null || row.age === undefined || 
        row.hypertension === null || row.hypertension === undefined ||
        row.heart_disease === null || row.heart_disease === undefined ||
        !row.smoking_history || row.bmi === null || row.bmi === undefined ||
        row.HbA1c_level === null || row.HbA1c_level === undefined ||
        row.blood_glucose_level === null || row.blood_glucose_level === undefined) {
      throw new Error(`第 ${i + 1} 行数据不完整，请检查所有字段`)
    }
  }
}

const predictDiabetes = async () => {
  try {
    diabetesLoading.value = true
    error.value = ''
    
    validateDiabetesInputData()

    const response = await fetch(`${DIABETES_API_BASE_URL}/predict`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        data: diabetesInputData.value
      })
    })

    if (!response.ok) {
      throw new Error('糖尿病预测请求失败')
    }

    const result = await response.json()
    if (result.success) {
      diabetesPredictions.value = result.predictions
      Object.assign(diabetesSummary, {
        total_count: result.total_count,
        positive_count: result.positive_count,
        negative_count: result.negative_count
      })
    } else {
      throw new Error(result.error || '糖尿病预测失败')
    }
  } catch (err) {
    error.value = `糖尿病预测失败: ${err.message}`
  } finally {
    diabetesLoading.value = false
  }
}

// ============= 肝硬化相关方法 =============
const getDefaultCirrhosisRow = () => ({
  Age: 50,
  Drug: 'D-penicillamine',
  Sex: 'F',
  Ascites: 'N',
  Hepatomegaly: 'N',
  Spiders: 'N',
  Edema: 'N',
  Bilirubin: 1.0,
  Cholesterol: 200,
  Albumin: 3.5,
  Copper: 100,
  Alk_Phos: 100,
  SGOT: 30,
  Tryglicerides: 150,
  Platelets: 250,
  Prothrombin: 11.0,
  N_Days: 1000
})

const initializeCirrhosisData = () => {
  cirrhosisInputData.value = [getDefaultCirrhosisRow()]
}

const addCirrhosisRow = () => {
  cirrhosisInputData.value.push(getDefaultCirrhosisRow())
}

const removeCirrhosisRow = (index) => {
  if (cirrhosisInputData.value.length > 1) {
    cirrhosisInputData.value.splice(index, 1)
  }
}

const clearCirrhosisData = () => {
  cirrhosisInputData.value = [getDefaultCirrhosisRow()]
  cirrhosisPredictions.value = []
  error.value = ''
  Object.assign(cirrhosisSummary, {
    total_count: 0,
    stage_distribution: {}
  })
}

const handleCirrhosisFileUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return

  const formData = new FormData()
  formData.append('file', file)

  try {
    cirrhosisLoading.value = true
    error.value = ''
    
    const response = await fetch(`${CIRRHOSIS_API_BASE_URL}/upload`, {
      method: 'POST',
      body: formData
    })

    if (!response.ok) {
      throw new Error('文件上传失败')
    }

    const result = await response.json()
    if (result.success) {
      cirrhosisInputData.value = result.data
      cirrhosisPredictions.value = []
      Object.assign(cirrhosisSummary, {
        total_count: 0,
        stage_distribution: {}
      })
    } else {
      throw new Error(result.error || '文件处理失败')
    }
  } catch (err) {
    error.value = `肝硬化文件上传失败: ${err.message}`
  } finally {
    cirrhosisLoading.value = false
    if (cirrhosisFileInput.value) {
      cirrhosisFileInput.value.value = ''
    }
  }
}

const validateCirrhosisInputData = () => {
  const requiredFields = ['Age', 'Drug', 'Sex', 'Ascites', 'Hepatomegaly', 'Spiders', 
                         'Edema', 'Bilirubin', 'Cholesterol', 'Albumin', 'Copper', 
                         'Alk_Phos', 'SGOT', 'Tryglicerides', 'Platelets', 'Prothrombin', 'N_Days']
  
  for (let i = 0; i < cirrhosisInputData.value.length; i++) {
    const row = cirrhosisInputData.value[i]
    for (const field of requiredFields) {
      if (row[field] === null || row[field] === undefined || row[field] === '') {
        throw new Error(`第 ${i + 1} 行的 ${field} 字段不能为空`)
      }
    }
  }
}

const predictCirrhosis = async () => {
  try {
    cirrhosisLoading.value = true
    error.value = ''
    
    validateCirrhosisInputData()

    const response = await fetch(`${CIRRHOSIS_API_BASE_URL}/predict`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        data: cirrhosisInputData.value
      })
    })

    if (!response.ok) {
      throw new Error('肝硬化预测请求失败')
    }

    const result = await response.json()
    if (result.success) {
      cirrhosisPredictions.value = result.predictions
      Object.assign(cirrhosisSummary, {
        total_count: result.total_count,
        stage_distribution: result.stage_distribution
      })
    } else {
      throw new Error(result.error || '肝硬化预测失败')
    }
  } catch (err) {
    error.value = `肝硬化预测失败: ${err.message}`
  } finally {
    cirrhosisLoading.value = false
  }
}

// ============= 高血压相关方法 =============
const getDefaultHypertensionRow = () => ({
  gender: 'Female',
  age: 30,
  diabetes: 0,
  heart_disease: 0,
  smoking_history: 'never',
  bmi: 25.0,
  HbA1c_level: 5.5,
  blood_glucose_level: 100
})

const initializeHypertensionData = () => {
  hypertensionInputData.value = [getDefaultHypertensionRow()]
}

const addHypertensionRow = () => {
  hypertensionInputData.value.push(getDefaultHypertensionRow())
}

const removeHypertensionRow = (index) => {
  if (hypertensionInputData.value.length > 1) {
    hypertensionInputData.value.splice(index, 1)
  }
}

const clearHypertensionData = () => {
  hypertensionInputData.value = [getDefaultHypertensionRow()]
  hypertensionPredictions.value = []
  error.value = ''
  Object.assign(hypertensionSummary, {
    total_count: 0,
    positive_count: 0,
    negative_count: 0
  })
}

const handleHypertensionFileUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return

  const formData = new FormData()
  formData.append('file', file)

  try {
    hypertensionLoading.value = true
    error.value = ''
    
    const response = await fetch(`${HYPERTENSION_API_BASE_URL}/upload`, {
      method: 'POST',
      body: formData
    })

    if (!response.ok) {
      throw new Error('文件上传失败')
    }

    const result = await response.json()
    if (result.success) {
      hypertensionInputData.value = result.data
      hypertensionPredictions.value = []
      Object.assign(hypertensionSummary, {
        total_count: 0,
        positive_count: 0,
        negative_count: 0
      })
    } else {
      throw new Error(result.error || '文件处理失败')
    }
  } catch (err) {
    error.value = `高血压文件上传失败: ${err.message}`
  } finally {
    hypertensionLoading.value = false
    if (hypertensionFileInput.value) {
      hypertensionFileInput.value.value = ''
    }
  }
}

const validateHypertensionInputData = () => {
  for (let i = 0; i < hypertensionInputData.value.length; i++) {
    const row = hypertensionInputData.value[i]
    if (!row.gender || row.age === null || row.age === undefined || 
        row.diabetes === null || row.diabetes === undefined ||
        row.heart_disease === null || row.heart_disease === undefined ||
        !row.smoking_history || row.bmi === null || row.bmi === undefined ||
        row.HbA1c_level === null || row.HbA1c_level === undefined ||
        row.blood_glucose_level === null || row.blood_glucose_level === undefined) {
      throw new Error(`第 ${i + 1} 行数据不完整，请检查所有字段`)
    }
  }
}

const predictHypertension = async () => {
  try {
    hypertensionLoading.value = true
    error.value = ''
    
    validateHypertensionInputData()

    const response = await fetch(`${HYPERTENSION_API_BASE_URL}/predict`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        data: hypertensionInputData.value
      })
    })

    if (!response.ok) {
      throw new Error('高血压预测请求失败')
    }

    const result = await response.json()
    if (result.success) {
      hypertensionPredictions.value = result.predictions
      Object.assign(hypertensionSummary, {
        total_count: result.total_count,
        positive_count: result.positive_count,
        negative_count: result.negative_count
      })
    } else {
      throw new Error(result.error || '高血压预测失败')
    }
  } catch (err) {
    error.value = `高血压预测失败: ${err.message}`
  } finally {
    hypertensionLoading.value = false
  }
}

// 初始化数据
initializeDiabetesData()
initializeCirrhosisData()
initializeHypertensionData()
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  overflow-x: auto;
}

.app-container {
  min-height: 100vh;
  width: 100vw;
  background: linear-gradient(135deg, #fff3cd, #fff8dc, #ffeaa7);
  padding: 20px;
  box-sizing: border-box;
}

.main-content {
  width: 100%;
  max-width: none;
  background: white;
  border-radius: 12px;
  box-shadow: 0 8px 32px rgba(255, 193, 7, 0.3);
  padding: 30px;
  box-sizing: border-box;
  margin: 0 auto;
}

.app-title {
  text-align: center;
  color: #e67e22;
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 30px;
  text-shadow: 2px 2px 4px rgba(230, 126, 34, 0.3);
}

/* 标签页样式 */
.tab-container {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
  border-bottom: 2px solid #f8f9fa;
}

.tab-btn {
  padding: 15px 30px;
  background: transparent;
  border: none;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border-bottom: 3px solid transparent;
  color: #666;
}

.tab-btn:hover {
  color: #e67e22;
  background: rgba(230, 126, 34, 0.1);
}

.tab-btn.active {
  color: #e67e22;
  border-bottom-color: #e67e22;
  background: rgba(230, 126, 34, 0.1);
}

.prediction-module {
  animation: fadeIn 0.3s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.control-panel {
  margin-bottom: 30px;
}

.upload-section {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
  justify-content: center;
}

.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  text-transform: none;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.btn-primary {
  background: linear-gradient(135deg, #f39c12, #e67e22);
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: linear-gradient(135deg, #e67e22, #d35400);
}

.btn-secondary {
  background: linear-gradient(135deg, #ffc107, #ffb300);
  color: #333;
}

.btn-secondary:hover {
  background: linear-gradient(135deg, #ffb300, #ff8f00);
}

.btn-danger {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
  color: white;
}

.btn-danger:hover {
  background: linear-gradient(135deg, #c0392b, #a93226);
}

.btn-danger-small {
  padding: 6px 12px;
  font-size: 12px;
  background: #e74c3c;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-danger-small:hover {
  background: #c0392b;
}

.table-section {
  margin-bottom: 30px;
}

.table-section h3 {
  color: #e67e22;
  font-size: 1.5rem;
  margin-bottom: 15px;
  text-align: center;
}

.table-container {
  overflow-x: auto;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(255, 193, 7, 0.2);
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  table-layout: auto;
}

.data-table th {
  background: linear-gradient(135deg, #f39c12, #e67e22);
  color: white;
  padding: 12px 6px;
  text-align: center;
  font-weight: 600;
  border-bottom: 2px solid #d35400;
  font-size: 13px;
  white-space: nowrap;
}

.data-table td {
  padding: 8px 4px;
  text-align: center;
  border-bottom: 1px solid #f8f9fa;
  font-size: 12px;
}

.data-table tbody tr:hover {
  background-color: #fff8dc;
}

/* 肝硬化表格特殊样式 */
.cirrhosis-table th,
.cirrhosis-table td {
  font-size: 11px;
  padding: 6px 3px;
}

.cirrhosis-result-table th,
.cirrhosis-result-table td {
  font-size: 11px;
  padding: 6px 4px;
}

.form-input {
  width: 100%;
  max-width: 120px;
  padding: 4px 6px;
  border: 2px solid #ffc107;
  border-radius: 4px;
  font-size: 12px;
  box-sizing: border-box;
  transition: border-color 0.3s ease;
}

.form-input:focus {
  outline: none;
  border-color: #e67e22;
  box-shadow: 0 0 5px rgba(230, 126, 34, 0.3);
}

.predict-section {
  text-align: center;
  margin: 30px 0;
}

.predict-btn {
  font-size: 18px;
  padding: 15px 40px;
  min-width: 200px;
}

.error-message {
  background: #ffebee;
  color: #c62828;
  padding: 15px;
  border-radius: 8px;
  border-left: 4px solid #e74c3c;
  margin: 20px 0;
  text-align: center;
  font-weight: 500;
}

.result-summary {
  display: flex;
  justify-content: center;
  gap: 30px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.summary-item {
  padding: 10px 20px;
  border-radius: 20px;
  font-weight: 600;
  color: white;
  background: linear-gradient(135deg, #95a5a6, #7f8c8d);
}

.summary-item.positive {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
}

.summary-item.negative {
  background: linear-gradient(135deg, #27ae60, #229954);
}

.summary-item.stage-item {
  background: linear-gradient(135deg, #3498db, #2980b9);
}

.prediction-result {
  padding: 6px 12px;
  border-radius: 12px;
  font-weight: 600;
  color: white;
}

.prediction-result.positive {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
}

.prediction-result.negative {
  background: linear-gradient(135deg, #27ae60, #229954);
}

/* 肝硬化分期结果样式 */
.stage-result {
  padding: 6px 12px;
  border-radius: 12px;
  font-weight: 600;
  color: white;
}

.stage-result.stage-1 {
  background: linear-gradient(135deg, #27ae60, #229954);
}

.stage-result.stage-2 {
  background: linear-gradient(135deg, #f39c12, #e67e22);
}

.stage-result.stage-3 {
  background: linear-gradient(135deg, #e67e22, #d35400);
}

.stage-result.stage-4 {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
}

.probabilities-cell {
  font-size: 10px;
  max-width: 150px;
}

.prob-item {
  margin: 2px 0;
  padding: 2px 4px;
  background: rgba(52, 152, 219, 0.1);
  border-radius: 3px;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .upload-section {
    justify-content: flex-start;
    flex-wrap: wrap;
  }
  
  .data-table th,
  .data-table td {
    font-size: 11px;
    padding: 6px 3px;
  }
  
  .form-input {
    max-width: 100px;
    font-size: 11px;
  }
}

@media (max-width: 768px) {
  .app-container {
    padding: 10px;
  }
  
  .main-content {
    padding: 20px;
  }
  
  .app-title {
    font-size: 2rem;
  }
  
  .tab-container {
    flex-direction: column;
    align-items: center;
  }
  
  .tab-btn {
    width: 200px;
    margin: 5px 0;
  }
  
  .upload-section {
    justify-content: center;
  }
  
  .btn {
    padding: 10px 20px;
    font-size: 14px;
  }
  
  .result-summary {
    flex-direction: column;
    align-items: center;
    gap: 10px;
  }
  
  .table-container {
    overflow-x: auto;
  }
  
  .data-table {
    min-width: 800px;
  }
  
  .cirrhosis-table {
    min-width: 1400px;
  }
  
  .cirrhosis-result-table {
    min-width: 1000px;
  }
}
</style>