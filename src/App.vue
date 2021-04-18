<template>
  <div id="app">
    <jobs-list :url="jobsUrl" />
  </div>
</template>

<script>
import JobsList from './components/JobsList.vue'

export default {
  name: 'App',
  components: {
    JobsList
  },
  data() {
    return {
      jobsUrl: 'http://careers.pageuppeople.com/1029/val/en/jobs.json',
    };
  },
}
</script>
