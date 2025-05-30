<template>
  <div>
    <aside class="shadow-bg bg-white rounded-lg p-4 mb-4 dashboard">
      <h5 class="text-center mb-3">Welcome, {{ currUser.name }}</h5>
      <img :src="currUser.profile_picture" alt="Profile Picture" class="rounded-circle mx-auto d-block" @error="onLogoError" />
      <p class="text-center mb-4 text-muted">{{ currUser.email }}</p>

      <div class="list-group">
        <router-link class="list-group-item list-group-item-action d-flex align-items-center gap-2" :to="{ name: 'EmployerHome' }">
          <i class="fas fa-house"></i> Home
        </router-link>
        <router-link class="list-group-item list-group-item-action d-flex align-items-center gap-2" :to="{ name: 'JobForm' }">
          <i class="fas fa-pen-to-square"></i> Post a Job
        </router-link>
        <router-link class="list-group-item list-group-item-action d-flex align-items-center gap-2" :to="{ name: 'JobList' }">
          <i class="fas fa-briefcase"></i> Job Listings
        </router-link>
        <router-link class="list-group-item list-group-item-action d-flex align-items-center gap-2" :to="{ name: 'EmployerApplications' }">
          <i class="fas fa-file-alt"></i> Applications
        </router-link>
        <router-link class="list-group-item list-group-item-action d-flex align-items-center gap-2" :to="{ name: 'EmployerProfile' }">
          <i class="fas fa-user"></i> My Profile
        </router-link>

  <router-link class="list-group-item list-group-item-action d-flex align-items-center gap-2" :to="{ name: 'AddProfile' }">
  <i class="fas fa-building"></i> Complete Company Profile
</router-link>


      </div>
    </aside>
  </div>
</template>


<script setup>
    import { RouterLink } from 'vue-router';
    import { ref } from 'vue';
    const currUser = JSON.parse(localStorage.getItem('user'));


</script>

<style>
  .dashboard {
    min-height: 50vh;
  }

  .shadow-bg {
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 15px rgba(0, 0, 0, 0.05);
    padding: 25px;
    margin-bottom: 30px;
    border: 1px solid var(--border-color);
    transition: all 0.3s;
  }
  
  /* This will style the active link */
  .router-link-active, .router-link-exact-active {
    
    color: #000;
    font-weight: bold;
  }
.list-group-item i {
  min-width: 20px;
  text-align: center;
  color: #047fec;
}

  img {
    width: 100px;
    height: 100px;
    object-fit: fit(contain);
  }
</style>