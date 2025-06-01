<template>
    <div class="d-flex justify-content-center">
      <button
        class="btn btn-primary"
        :disabled="loading"
        @click="handlePayment"
      >
        <span v-if="loading" class="spinner-border spinner-border-sm me-2" role="status" aria-hidden="true"></span>
        Accept
      </button>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue'
  import axios from 'axios'
  
  const props = defineProps({
    jobId: {
      type: Number,
      required: true
    },
    amount: {
      type: Number,
      required: true
      
    }
  })
  
  const loading = ref(false)
  
  const handlePayment = async () => {
    loading.value = true
  
    try {
      const { data } = await axios.post(
        `http://localhost:8000/api/create-checkout-session/${props.jobId}`,
        { amount: props.amount },
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`
          }
        }
      )
  
      const stripe = await loadStripe(import.meta.env.VITE_STRIPE_KEY)
      stripe.redirectToCheckout({ sessionId: data.id })
    } catch (error) {
      console.error('Stripe checkout error:', error)
      alert('Failed to initiate payment.')
    } finally {
      loading.value = false
    }
  }
  
  const loadStripe = async (publicKey) => {
    if (!window.Stripe) {
      await loadStripeScript()
    }
    return window.Stripe(publicKey)
  }
  
  const loadStripeScript = () => {
    return new Promise((resolve, reject) => {
      if (document.querySelector('#stripe-js')) return resolve()
  
      const script = document.createElement('script')
      script.id = 'stripe-js'
      script.src = 'https://js.stripe.com/v3/'
      script.onload = resolve
      script.onerror = reject
      document.head.appendChild(script)
    })
  }
  </script>
  