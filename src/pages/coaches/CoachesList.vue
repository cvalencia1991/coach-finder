<template>
  <div>
    <base-dialog :show="!!error" 
      title="An error Occur"
      @close="handleError"
      >
      <p>{{ error }}</p>
    </base-dialog>
    <section>
      <coach-filter @change-filter="setFilters"></coach-filter>
    </section>
    <section>
      <base-card>
        <div class="controls">
          <base-button mode="outline" 
                       @click="loadCoaches(true)">Refresh</base-button>
          <base-button link 
                       to="/auth?redirect=coaches" 
                       v-if="!isloggedIn">Login to Register as Coach</base-button>
          <base-button v-if="isloggedIn && !isCoach && !isLoading" 
                       link to="/register">Register as Coach</base-button>
        </div>
        <div v-if="isLoading">
          <base-spinner></base-spinner>
        </div>
        <ul v-else-if="hasCoaches">
          <CoachItem
              v-for="coach in filteredCoaches"
              :key="coach.id"
              :id="coach.id"
              :first-name="coach.firstName"
              :last-name="coach.lastName"
              :rate="coach.hourlyRate"
              :areas="coach.areas"
              />
        </ul>
        <h3 v-else>No Coaches Found.</h3>
      </base-card>
    </section>
  </div>
</template>

<script>
import CoachItem from '../../components/coaches/CoachItem.vue';
import CoachFilter from '../../components/coaches/CoachFilter.vue';

export default {
  components: {
    CoachItem,
    CoachFilter
  },
  data() {
    return {
      isLoading: false,
      error: null,
      Activefilters: {
        frontend: true,
        backend: true,
        carrer: true
      }
    }
  },
  computed: {
    isloggedIn(){
      return this.$store.getters.isAuthenticated;
    },
    isCoach(){
      return this.$store.getters['coaches/isCoach'];
    },
    filteredCoaches() {
      const coaches = this.$store.getters['coaches/coaches'];
      return coaches.filter(coach => {
        if(this.Activefilters.frontend && 
          coach.areas.includes('frontend')){
          return true;
        }
        if(this.Activefilters.backend && 
          coach.areas.includes('backend')){
          return true;
        }
        if(this.Activefilters.carrer && 
          coach.areas.includes('carrer')){
          return true;
        }
        return false;
      });
    },
    hasCoaches() {
      return !this.isLoading && this.$store.getters['coaches/hasCoaches'];
    }
  },
  created() {
    this.loadCoaches();
  },
  methods: {
    setFilters(updatedFilters){
      this.Activefilters = updatedFilters;
    },
    async loadCoaches(refresh = false){
      this.isLoading = true;
      try{
        await this.$store.dispatch('coaches/loadCoaches', 
          { forceRefresh:refresh });
      } catch(error){
        this.error = error.message || 'Something wen wrong!';
      }
      this.isLoading = false;
    },
    handleError(){
      this.error = null;
    }
  },
}
</script>

<style scoped>
ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

.controls {
  display: flex;
  justify-content: space-between;
}
</style>
