<template>
  <div class="job-details-container">
    <Transition name="fade-slide" appear>
      <div v-if="job" class="job-content">
        <div class="job-header">
          <h1>{{ job.title }}</h1>
          <div class="job-meta">
            <span class="category">{{ getCategoryName(job.category_id) }}</span>
            <span class="location">{{ job.location }}</span>
            <span class="experience">{{ job.experience_level }}</span>
            <span class="work-type">{{ job.work_type }}</span>
            <span class="salary">{{ formatSalary(job.salary) }}</span>
          </div>
        </div>

        <div class="job-body">
          <div class="job-section">
            <h2>Job Description</h2>
            <div class="text-content">{{ job.description }}</div>
          </div>

          <div class="job-section">
            <h2>Responsibilities</h2>
            <div class="text-content">{{ job.responsibilities }}</div>
          </div>

          <div class="job-section">
            <h2>Required Skills</h2>
            <div class="skills-list">
              <span v-for="(skill, index) in formattedSkills" :key="index" class="skill-tag">
                {{ skill }}
              </span>
            </div>
          </div>

          <div class="job-section">
            <h2>Technologies</h2>
            <div class="technologies-list">
              <span v-for="(tech, index) in formattedTechnologies" :key="index" class="tech-tag">
                {{ tech }}
              </span>
            </div>
          </div>
        </div>

        <div class="job-footer">
          <button @click="handleApplyClick" class="apply-button">
            Apply for this position
          </button>
          <p class="deadline">Application deadline: {{ formatDate(job.deadline) }}</p>
        </div>
      </div>
    </Transition>

    <div v-if="isLoading" class="loading-state">
      <p>Loading job details...</p>
    </div>

    <div v-if="error" class="error-state">
      <p>{{ error }}</p>
      <button @click="fetchJob">Try Again</button>
    </div>

    <Transition name="fade" appear>
      <div v-if="showForm" class="modal-overlay" @click.self="closeForm">
        <div class="application-modal">
          <button class="close-button" @click="closeForm">&times;</button>
          
          <JobComments />
          <h2 class="mb-4">Apply for {{ job.title }}</h2>
          
          <form 
            @submit.prevent="submitApplication" 
            class="application-form needs-validation" 
            novalidate
            ref="applicationForm"
          >
            <div class="mb-3">
              <label for="name" class="form-label">Full Name</label>
              <input 
                type="text" 
                class="form-control" 
                id="name" 
                v-model="application.name" 
                required
                placeholder="Your full name"
                :class="{ 'is-invalid': errors.name }"
              >
              <div class="invalid-feedback">
                {{ errors.name || 'Please provide your full name.' }}
              </div>
            </div>

            <div class="mb-3">
              <label for="email" class="form-label">Email</label>
              <input 
                type="email" 
                class="form-control" 
                id="email" 
                v-model="application.email" 
                required
                placeholder="Your email address"
                :class="{ 'is-invalid': errors.email }"
              >
              <div class="invalid-feedback">
                {{ errors.email || 'Please provide a valid email.' }}
              </div>
            </div>

            <div class="mb-3">
              <label for="phone" class="form-label">Phone Number</label>
              <input 
                type="tel" 
                class="form-control" 
                id="phone" 
                v-model="application.phone" 
                required
                placeholder="Your phone number"
                pattern="[0-9]{10,15}"
                :class="{ 'is-invalid': errors.phone }"
              >
              <div class="invalid-feedback">
                {{ errors.phone || 'Please provide a valid phone number (10-15 digits).' }}
              </div>
            </div>

            <div class="mb-3">
              <label for="resume" class="form-label">Resume (PDF/DOC)</label>
              <input 
                type="file" 
                class="form-control" 
                id="resume" 
                @change="handleFileUpload" 
                accept=".pdf,.doc,.docx" 
                required
                :class="{ 'is-invalid': errors.resume }"
              >
              <div class="invalid-feedback">
                {{ errors.resume || 'Please upload your resume.' }}
              </div>
            </div>

            <div class="mb-3">
              <label for="cover-letter" class="form-label">Cover Letter</label>
              <textarea 
                class="form-control" 
                id="cover-letter" 
                v-model="application.coverLetter" 
                rows="5" 
                placeholder="Why are you a good fit for this position?"
                :class="{ 'is-invalid': errors.coverLetter }"
              ></textarea>
              <div class="invalid-feedback">
                {{ errors.coverLetter || 'Please provide a cover letter.' }}
              </div>
            </div>

            <button 
              type="submit" 
              class="btn btn-primary w-100"
              :disabled="isSubmitting"
            >
              {{ isSubmitting ? 'Submitting...' : 'Submit Application' }}
            </button>

            <div 
              v-if="submitMessage" 
              class="alert mt-3" 
              :class="submitSuccess ? 'alert-success' : 'alert-danger'"
            >
              {{ submitMessage }}
            </div>
          </form>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import axios from 'axios'
import dayjs from 'dayjs'

const route = useRoute()
const router = useRouter()

const job = ref(null)
const categories = ref([])
const isLoading = ref(true)
const error = ref(null)
const showForm = ref(false)
const isSubmitting = ref(false)
const submitMessage = ref('')
const submitSuccess = ref(false)
const applicationForm = ref(null)
const errors = ref({
  name: '',
  email: '',
  phone: '',
  resume: '',
  coverLetter: ''
})

const application = ref({
  name: '',
  email: '',
  phone: '',
  resume: null,
  coverLetter: ''
})

const fetchJob = async () => {
  isLoading.value = true
  error.value = null
  try {
    const [jobRes, categoriesRes] = await Promise.all([
      axios.get(`http://localhost:8000/api/jobs/${route.params.id}`),
      axios.get('http://localhost:8000/api/categories')
    ])
    
    job.value = jobRes.data.data
    categories.value = categoriesRes.data.data
  
    if (new Date(job.value.deadline) < new Date()) {
      error.value = 'This job is no longer accepting applications.'
    }
  } catch (err) {
    console.error('Error fetching job details:', err)
    error.value = 'Failed to load job details. Please try again later.'
  } finally {
    isLoading.value = false
  }
}

const formattedSkills = computed(() => {
  if (!job.value?.skills) return []
  return job.value.skills.split(',').map(skill => skill.trim())
})

const formattedTechnologies = computed(() => {
  if (!job.value?.technologies) return []
  return job.value.technologies.split(',').map(tech => tech.trim())
})

const getCategoryName = (categoryId) => {
  const category = categories.value.find(cat => cat.id === categoryId)
  return category ? category.name : 'Uncategorized'
}

const formatSalary = (salary) => {
  if (!salary) return 'Salary not specified'
  return `$${salary.toLocaleString()}`
}

const formatDate = (dateString) => {
  return dayjs(dateString).format('MMMM D, YYYY')
}

const handleFileUpload = (event) => {
  const file = event.target.files[0]
  if (file) {
    const validTypes = ['application/pdf', 'application/msword', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document']
    const maxSize = 5 * 1024 * 1024 // 5MB
    
    if (!validTypes.includes(file.type)) {
      errors.value.resume = 'Please upload a PDF or DOC file.'
      application.value.resume = null
      return
    }
    
    if (file.size > maxSize) {
      errors.value.resume = 'File size should not exceed 5MB.'
      application.value.resume = null
      return
    }
    
    errors.value.resume = ''
    application.value.resume = file
  }
}

const validateForm = () => {
  let isValid = true
  errors.value = {
    name: '',
    email: '',
    phone: '',
    resume: '',
    coverLetter: ''
  }

  // Validate name
  if (!application.value.name.trim()) {
    errors.value.name = 'Full name is required.'
    isValid = false
  }

  // Validate email
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!application.value.email) {
    errors.value.email = 'Email is required.'
    isValid = false
  } else if (!emailRegex.test(application.value.email)) {
    errors.value.email = 'Please enter a valid email address.'
    isValid = false
  }

  // Validate phone
  const phoneRegex = /^[0-9]{10,15}$/
  if (!application.value.phone) {
    errors.value.phone = 'Phone number is required.'
    isValid = false
  } else if (!phoneRegex.test(application.value.phone)) {
    errors.value.phone = 'Please enter a valid phone number (10-15 digits).'
    isValid = false
  }

  // Validate resume
  if (!application.value.resume) {
    errors.value.resume = 'Resume is required.'
    isValid = false
  }

  // Validate cover letter (مطلوب)
  if (!application.value.coverLetter.trim()) {
    errors.value.coverLetter = 'Cover letter is required.'
    isValid = false
  }

  return isValid
}

const handleApplyClick = async () => {
  if (new Date(job.value.deadline) < new Date()) {
    submitMessage.value = 'This job is no longer accepting applications.'
    return
  }

  const token = localStorage.getItem('token')

  if (!token) {
    router.push('/login')
  } else {
    showForm.value = true
    const userData = JSON.parse(localStorage.getItem('userData') || '{}')
    application.value.name = userData.name || ''
    application.value.email = userData.email || ''
    application.value.phone = userData.phone || ''

    await nextTick()
    if (applicationForm.value) {
      applicationForm.value.classList.remove('was-validated')
    }
  }
}

const closeForm = () => {
  showForm.value = false
  submitMessage.value = ''
  errors.value = {
    name: '',
    email: '',
    phone: '',
    resume: '',
    coverLetter: ''
  }
  application.value = {
    name: '',
    email: '',
    phone: '',
    resume: null,
    coverLetter: ''
  }
  // تنظيف قيمة ملف الرفع input عند إغلاق الفورم
  const resumeInput = document.getElementById('resume')
  if (resumeInput) resumeInput.value = ''
}

const submitApplication = async () => {
  // Validate form
  if (!validateForm()) {
    await nextTick()
    if (applicationForm.value) {
      applicationForm.value.classList.add('was-validated')
    }
    return
  }

  isSubmitting.value = true
  submitMessage.value = ''

  const token = localStorage.getItem('token')
  const jobId = route.params.id

  try {
    const formData = new FormData()
    formData.append('resume_snapshot', application.value.resume)
    formData.append('cover_letter', application.value.coverLetter || '')
    formData.append('contact_email', application.value.email)
    formData.append('contact_phone', application.value.phone)
    formData.append('name', application.value.name)

    // Debug: طباعة محتويات formData
    for (const pair of formData.entries()) {
      console.log(pair[0] + ':', pair[1])
    }

    await axios.post(
      `http://localhost:8000/api/jobs/${jobId}/apply`,
      formData,
      {
        headers: {
          'Content-Type': 'multipart/form-data',
          Authorization: `Bearer ${token}`
        }
      }
    )

    submitSuccess.value = true
    submitMessage.value = 'Application submitted successfully!'

    setTimeout(() => {
      application.value = {
        name: '',
        email: '',
        phone: '',
        resume: null,
        coverLetter: ''
      }
      showForm.value = false
      submitMessage.value = ''
      if (applicationForm.value) {
        applicationForm.value.classList.remove('was-validated')
      }
      // تنظيف قيمة ملف الرفع بعد الإرسال
      const resumeInput = document.getElementById('resume')
      if (resumeInput) resumeInput.value = ''
    }, 3000)
  } catch (error) {
    // طباعة تفاصيل الخطأ كاملة
    console.error('Full error response:', error.response)
    console.error('Error data:', error.response?.data)

    if (error.response?.status === 422) {
      // Handle validation errors from server
      const serverErrors = error.response.data.errors
      if (serverErrors) {
        for (const field in serverErrors) {
          if (errors.value.hasOwnProperty(field)) {
            errors.value[field] = serverErrors[field][0]
          }
        }
      }
      submitMessage.value = 'Please check your inputs and try again.'
    } else if (error.response?.status === 409) {
      submitMessage.value =
        error.response.data.message ||
        'You may have already applied for this position or the job is no longer accepting applications.'
    } else {
      submitMessage.value = 'Failed to submit application. Please try again.'
    }

    submitSuccess.value = false
  } finally {
    isSubmitting.value = false
  }
}


// تحميل بيانات الوظيفة عند تهيئة المكون
onMounted(() => {
  fetchJob()
})

</script>

<style scoped>
.job-details-container {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.job-content {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 2rem;
  word-wrap: break-word;
  overflow-wrap: break-word;
}

.job-header h1 {
  color: #047fec;
  margin-bottom: 1rem;
  word-break: break-word;
}

.job-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-bottom: 2rem;
  color: #666;
}

.job-meta span {
  padding-right: 1rem;
  border-right: 1px solid #ddd;
  word-break: break-word;
}

.job-meta span:last-child {
  border-right: none;
}

.job-section {
  margin-bottom: 2rem;
}

.job-section h2 {
  color: #333;
  margin-bottom: 0.5rem;
  font-size: 1.25rem;
}

.text-content {
  white-space: pre-wrap;
  word-break: break-word;
  line-height: 1.6;
}

.skills-list, .technologies-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.skill-tag, .tech-tag {
  background: #f0f7ff;
  color: #047fec;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.9rem;
  word-break: break-word;
}

.job-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 2rem;
  padding-top: 2rem;
  border-top: 1px solid #eee;
  flex-wrap: wrap;
  gap: 1rem;
}

.apply-button {
  background: #00e56c;
  color: white;
  padding: 0.75rem 2rem;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.3s;
  flex-shrink: 0;
}

.apply-button:hover {
  background: #00c45a;
}

.deadline {
  color: #666;
  word-break: break-word;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.application-modal {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  width: 100%;
  max-width: 600px;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
}

.close-button {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #666;
}

.application-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-group label {
  font-weight: 600;
  color: #333;
}

.form-group input,
.form-group textarea,
.form-group select {
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  width: 100%;
  box-sizing: border-box;
  word-break: break-word;
}

.form-group textarea {
  min-height: 150px;
  resize: vertical;
  white-space: pre-wrap;
}

.submit-button {
  background: #047fec;
  color: white;
  padding: 0.75rem;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.3s;
  margin-top: 1rem;
}

.submit-button:hover:not(:disabled) {
  background: #0366c4;
}

.submit-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.message {
  padding: 1rem;
  border-radius: 4px;
  margin-top: 1rem;
  text-align: center;
  word-break: break-word;
}

.message.success {
  background: #e6f7e6;
  color: #00a854;
}

.message.error {
  background: #fff1f0;
  color: #f5222d;
}

.loading-state, .error-state {
  text-align: center;
  padding: 2rem;
}

.error-state button {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  background: #047fec;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* Transition effects */
.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.3s ease;
}

.fade-slide-enter-from,
.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(20px);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

@media (max-width: 768px) {
  .job-content {
    padding: 1rem;
  }
  
  .job-meta {
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .job-meta span {
    border-right: none;
    padding-right: 0;
  }
  
  .job-footer {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .application-modal {
    width: 95%;
    padding: 1rem;
  }
}
</style>