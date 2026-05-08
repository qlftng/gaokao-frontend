<template>
  <div>

    <!-- ① AI 动画过渡页 -->
    <div v-if="analyzing" class="ai-page">
      <div class="ai-orb">
        <div class="orb-ring r1"></div>
        <div class="orb-ring r2"></div>
        <div class="orb-ring r3"></div>
        <div class="orb-core">
          <span class="orb-icon">🧠</span>
        </div>
      </div>
      <div class="ai-label">正在为你制定</div>
      <div class="ai-title">专属学习计划</div>
      <div class="ai-dots">
        <span></span><span></span><span></span>
      </div>
      <div class="ai-steps">
        <div class="ai-step" :class="{ done: aiStep >= 1 }">✦ 分析你的学科诊断数据</div>
        <div class="ai-step" :class="{ done: aiStep >= 2 }">✦ 匹配最优提分路径</div>
        <div class="ai-step" :class="{ done: aiStep >= 3 }">✦ 生成21天专属计划</div>
      </div>
    </div>

    <!-- ② 成功页 -->
    <div v-else-if="submitted" class="success-page">
      <div class="success-badge">计划已生成</div>
      <div class="success-icon">🎯</div>
      <h2>专属计划已就绪！</h2>
      <p class="success-desc">
        我们已为你制定21天专属学习计划<br>
        请长按二维码添加企业微信好友，计划将在24小时内送达
      </p>
      <button class="fix-btn" @click="goBack">📝 我填错了，重新填写</button>

      <!-- 二维码区域 -->
      <div class="qr-card">
        <div class="qr-label">长按二维码添加企业微信好友</div>
        <!-- 把你的二维码图片放到 src/assets/qrcode.png，或直接替换下面的 src -->
        <div class="qr-wrap">
          <a v-if="!qrFallback" href="/qrcode.png" target="_blank" title="点击查看大图">
            <img
              src="/qrcode.png"
              alt="企业微信二维码"
              class="qr-img"
              @error="qrFallback = true"
                          />
          </a>
          <!-- 图片加载失败时的占位 -->
          <div class="qr-placeholder" v-else>
            <div class="qr-placeholder-icon">📱</div>
            <div class="qr-placeholder-text">二维码图片<br>请替换 public/qrcode.png</div>
          </div>
        </div>
        <div class="qr-hint">长按保存或扫描二维码 · 点击可放大 · 添加后发送「学习计划」</div>
      </div>
    </div>

    <!-- ③ 问卷主体 -->
    <div v-else>
      <!-- 顶部横幅 -->
      <div class="banner">
        <div class="banner-tag">2026 高考备战</div>
        <h1>学习诊断问卷</h1>
        <p>帮助我们了解你的学习情况<br>为你定制21天专属提升计划</p>
      </div>

      <!-- 进度条 -->
      <div class="progress-wrap">
        <div class="progress-label">
          <span>答题进度</span>
          <span>{{ answeredCount }} / {{ totalCount }}</span>
        </div>
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: progressPct + '%' }"></div>
        </div>
      </div>

      <div class="container">

        <!-- 基本信息 -->
        <div class="section-title">基本信息</div>

        <div class="card">
          <div class="q-num">Q1</div>
          <div class="q-text">请填写你的个人信息</div>
          <div class="row-2">
            <input class="text-input" v-model="form.name" placeholder="你的姓名（必填）">
            <div class="input-with-tip">
              <input class="text-input" v-model="form.wx_name" placeholder="微信昵称">
              <span class="tip-icon" title="请填写你微信上的真实名称，方便我们准确找到你">?</span>
            </div>
          </div>
          <div class="row-2" style="margin-top: 8px;">
            <input class="text-input" v-model="form.phone" type="tel" placeholder="手机号（必填）">
          </div>
        </div>

        <div class="card">
          <div class="q-num">📷</div>
          <div class="q-text">上传微信主页截图<span class="q-sub">（选填）</span></div>
          <p class="q-sub" style="margin-bottom: 12px;">
            请截取你的微信「我」页面，方便我们准确找到你并发送学习计划
          </p>
          <div class="upload-area" @click="triggerUpload" :class="{ has: screenshotPreview }">
            <img v-if="screenshotPreview" :src="screenshotPreview" class="preview-img" />
            <div v-else class="upload-placeholder">
              <span class="upload-icon">📁</span>
              <span>点击上传截图</span>
              <span class="upload-hint">支持 JPG/PNG，不超过 5MB</span>
            </div>
          </div>
          <input ref="fileInput" type="file" accept="image/jpeg,image/png,image/webp"
            style="display:none" @change="onFileChange" />
          <div v-if="screenshotPreview" class="upload-actions">
            <button type="button" class="retry-btn" @click="clearScreenshot">重新上传</button>
          </div>
        </div>

        <div class="card">
          <div class="q-num">Q2</div>
          <div class="q-text">你所在的省份是？</div>
          <div class="select-wrapper">
            <select v-model="form.province">
              <option value="">请选择省份</option>
              <option value="北京">北京</option>
              <option value="天津">天津</option>
              <option value="河北">河北</option>
              <option value="山西">山西</option>
              <option value="内蒙古">内蒙古</option>
              <option value="辽宁">辽宁</option>
              <option value="吉林">吉林</option>
              <option value="黑龙江">黑龙江</option>
              <option value="上海">上海</option>
              <option value="江苏">江苏</option>
              <option value="浙江">浙江</option>
              <option value="安徽">安徽</option>
              <option value="福建">福建</option>
              <option value="江西">江西</option>
              <option value="山东">山东</option>
              <option value="河南">河南</option>
              <option value="湖北">湖北</option>
              <option value="湖南">湖南</option>
              <option value="广东">广东</option>
              <option value="广西">广西</option>
              <option value="海南">海南</option>
              <option value="重庆">重庆</option>
              <option value="四川">四川</option>
              <option value="贵州">贵州</option>
              <option value="云南">云南</option>
              <option value="西藏">西藏</option>
              <option value="陕西">陕西</option>
              <option value="甘肃">甘肃</option>
              <option value="青海">青海</option>
              <option value="宁夏">宁夏</option>
              <option value="新疆">新疆</option>
            </select>
          </div>
        </div>

        <div class="card">
          <div class="q-num">Q3</div>
          <div class="q-text">你目前就读的学校类型是？</div>
          <div class="options">
            <label v-for="opt in schoolOpts" :key="opt.v"
              class="opt-label" :class="{ checked: form.school_type === opt.v }"
              @click="form.school_type = opt.v">
              <span class="opt-key" :class="{ active: form.school_type === opt.v }">{{ opt.k }}</span>
              {{ opt.label }}
            </label>
          </div>
        </div>

        <!-- 各科成绩 -->
        <div class="section-title">各科成绩评估</div>

        <div class="card" v-for="subj in coreSubjects" :key="subj.key">
          <div class="q-num">{{ subj.qn }}</div>
          <div class="q-text">你目前的【{{ subj.label }}】成绩处于哪个分数段？</div>
          <div class="options">
            <label v-for="opt in subj.opts" :key="opt.v"
              class="opt-label" :class="{ checked: form[subj.key] === opt.v }"
              @click="form[subj.key] = opt.v">
              <span class="opt-key" :class="{ active: form[subj.key] === opt.v }">{{ opt.k }}</span>
              <span>{{ opt.label }} <span class="q-sub">{{ opt.sub }}</span></span>
            </label>
          </div>
        </div>

        <!-- 选考科目 -->
        <div class="section-title">选考科目评估 <span class="q-sub">（不选该科请选E）</span></div>

        <div class="card" v-for="subj in electives" :key="subj.key">
          <div class="q-num">{{ subj.qn }}</div>
          <div class="q-text">你的【{{ subj.label }}】科目目前不赋分得分情况？</div>
          <div class="options">
            <label v-for="opt in subj.opts" :key="opt.v"
              class="opt-label" :class="{ checked: form[subj.key] === opt.v }"
              @click="selectElective(subj.key, opt.v)">
              <span class="opt-key" :class="{ active: form[subj.key] === opt.v }">{{ opt.k }}</span>
              <span>{{ opt.label }} <span class="q-sub" v-if="opt.sub">{{ opt.sub }}</span></span>
            </label>
          </div>
        </div>

        <!-- 学习情况 -->
        <div class="section-title">学习情况</div>

        <div class="card">
          <div class="q-num">Q13</div>
          <div class="q-text">你目前在学习上最大的困扰是什么？<span class="q-sub">（可多选）</span></div>
          <div class="options">
            <label v-for="opt in problemOpts" :key="opt.v"
              class="opt-label multi" :class="{ checked: form.problems.includes(opt.v) }"
              @click="toggleMulti('problems', opt.v)">
              <span class="opt-key" :class="{ active: form.problems.includes(opt.v) }">{{ opt.k }}</span>
              {{ opt.label }}
            </label>
          </div>
        </div>

        <div class="card">
          <div class="q-num">Q14</div>
          <div class="q-text">你平时获取学习资料/辅导的主要渠道是？<span class="q-sub">（可多选）</span></div>
          <div class="options">
            <label v-for="opt in channelOpts" :key="opt.v"
              class="opt-label multi" :class="{ checked: form.channels.includes(opt.v) }"
              @click="toggleMulti('channels', opt.v)">
              <span class="opt-key" :class="{ active: form.channels.includes(opt.v) }">{{ opt.k }}</span>
              {{ opt.label }}
            </label>
          </div>
        </div>

        <div class="card">
          <div class="q-num">Q15</div>
          <div class="q-text">为了备战高考，你愿意在哪些方面进行投入？<span class="q-sub">（可多选）</span></div>
          <div class="options">
            <label v-for="opt in investOpts" :key="opt.v"
              class="opt-label multi" :class="{ checked: form.invest.includes(opt.v) }"
              @click="toggleMulti('invest', opt.v)">
              <span class="opt-key" :class="{ active: form.invest.includes(opt.v) }">{{ opt.k }}</span>
              {{ opt.label }}
            </label>
          </div>
        </div>

        <div class="card">
          <div class="q-num">Q16</div>
          <div class="q-text">为了达成理想的高考目标，你计划投入的预算范围是？</div>
          <div class="options">
            <label v-for="opt in budgetOpts" :key="opt.v"
              class="opt-label" :class="{ checked: form.budget === opt.v }"
              @click="form.budget = opt.v">
              <span class="opt-key" :class="{ active: form.budget === opt.v }">{{ opt.k }}</span>
              <span>{{ opt.label }} <span class="q-sub">{{ opt.sub }}</span></span>
            </label>
          </div>
        </div>

        <!-- 提交 -->
        <div class="submit-wrap">
          <div class="error-msg" v-if="errorMsg">{{ errorMsg }}</div>
          <button class="submit-btn" :disabled="loading" @click="submitForm(showForceBtn)">
            {{ loading ? '提交中...' : showForceBtn ? '重新提交（覆盖原有数据）' : '提交问卷' }}
          </button>
          <p class="hint">提交后我们将在24小时内为你生成专属学习计划<br>并通过企业微信发送给你</p>
        </div>

      </div>
    </div>

    <!-- Toast 弹窗（选科约束提示） -->
    <div class="toast-overlay" v-if="toastMsg" @click="toastMsg = ''">
      <div class="toast-box" @click.stop>
        <div class="toast-icon">⚠️</div>
        <div class="toast-text">{{ toastMsg }}</div>
        <button class="toast-btn" @click="toastMsg = ''">知道了</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'

const API_URL = 'https://gaokao-backend.onrender.com'

const submitted       = ref(false)
const analyzing       = ref(false)
const loading         = ref(false)
const errorMsg        = ref('')
const showForceBtn    = ref(false)
const toastMsg        = ref('')
const aiStep          = ref(0)
const qrFallback      = ref(false)
const screenshotFile  = ref(null)
const screenshotPreview = ref(null)
const fileInput       = ref(null)

const form = ref({
  name: '', wx_name: '', phone: '', province: '', school_type: '',
  chinese: '', math: '', english: '',
  physics: 'E', chemistry: 'E', biology: 'E',
  politics: 'E', history: 'E', geography: 'E',
  problems: [], channels: [], invest: [], budget: '',
})

// 选项数据
const schoolOpts = [
  { k:'A', v:'A', label:'重点高中（省/市重点）' },
  { k:'B', v:'B', label:'普通高中' },
  { k:'C', v:'C', label:'私立/国际学校' },
  { k:'D', v:'D', label:'自学' },
]

const coreSubjects = [
  { key:'chinese', label:'语文', qn:'Q4', opts:[
    { k:'A', v:'A', label:'90分以下', sub:'基础薄弱，急需巩固' },
    { k:'B', v:'B', label:'90-105分', sub:'处于中游，有提升空间' },
    { k:'C', v:'C', label:'106-120分', sub:'成绩良好，寻求突破' },
    { k:'D', v:'D', label:'121分以上', sub:'高分段，希望保持' },
  ]},
  { key:'math', label:'数学', qn:'Q5', opts:[
    { k:'A', v:'A', label:'60分以下', sub:'听不懂，跟不上' },
    { k:'B', v:'B', label:'60-90分', sub:'基础不牢，波动较大' },
    { k:'C', v:'C', label:'91-120分', sub:'中档题能做，难题卡壳' },
    { k:'D', v:'D', label:'121分以上', sub:'思维敏捷，拔高困难' },
  ]},
  { key:'english', label:'英语', qn:'Q6', opts:[
    { k:'A', v:'A', label:'70分以下', sub:'词汇量匮乏，阅读困难' },
    { k:'B', v:'B', label:'70-100分', sub:'语法有漏洞，完形填空弱' },
    { k:'C', v:'C', label:'101-130分', sub:'读写能力尚可，追求精准' },
    { k:'D', v:'D', label:'131分以上', sub:'接近满分，保持语感' },
  ]},
]

const electiveBase = [
  { k:'A', v:'A', label:'50分以下' },
  { k:'B', v:'B', label:'50-70分' },
  { k:'C', v:'C', label:'71-90分' },
  { k:'D', v:'D', label:'91分以上' },
  { k:'E', v:'E', label:'不选该科' },
]

const electiveSubs = {
  physics:   ['概念模糊，公式记不住','基础薄弱，大题动笔难','模型掌握一般，压轴题有难度','逻辑严密，冲刺满分',''],
  chemistry: ['元素性质混淆，方程式不会写','知识点零散，缺乏体系','原理懂但不深，实验题丢分','反应机理清晰，细节把控',''],
  biology:   ['课本生疏，专业术语不会用','遗传计算弱，长句表达不规范','逻辑链条完整，综合分析弱','思维强，尽量不丢分',''],
  politics:  ['概念混淆，只会死记硬背','知识点有印象，答题无逻辑','能结合材料，但术语运用不准','时政敏感度高，答题模板熟练',''],
  history:   ['时空观念混乱，史实不清','线索模糊，选择题靠蒙','主干知识掌握，但缺乏深度分析','历史素养高，史论结合好',''],
  geography: ['读图困难，自然地理听不懂','原理一知半解，答题抓不住点','区域定位尚可，综合分析弱','综合思维强，答题规范',''],
}

const electives = ['physics','chemistry','biology','politics','history','geography'].map((key, i) => ({
  key,
  label: { physics:'物理', chemistry:'化学', biology:'生物', politics:'政治', history:'历史', geography:'地理' }[key],
  qn: `Q${7 + i}`,
  opts: electiveBase.map((o, j) => ({ ...o, sub: electiveSubs[key][j] })),
}))

const problemOpts = [
  { k:'A', v:'A', label:'基础知识薄弱，跟不上学校进度' },
  { k:'B', v:'B', label:'偏科严重，某一科拉分太多' },
  { k:'C', v:'C', label:'刷题效率低，做不完卷子' },
  { k:'D', v:'D', label:'缺乏解题技巧，懂了但拿不到分' },
  { k:'E', v:'E', label:'心态崩了，无法集中注意力' },
  { k:'F', v:'F', label:'睡眠不足，精力跟不上' },
  { k:'G', v:'G', label:'没有困扰，状态很好' },
]

const channelOpts = [
  { k:'A', v:'A', label:'学校老师课堂/补习' },
  { k:'B', v:'B', label:'校外培训机构（线下）' },
  { k:'C', v:'C', label:'网课平台（线上）' },
  { k:'D', v:'D', label:'购买教辅书/试卷自学' },
  { k:'E', v:'E', label:'学长学姐/同学推荐' },
  { k:'F', v:'F', label:'社交媒体（小红书、抖音、B站等）' },
  { k:'G', v:'G', label:'从来没了解过' },
]

const investOpts = [
  { k:'A', v:'A', label:'优质的网课/录播课' },
  { k:'B', v:'B', label:'一对一/小班辅导' },
  { k:'C', v:'C', label:'押题卷/内部资料' },
  { k:'D', v:'D', label:'营养品/补脑食品' },
  { k:'E', v:'E', label:'学习硬件（台灯、词典笔、降噪耳机等）' },
  { k:'F', v:'F', label:'志愿填报咨询服务' },
  { k:'G', v:'G', label:'不愿意课后花时间学习' },
]

const budgetOpts = [
  { k:'A', v:'A', label:'199元以下', sub:'高性价比的资料' },
  { k:'B', v:'B', label:'200-999元', sub:'系统的专题课程' },
  { k:'C', v:'C', label:'1000-3999元', sub:'带服务的陪跑营' },
  { k:'D', v:'D', label:'5000元以上', sub:'定制化服务/一对一咨询' },
]

function toggleMulti(field, val) {
  const arr = form.value[field]
  const idx = arr.indexOf(val)
  if (idx === -1) arr.push(val)
  else arr.splice(idx, 1)
}

function triggerUpload() { fileInput.value.click() }

function onFileChange(e) {
  const file = e.target.files[0]
  if (!file) return
  if (file.size > 5 * 1024 * 1024) {
    errorMsg.value = '图片大小不能超过 5MB'
    return
  }
  screenshotFile.value = file
  screenshotPreview.value = URL.createObjectURL(file)
}

function clearScreenshot() {
  screenshotFile.value = null
  screenshotPreview.value = null
  if (fileInput.value) fileInput.value.value = ''
}

const totalCount = 11
const answeredCount = computed(() => {
  let n = 0
  if (form.value.name.trim()) n++
  if (form.value.province.trim()) n++
  if (form.value.school_type) n++
  if (form.value.chinese) n++
  if (form.value.math) n++
  if (form.value.english) n++
  const elects = ['physics','chemistry','biology','politics','history','geography']
  if (elects.some(k => form.value[k] && form.value[k] !== 'E')) n++
  if (form.value.problems.length) n++
  if (form.value.channels.length) n++
  if (form.value.invest.length) n++
  if (form.value.budget) n++
  return n
})
const progressPct = computed(() => Math.round((answeredCount.value / totalCount) * 100))

const ELECTIVE_KEYS = ['physics', 'chemistry', 'biology', 'politics', 'history', 'geography']

function selectElective(key, value) {
  const next = { ...form.value, [key]: value }

  // 六选三规则：物理/化学/生物/政治/历史/地理中恰好选 3 门
  if (value !== 'E') {
    const count = ELECTIVE_KEYS.filter(k => next[k] !== 'E').length
    if (count > 3) {
      toastMsg.value = '六门选考科目中最多选择 3 门，请先取消其他科目'
      return
    }
  }

  form.value = next
  errorMsg.value = ''
}

async function submitForm(force = false) {
  errorMsg.value = ''

  const name = form.value.name.trim()
  if (!name) { errorMsg.value = '请填写姓名'; return }

  const phone = (form.value.phone || '').trim()
  if (!phone) { errorMsg.value = '请填写手机号'; return }
  if (!/^1[3-9]\d{9}$/.test(phone)) { errorMsg.value = '请填写正确的11位手机号'; return }

  const province = (form.value.province || '').trim()
  if (!province) { errorMsg.value = '请选择省份'; return }

  if (!form.value.chinese || !form.value.math || !form.value.english) {
    errorMsg.value = '请完成语数英成绩评估'; return
  }
  // 六选三校验：物理/化学/生物/政治/历史/地理中恰好选 3 门
  const electiveCount = ELECTIVE_KEYS.filter(k => form.value[k] && form.value[k] !== 'E').length
  if (electiveCount !== 3) {
    errorMsg.value = '请在物理、化学、生物、政治、历史、地理中选择恰好 3 门选考科目'
    return
  }

  loading.value = true
  try {
    let screenshotUrl = ''
    if (screenshotFile.value) {
      const fd = new FormData()
      fd.append('file', screenshotFile.value)
      const uploadRes = await axios.post(`${API_URL}/api/upload-screenshot`, fd)
      screenshotUrl = uploadRes.data.url
    }

    await axios.post(`${API_URL}/api/submit`, {
      ...form.value,
      problems: form.value.problems.join(','),
      channels: form.value.channels.join(','),
      invest:   form.value.invest.join(','),
      screenshot_url: screenshotUrl,
      force,
    })

    loading.value = false
    analyzing.value = true
    window.scrollTo(0, 0)

    setTimeout(() => { aiStep.value = 1 }, 600)
    setTimeout(() => { aiStep.value = 2 }, 1400)
    setTimeout(() => { aiStep.value = 3 }, 2200)
    setTimeout(() => {
      analyzing.value = false
      submitted.value = true
    }, 3500)

  } catch (e) {
    loading.value = false
    // 已存在：提示用户可以覆盖
    if (e.response?.status === 409) {
      errorMsg.value = '你已经提交过了！如果想修改答案，请点击下方按钮重新提交（将覆盖原有数据）'
      showForceBtn.value = true
      return
    }
    errorMsg.value = e.response?.data?.detail || '提交失败，请重试'
  }
}

function goBack() {
  submitted.value = false
  analyzing.value = false
  aiStep.value = 0
  showForceBtn.value = true
  clearScreenshot()
  window.scrollTo(0, 0)
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@400;600;700&family=Noto+Sans+SC:wght@300;400;500&display=swap');

/* ─── AI 动画页 ──────────────────────────────────────────── */
.ai-page {
  min-height: 100vh;
  background: linear-gradient(160deg, #0d0d1a 0%, #1a1a2e 60%, #0d0d1a 100%);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px 24px;
  text-align: center;
}

.ai-orb {
  position: relative;
  width: 120px;
  height: 120px;
  margin-bottom: 36px;
}

.orb-core {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 3;
}

.orb-icon {
  font-size: 48px;
  animation: pulse 2s ease-in-out infinite;
}

.orb-ring {
  position: absolute;
  border-radius: 50%;
  border: 1.5px solid rgba(192, 57, 43, 0.5);
  animation: ripple 2.4s ease-out infinite;
}
.r1 { inset: 0; animation-delay: 0s; }
.r2 { inset: -16px; animation-delay: 0.6s; border-color: rgba(192,57,43,0.3); }
.r3 { inset: -32px; animation-delay: 1.2s; border-color: rgba(192,57,43,0.15); }

@keyframes ripple {
  0%   { transform: scale(0.85); opacity: 1; }
  100% { transform: scale(1.15); opacity: 0; }
}
@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50%       { transform: scale(1.1); }
}

.ai-label {
  font-size: 14px;
  color: rgba(255,255,255,0.5);
  letter-spacing: 3px;
  margin-bottom: 8px;
}

.ai-title {
  font-family: 'Noto Serif SC', serif;
  font-size: 26px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 2px;
  margin-bottom: 28px;
}

.ai-dots {
  display: flex;
  gap: 8px;
  margin-bottom: 40px;
}
.ai-dots span {
  width: 7px; height: 7px;
  border-radius: 50%;
  background: #c0392b;
  animation: bounce 1.2s ease-in-out infinite;
}
.ai-dots span:nth-child(2) { animation-delay: 0.2s; }
.ai-dots span:nth-child(3) { animation-delay: 0.4s; }

@keyframes bounce {
  0%, 80%, 100% { transform: translateY(0); opacity: 0.4; }
  40%           { transform: translateY(-10px); opacity: 1; }
}

.ai-steps {
  display: flex;
  flex-direction: column;
  gap: 14px;
  text-align: left;
}
.ai-step {
  font-size: 13px;
  color: rgba(255,255,255,0.25);
  letter-spacing: 1px;
  transition: color 0.5s ease;
  padding-left: 4px;
}
.ai-step.done {
  color: rgba(255,255,255,0.85);
}

/* ─── 成功页 ─────────────────────────────────────────────── */
.success-page {
  min-height: 100vh;
  background: linear-gradient(160deg, #0d0d1a 0%, #1a1a2e 100%);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 24px;
  text-align: center;
}

.success-badge {
  display: inline-block;
  background: rgba(192,57,43,0.15);
  border: 1px solid rgba(192,57,43,0.4);
  color: #e74c3c;
  font-size: 11px;
  letter-spacing: 3px;
  padding: 5px 16px;
  border-radius: 20px;
  margin-bottom: 20px;
}

.success-icon { font-size: 56px; margin-bottom: 16px; }

.success-page h2 {
  font-family: 'Noto Serif SC', serif;
  font-size: 24px;
  font-weight: 700;
  color: #fff;
  margin-bottom: 12px;
}

.success-desc {
  font-size: 14px;
  color: rgba(255,255,255,0.5);
  line-height: 1.9;
  margin-bottom: 20px;
}

.fix-btn {
  display: inline-block;
  margin-bottom: 24px;
  background: none;
  border: 1px solid rgba(255,255,255,0.15);
  color: rgba(255,255,255,0.4);
  font-size: 12px;
  padding: 8px 20px;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.2s;
}
.fix-btn:hover { border-color: rgba(255,255,255,0.35); color: rgba(255,255,255,0.7); }

/* 二维码卡片 */
.qr-card {
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 16px;
  padding: 28px 32px;
  width: 100%;
  max-width: 280px;
}

.qr-label {
  font-size: 12px;
  letter-spacing: 2px;
  color: rgba(255,255,255,0.4);
  margin-bottom: 18px;
}

.qr-wrap a { display: block; width: 100%; height: 100%; cursor: pointer; }
.qr-wrap {
  width: 180px;
  height: 180px;
  margin: 0 auto 16px;
  border-radius: 10px;
  overflow: hidden;
  background: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
}

.qr-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.qr-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 16px;
}
.qr-placeholder-icon { font-size: 36px; }
.qr-placeholder-text {
  font-size: 11px;
  color: #999;
  line-height: 1.6;
  text-align: center;
}

.qr-hint {
  font-size: 11px;
  color: rgba(255,255,255,0.25);
  line-height: 1.7;
}

/* ─── 问卷样式（不变）────────────────────────────────────── */
.banner {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  color: #fff;
  text-align: center;
  padding: 36px 20px 28px;
}
.banner-tag {
  display: inline-block;
  background: #c0392b;
  color: #fff;
  font-size: 11px;
  letter-spacing: 2px;
  padding: 4px 12px;
  border-radius: 2px;
  margin-bottom: 14px;
}
.banner h1 {
  font-family: 'Noto Serif SC', serif;
  font-size: 26px;
  font-weight: 700;
  margin-bottom: 10px;
}
.banner p { font-size: 13px; color: rgba(255,255,255,0.65); line-height: 1.7; }

.progress-wrap {
  background: #f5ead0;
  padding: 14px 20px 10px;
  border-bottom: 1px solid #d4c5a0;
  position: sticky; top: 0; z-index: 100;
}
.progress-label {
  font-size: 12px; color: #7f8c8d;
  margin-bottom: 6px;
  display: flex; justify-content: space-between;
}
.progress-bar { height: 4px; background: #d4c5a0; border-radius: 2px; overflow: hidden; }
.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #c0392b, #f39c12);
  border-radius: 2px;
  transition: width 0.4s ease;
}

.container { max-width: 680px; margin: 0 auto; padding: 0 16px; }

.section-title {
  font-family: 'Noto Serif SC', serif;
  font-size: 13px; font-weight: 600;
  color: #c0392b; letter-spacing: 2px;
  margin: 28px 0 12px;
  display: flex; align-items: center; gap: 8px;
}
.section-title::after { content: ''; flex: 1; height: 1px; background: #d4c5a0; }

.card {
  background: #fff;
  border: 1px solid #d4c5a0;
  border-radius: 8px;
  padding: 22px 20px;
  margin-bottom: 14px;
  box-shadow: 0 4px 24px rgba(26,26,46,0.08);
}
.q-num { font-size: 11px; font-weight: 500; color: #c0392b; letter-spacing: 1px; margin-bottom: 8px; }
.q-text {
  font-family: 'Noto Serif SC', serif;
  font-size: 16px; font-weight: 600;
  line-height: 1.6; margin-bottom: 18px;
}
.q-sub { font-family: 'Noto Sans SC', sans-serif; font-size: 12px; color: #7f8c8d; font-weight: 300; }

.options { display: flex; flex-direction: column; gap: 10px; }
.opt-label {
  display: flex; align-items: flex-start; gap: 12px;
  padding: 12px 14px;
  border: 1.5px solid #d4c5a0;
  border-radius: 6px; cursor: pointer;
  transition: all 0.2s;
  background: #fdf6e3;
  font-size: 14px; line-height: 1.5;
  user-select: none;
}
.opt-label:hover { border-color: #e74c3c; background: #fff5f5; }
.opt-label.checked { border-color: #c0392b; background: #fff0ef; }
.opt-label.multi.checked { border-color: #1a1a2e; background: #f0f4ff; }
.opt-label.multi:hover { border-color: #1a1a2e; background: #f5f7ff; }

.opt-key {
  flex-shrink: 0; width: 24px; height: 24px;
  border-radius: 50%; border: 1.5px solid #d4c5a0;
  display: flex; align-items: center; justify-content: center;
  font-size: 11px; font-weight: 700; color: #7f8c8d;
  background: #fff; transition: all 0.2s; margin-top: 1px;
}
.opt-key.active { background: #c0392b; border-color: #c0392b; color: #fff; }
.opt-label.multi .opt-key.active { background: #1a1a2e; border-color: #1a1a2e; }

.text-input {
  width: 100%; border: 1.5px solid #d4c5a0;
  border-radius: 6px; padding: 12px 14px;
  font-family: 'Noto Sans SC', sans-serif;
  font-size: 14px; background: #fdf6e3;
  color: #1a1a2e; outline: none; transition: border-color 0.2s;
  box-sizing: border-box;
}
.text-input:focus { border-color: #c0392b; background: #fff; }
.text-input::placeholder { color: #7f8c8d; }

.select-wrapper {
  position: relative;
}
.select-wrapper select {
  width: 100%;
  padding: 12px 14px;
  border: 1.5px solid #d4c5a0;
  border-radius: 6px;
  font-family: 'Noto Sans SC', sans-serif;
  font-size: 14px;
  background: #fdf6e3;
  color: #1a1a2e;
  outline: none;
  appearance: none;
  cursor: pointer;
  transition: border-color 0.2s;
}
.select-wrapper select:focus {
  border-color: #c0392b;
  background: #fff;
}

.input-with-tip { position: relative; display: flex; align-items: center; gap: 6px; }
.input-with-tip .text-input { flex: 1; }
.tip-icon {
  width: 18px; height: 18px;
  border-radius: 50%;
  background: #d4c5a0;
  color: #1a1a2e;
  font-size: 11px; font-weight: 700;
  display: flex; align-items: center; justify-content: center;
  cursor: help;
  flex-shrink: 0;
}

.upload-area {
  border: 2px dashed #d4c5a0;
  border-radius: 8px;
  padding: 24px;
  text-align: center;
  cursor: pointer;
  transition: all 0.2s;
  background: #fdf6e3;
}
.upload-area:hover { border-color: #c0392b; background: #fff5f5; }
.upload-area.has { padding: 8px; }

.upload-placeholder { display: flex; flex-direction: column; align-items: center; gap: 8px; color: #7f8c8d; font-size: 14px; }
.upload-icon { font-size: 32px; }
.upload-hint { font-size: 11px; color: #b0a080; }

.preview-img { max-width: 100%; max-height: 200px; border-radius: 6px; object-fit: contain; }

.upload-actions { text-align: center; margin-top: 8px; }
.retry-btn {
  background: none; border: 1px solid #d4c5a0;
  color: #7f8c8d; font-size: 12px;
  padding: 4px 16px; border-radius: 16px;
  cursor: pointer; transition: all 0.2s;
}
.retry-btn:hover { border-color: #c0392b; color: #c0392b; }

.row-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
@media (max-width: 480px) { .row-2 { grid-template-columns: 1fr; } }

.submit-wrap { padding: 28px 0 40px; text-align: center; }
.error-msg {
  background: #fff0ef; border: 1px solid #c0392b;
  border-radius: 6px; padding: 12px 16px;
  font-size: 13px; color: #c0392b; margin-bottom: 16px;
}
.submit-btn {
  background: linear-gradient(135deg, #c0392b 0%, #a93226 100%);
  color: #fff; border: none; border-radius: 8px;
  padding: 16px 48px;
  font-family: 'Noto Serif SC', serif;
  font-size: 17px; font-weight: 600; letter-spacing: 2px;
  cursor: pointer;
  box-shadow: 0 6px 20px rgba(192,57,43,0.35);
  transition: all 0.2s; width: 100%; max-width: 360px;
}
.submit-btn:hover { transform: translateY(-2px); box-shadow: 0 8px 28px rgba(192,57,43,0.45); }
.submit-btn:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }
.hint { font-size: 12px; color: #7f8c8d; margin-top: 12px; line-height: 1.6; }

/* ─── Toast 弹窗 ──────────────────────────────────────────── */
.toast-overlay {
  position: fixed; inset: 0;
  background: rgba(0,0,0,0.5);
  display: flex; align-items: center; justify-content: center;
  z-index: 9999;
  animation: fadeIn 0.2s ease;
}
.toast-box {
  background: #fff; border-radius: 16px;
  padding: 32px 28px 24px;
  text-align: center; max-width: 320px; width: 90%;
  box-shadow: 0 16px 48px rgba(0,0,0,0.25);
  animation: scaleIn 0.25s ease;
}
.toast-icon { font-size: 40px; margin-bottom: 12px; }
.toast-text { font-size: 15px; color: #2d3436; line-height: 1.6; margin-bottom: 20px; }
.toast-btn {
  background: #c0392b; color: #fff; border: none;
  border-radius: 8px; padding: 10px 36px;
  font-size: 14px; font-weight: 500; cursor: pointer;
}
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
@keyframes scaleIn { from { transform: scale(0.85); opacity: 0; } to { transform: scale(1); opacity: 1; } }
</style>