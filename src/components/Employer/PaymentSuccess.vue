<template>
  <div class="payment-success-container">
    <div class="payment-success-card">
      <div class="icon-container">
        <svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="#28a745" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path>
          <polyline points="22 4 12 14.01 9 11.01"></polyline>
        </svg>
      </div>
      <h1 class="success-title">Payment Successful!</h1>
      <p class="success-message">{{ message }}</p>
      
      <div class="action-buttons">
        <router-link to="/employer" class="btn btn-primary">
          Go to Dashboard
        </router-link>
        <router-link to="/jobs" class="btn btn-secondary">
          Browse More Jobs
        </router-link>
      </div>
      
      <div class="additional-info">
        <p>Need help? <a href="/contact">Contact our support team</a></p>
        <p>A receipt has been sent to your email</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';
import axios from 'axios';

const message = ref('Processing your payment...');

onMounted(async () => {
  try {
    // Get URL parameters
    const urlParams = new URLSearchParams(window.location.search);
    const jobId = urlParams.get('job_id');
    const applicationId = urlParams.get('application_id');

    if (!jobId || !applicationId) {
      message.value = 'Missing required parameters. Please contact support.';
      return;
    }

    // Update job status to closed
    await axios.put(`http://localhost:8000/api/jobs/${jobId}`, {
      status: 'closed'
    }, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`,
        'Content-Type': 'application/json',
        Accept: 'application/json',
      },
    });

    // Update application status to accepted
    await axios.put(`http://localhost:8000/api/applications/${applicationId}/status`, {
      status: 'accepted'
    }, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`,
        'Content-Type': 'application/json',
        Accept: 'application/json',
      },
    });

    message.value = 'Your payment has been processed successfully. The job application has been accepted and the job posting has been closed.';
  } catch (error) {
    console.error('Error updating status:', error);
    message.value = 'Payment was successful, but we failed to update the job status. Please contact support.';
  }
});
</script>

<style scoped>
.payment-success-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f8f9fa;
  padding: 2rem;
}

.payment-success-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 3rem;
  max-width: 600px;
  width: 100%;
  text-align: center;
  transition: transform 0.3s ease;
}

.payment-success-card:hover {
  transform: translateY(-5px);
}

.icon-container {
  margin-bottom: 1.5rem;
}

.icon-container svg {
  animation: checkmark 0.6s ease;
}

.success-title {
  color: #28a745;
  font-size: 2rem;
  font-weight: 700;
  margin-bottom: 1.5rem;
}

.success-message {
  color: #495057;
  font-size: 1.1rem;
  line-height: 1.6;
  margin-bottom: 2rem;
}

.action-buttons {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 2rem;
}

.btn {
  display: inline-block;
  padding: 0.8rem 1.5rem;
  border-radius: 8px;
  font-weight: 600;
  text-decoration: none;
  transition: all 0.3s ease;
}

.btn-primary {
  background-color: #28a745;
  color: white;
  border: 2px solid #28a745;
}

.btn-primary:hover {
  background-color: #218838;
  border-color: #218838;
  transform: translateY(-2px);
}

.btn-secondary {
  background-color: white;
  color: #28a745;
  border: 2px solid #28a745;
}

.btn-secondary:hover {
  background-color: #f8f9fa;
  transform: translateY(-2px);
}

.additional-info {
  color: #6c757d;
  font-size: 0.9rem;
  line-height: 1.6;
}

.additional-info a {
  color: #007bff;
  text-decoration: none;
}

.additional-info a:hover {
  text-decoration: underline;
}

@keyframes checkmark {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@media (max-width: 768px) {
  .payment-success-card {
    padding: 2rem;
  }
  
  .action-buttons {
    flex-direction: column;
  }
}
</style>