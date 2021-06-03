<template>
  <div id="app">
    <jobs-list :url="jobsUrl" />
    <subscribe-form v-if="showSubscriberForm" />
  </div>
</template>

<script>
import JobsList from './components/JobsList.vue'
import SubscribeForm from './components/SubscribeForm.vue';

export default {
  name: 'App',
  components: {
    JobsList,
    SubscribeForm
  },
  data() {
    return {
      jobsUrl: 'https://careers.pageuppeople.com/1029/val/en/jobs.json',
      showSubscriberForm: true,
    };
  },
}
</script>
