<template>
  <div id="app" class="jobs-list-widget has-outline-buttons is-outdoor">
    <jobs-list v-show="!showSubscriberForm" :url="jobsUrl" />
    <a class="button" @click="showSubscriberForm = !showSubscriberForm">
      <span v-if="!showSubscriberForm">Subscribe to job updates</span>
      <span v-else>Back</span>
    </a>
    <subscribe-form v-if="showSubscriberForm" />
    <a
      class="button"
      href="https://secure.dc2.pageuppeople.com/apply/1029/vm/applicationForm/default.asp"
      target="_blank"
      >Existing applicant login</a
    >
  </div>
</template>

<script>
import JobsList from "./components/JobsList.vue";
import SubscribeForm from "./components/SubscribeForm.vue";

export default {
  name: "App",
  components: {
    JobsList,
    SubscribeForm,
  },
  data() {
    return {
      jobsUrl: "https://careers.pageuppeople.com/1029/vmo/en/jobs.json",
      showSubscriberForm: false,
    };
  },
};
</script>

<style lang="scss">
@import "/src/assets/scss/common.scss";
</style>
