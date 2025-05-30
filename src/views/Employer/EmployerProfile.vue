<template>
  <div class="bg-light min-vh-100 py-5">
    <div class="container">
      <div class="card shadow-lg p-4">
        <div class="d-flex align-items-center mb-4">
          <div class="me-4">
            <img
              :src="employer.company_logo || defaultLogo"
              @error="onLogoError"
              alt="Company Logo"
              class="rounded-circle border"
              style="width: 100px; height: 100px; object-fit: cover"
            />
          </div>
          <div>
            <h3 class="mb-1">{{ employer.company_name }}</h3>
            <p class="text-muted mb-0">{{ employer.user.email }}</p>
          </div>
        </div>

        <div class="row">
          <div class="col-md-6 mb-3">
            <h6 class="text-muted">Bio</h6>
            <p class="fw-medium">{{ employer.bio || 'N/A' }}</p>
          </div>
          <div class="col-md-6 mb-3">
            <h6 class="text-muted">Website</h6>
            <a
              :href="employer.company_website"
              class="text-primary text-decoration-none"
              target="_blank"
            >
              {{ employer.company_website || 'N/A' }}
            </a>
          </div>
          <div class="col-md-6 mb-3">
            <h6 class="text-muted">Phone</h6>
            <p>{{ employer.user.phone }}</p>
              
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      employer: {
        company_name: '',
        company_website: '',
        company_logo: '',
        bio: '',
        user_id: '',
        created_at: '',
        updated_at: '',
        user: {},
      },
      defaultLogo: 'https://via.placeholder.com/150?text=No+Logo',
      currUser: JSON.parse(localStorage.getItem('user')),
    };
  },
  mounted() {
    if (!this.currUser || this.currUser.role !== 'employer') {
      this.$router.push('/login');
    } else {
      this.fetchProfile();
    }
  },
  methods: {
    fetchProfile() {
      axios
        .get('http://localhost:8000/api/employer-profile', {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then((response) => {
          this.employer = response.data.data;
        })
        .catch((error) => {
          console.error('Error fetching profile:', error);
        });
    },
    onLogoError(event) {
      event.target.src = this.defaultLogo;
    },
  },
};
</script>

<style scoped>
img {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

img:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
</style>