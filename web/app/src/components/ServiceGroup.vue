<template>
  <div :class="services.length === 0 ? 'mt-3' : 'mt-4'">
    <slot v-if="name !== 'undefined'">
      <div class="service-group pt-2 border dark:bg-gray-800 dark:border-gray-500" @click="toggleGroup">
        <h5 class='font-mono text-gray-400 text-xl font-medium pb-2 px-3 dark:text-gray-200 dark:hover:text-gray-500 dark:border-gray-500'>
          <span v-if="healthy" class='text-green-600'>&#10003;</span>
          <span v-else class='text-yellow-400'>~</span>
          {{ name }}
          <span class='float-right service-group-arrow'>
            {{ collapsed ? '&#9660;' : '&#9650;' }}
          </span>
        </h5>
      </div>
    </slot>
    <div v-if="!collapsed" :class="name === 'undefined' ? '' : 'service-group-content'">
      <slot v-for="service in services" :key="service">
        <Service
            :data="service"
            :maximumNumberOfResults="20"
            @showTooltip="showTooltip"
            @toggleShowAverageResponseTime="toggleShowAverageResponseTime" :showAverageResponseTime="showAverageResponseTime"
        />
      </slot>
    </div>
  </div>
</template>


<script>
import Service from './Service.vue';

export default {
  name: 'ServiceGroup',
  components: {
    Service
  },
  props: {
    name: String,
    services: Array,
    showAverageResponseTime: Boolean
  },
  emits: ['showTooltip', 'toggleShowAverageResponseTime'],
  methods: {
    healthCheck() {
      if (this.services) {
        for (let i in this.services) {
          for (let j in this.services[i].results) {
            if (!this.services[i].results[j].success) {
              // Set the service group to unhealthy (only if it's currently healthy)
              if (this.healthy) {
                this.healthy = false;
              }
              return;
            }
          }
        }
      }
      // Set the service group to healthy (only if it's currently unhealthy)
      if (!this.healthy) {
        this.healthy = true;
      }
    },
    toggleGroup() {
      this.collapsed = !this.collapsed;
      sessionStorage.setItem(`gatus:service-group:${this.name}:collapsed`, this.collapsed);
    },
    showTooltip(result, event) {
      this.$emit('showTooltip', result, event);
    },
    toggleShowAverageResponseTime() {
      this.$emit('toggleShowAverageResponseTime');
    }
  },
  watch: {
    services: function () {
      this.healthCheck();
    }
  },
  created() {
    this.healthCheck();
  },
  data() {
    return {
      healthy: true,
      collapsed: sessionStorage.getItem(`gatus:service-group:${this.name}:collapsed`) === "true"
    }
  }
}
</script>


<style>
.service-group {
  cursor: pointer;
  user-select: none;
}

.service-group h5:hover {
  color: #1b1e21;
}
</style>