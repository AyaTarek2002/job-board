  <template>
    <div class="container">
      <h2 class="text-center mb-4">Job Listings</h2>

      <div v-if="jobs.length && jobs[0] !== 'You don\'t have any jobs yet.'" class="table-responsive">
        <table class="table table-striped table-hover">
          <thead class="table-dark">
            <tr>
              <th scope="col">Job Title</th>
              <th scope="col">Technologies</th>
              <th scope="col">Salary</th>
              <th scope="col">Status</th>
              <th scope="col">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(job, index) in jobs" :key="index">
              <td>
                <div>
                  <strong>{{ job.title }}</strong>
                  <br>
                  <small class="text-muted">{{ job.location }} | {{ job.work_type }}</small>
                </div>
              </td>
              <td>
                <span class="badge bg-secondary me-1" v-for="tech in getTechnologiesArray(job.technologies)" :key="tech">
                  {{ tech }}
                </span>
              </td>
              <td>{{ job.salary }}</td>
              <td>
                <span class="badge" :class="getStatusBadgeClass(job.status)">
                  {{ job.status || 'Active' }}
                </span>
              </td>
              <td>
                <button class="btn btn-outline-primary btn-sm" @click="viewJobDetails(job)">
                  <i class="fas fa-eye me-1"></i>View Details
                </button>
              </td>
            </tr>
          </tbody>
        </table>
        <div v-if="jobs.length && jobs[0] !== 'You don\'t have any jobs yet.'" class="text-center mt-4">
          <nav>
            <ul class="pagination justify-content-center">
              <li class="page-item" :class="{ disabled: currentPage === 1 }">
                <button class="page-link" @click="fetchJobs(currentPage - 1)">Previous</button>
              </li>

              <li
                v-for="page in paginationRange"
                :key="page"
                class="page-item"
                :class="{ active: page === currentPage }"
              >
                <button class="page-link" @click="fetchJobs(page)">
                  {{ page }}
                </button>
              </li>

              <li class="page-item" :class="{ disabled: currentPage === lastPage }">
                <button class="page-link" @click="fetchJobs(currentPage + 1)">Next</button>
              </li>
            </ul>
          </nav>
        </div>  

      </div>

      <div v-else class="text-center">
        <div class="alert alert-info" role="alert">
          <i class="fas fa-info-circle me-2"></i>
          {{ jobs[0] === "You don't have any jobs yet." ? jobs[0] : "No jobs available at the moment." }}
        </div>
      </div>
    </div>
  </template>

  <script setup>
    import { ref, onMounted, computed } from 'vue';
    import axios from 'axios';
    import { useRouter } from 'vue-router';

    const router = useRouter();
    const currUser = JSON.parse(localStorage.getItem('user'));

    if (!currUser) {
      router.push('/login');
    }
    if (currUser.role !== 'employer') {
      router.push('/');
    }

    const jobs = ref([]);
    const currentPage = ref(1);
    const lastPage = ref(1);

    function fetchJobs(page = 1) {
      axios.get(`http://localhost:8000/api/employers/jobs/${currUser.id}?page=${page}&per_page=3`, {
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then(response => {
        if (response.data.message === "You don't have any jobs yet.") {
          jobs.value = [response.data.message];
        } else {
          jobs.value = response.data.data.data; // because paginated data is nested under data
          currentPage.value = response.data.data.current_page;
          lastPage.value = response.data.data.last_page;
        }
      })
      .catch(error => {
        console.error(error);
      });
    }

    onMounted(() => {
      fetchJobs();
    });

    function viewJobDetails(job) {
      router.push({ path: '/jobs/' + job.id });
    }

    function getTechnologiesArray(technologies) {
      if (!technologies) return [];
      if (typeof technologies === 'string') {
        return technologies.split(',').map(tech => tech.trim());
      }
      return Array.isArray(technologies) ? technologies : [];
    }

    function getStatusBadgeClass(status) {
      const statusLower = (status || 'active').toLowerCase();
      switch (statusLower) {
        case 'published':
          return 'bg-success';
        case 'rejected':
          return 'bg-danger';
        case 'pending':
          return 'bg-warning';
        default:
          return 'bg-primary';
      }
    }


    const paginationRange = computed(() => {
      const range = [];
      const delta = 2; // how many pages to show before/after current
      const start = Math.max(1, currentPage.value - delta);
      const end = Math.min(lastPage.value, currentPage.value + delta);

      for (let i = start; i <= end; i++) {
        range.push(i);
      }
      return range;
    });
  </script>

<style scoped>
.table {
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
  background-color: #fdfdfd;
}

thead th {
  background-color: #e2e8f0; /* رمادي فاتح للهيدر */
  color: #2d3748; /* رمادي غامق للنص */
  text-align: center;
  vertical-align: middle;
}

tbody td {
  vertical-align: middle;
  text-align: center;
  color: #4a5568; /* رمادي معتدل */
}

tbody td strong {
  color: #2d3748; /* أغمق شويه للعنوان */
}

tbody td small {
  color: #718096; /* رمادي فاتح للتفاصيل */
  font-weight: 500;
}

.badge {
  font-size: 0.85rem;
  padding: 0.4em 0.7em;
  border-radius: 1rem;
  font-weight: 500;
}

.bg-success {
  background-color: #38a169 !important; /* أخضر هادي */
  color: white;
}

.bg-danger {
  background-color: #e53e3e !important; /* أحمر أنعم */
  color: white;
}

.bg-warning {
  background-color: #d69e2e !important; /* أصفر ذهبي */
  color: white;
}

.bg-primary {
  background-color: #4299e1 !important; /* أزرق ناعم */
  color: white;
}

.bg-secondary {
  background-color: #cbd5e0 !important; /* رمادي أزرق */
  color: #2d3748;
}

.table td, .table th {
  padding: 1rem;
}

.table-hover tbody tr:hover {
  background-color: #f0f4f8; /* Hover ناعم */
}

button.btn {
  min-width: 90px;
  font-weight: 500;
}

.btn-outline-primary {
  border-color: #4299e1;
  color: #4299e1;
}

.btn-outline-primary:hover {
  background-color: #4299e1;
  color: white;
}

.modal-title {
  font-weight: bold;
} 

@media (max-width: 768px) {
  .table-responsive {
    font-size: 0.875rem;
  }

  .btn-sm {
    padding: 0.2rem 0.4rem;
    font-size: 0.75rem;
  }
}
</style>
