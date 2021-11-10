<template>
  <div>
    <div v-if="!selectedJob" class="jobs-list">
      <div class="jobs-fields-flex-parent">
        <div class="job-fields-flex-child">
          <div class="jobs-fields jobs-field-headers">
            <div class="job-field job-field--position">Position</div>
            <div class="job-field job-field--type">Type</div>
            <div class="job-field job-field--location">Location</div>
            <div class="job-field job-field--closing">Closing</div>
          </div>
          <div v-for="job in filteredJobs" :key="job.Id" class="jobs-fields">
            <div class="job-field job-field--position">
              <a class="job-title" :href="'#jobs/' + job.Id">{{ job.Title }}</a>
              <p>{{ job.Summary }}</p>
            </div>
            <div class="job-field job-field--type">{{ job.workTypesList }}</div>
            <div class="job-field job-field--location">
              {{ job.locationsList }}
            </div>
            <div class="job-field job-field--closing">
              {{ job.closingDate }}
            </div>
          </div>
        </div>
        <div class="jobs-fields-flex-child" id="job-filter">
          <div class="job-filters">
            <div
              v-for="filter in filterOptions"
              :key="filter.label"
              class="filter-title"
            >
              <p>
                <strong>{{ filter.label }}:</strong>
              </p>
              <div
                v-for="(transformedOption, option) in filter.options"
                :key="option"
              >
                <label>
                  <input
                    v-model="appliedFilters[filter.field]"
                    :value="option"
                    type="checkbox"
                  />
                  {{ transformedOption }}
                </label>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="job-detail-view">
      <div class="job-detail-view--hero">
        <p class="job-title">{{ selectedJob.Title }}</p>
        <div class="job-detail-view--buttons">
          <a :href="selectedJob.ApplyUrl" target="_blank" class="button"
            >Apply now</a
          >
          <a href="#" class="job-detail-view--text-link"
            >Back to job listings</a
          >
        </div>
      </div>
      <p><strong>Job no:</strong> {{ selectedJob.Id }}</p>
      <p><strong>Work type:</strong> {{ selectedJob.workTypesList }}</p>
      <p><strong>Location:</strong> {{ selectedJob.locationsList }}</p>
      <p><strong>Categories:</strong> {{ selectedJob.categoriesList }}</p>
      <div v-html="selectedJob.Overview"></div>
      <div class="job-detail-view--buttons">
        <a :href="selectedJob.ApplyUrl" target="_blank" class="button"
          >Apply now</a
        >
        <p>
          <a href="#" class="job-detail-view--text-link"
            >Back to job listings</a
          >
        </p>
      </div>
      <p><strong>Advertised:</strong> {{ selectedJob.openingDate }}</p>
      <p>
        <strong>Applications close:</strong>
        {{ selectedJob.closingDateWritten }}
        {{ selectedJob.ClosingDateTimeZoneAbbr }}
      </p>
    </div>
  </div>
</template>

<script>
import { DateTime } from "luxon";
import axios from "axios";
const DateFormat = {
  DMY: "d/L/y",
  Written: "d MMMM y",
};
export default {
  props: {
    url: {
      type: String,
      default: null,
    },
  },
  data() {
    return {
      data: null,
      selectedJob: null,
      selectedJobId: null,
      appliedFilters: {
        WorkTypeList: [],
        LocationList: [],
        CategoryList: [],
      },
      filterFields: [
        {
          label: "Work types",
          field: "WorkTypeList",
        },
        {
          label: "Locations",
          field: "LocationList",
          transformer: this.formatField,
        },
        {
          label: "Categories",
          field: "CategoryList",
          transformer: this.formatField,
        },
      ],
    };
  },
  computed: {
    jobs() {
      return (this.data || []).map((job) => {
        const j = { ...job };
        j.workTypesList = j.WorkTypeList.join(", ");
        j.locationsList = j.LocationList.map(this.formatField).join(", ");
        j.categoriesList = j.CategoryList.map(this.formatField).join(", ");
        j.closingDate = this.getClosingDate(j, DateFormat.DMY);
        j.openingDate = this.getOpeningDate(j);
        j.closingDateWritten = this.getClosingDate(j, DateFormat.Written);
        return j;
      });
    },
    filteredJobs() {
      if (!this.isFilterSet) {
        return this.jobs;
      }
      return this.jobs.filter((job) => {
        for (const [key, filterValue] of this.setFilters) {
          const jobValueForFilter = job[key];
          const jobIncludesValue = jobValueForFilter.some((v) =>
            filterValue.includes(v)
          );
          if (!jobIncludesValue) {
            return false;
          }
        }
        return true;
      });
    },
    filterOptions() {
      return this.filterFields.map((field) => {
        const valueOptions = {};
        this.jobs.forEach((j) => {
          for (const v of j[field.field]) {
            const value = field.transformer ? field.transformer(v) : v;
            valueOptions[v] = value;
          }
        });
        return {
          ...field,
          options: valueOptions,
        };
      });
    },
    isFilterSet() {
      return this.setFilters.length;
    },
    setFilters() {
      return Object.entries(this.appliedFilters).filter(
        ([, value]) => value.length > 0
      );
    },
  },
  watch: {
    selectedJobId() {
      if (this.selectedJobId) {
        this.selectedJob = this.jobs.find((j) => j.Id === this.selectedJobId);
      } else {
        this.selectedJob = null;
      }
    },
    jobs() {
      this.onHashChange();
    },
  },
  created() {
    window.addEventListener("hashchange", this.onHashChange);
    axios.get(this.url).then((resp) => {
      this.data = resp.data;
    });
  },
  methods: {
    getClosingDate(job, format) {
      if (!job.ClosingDateUtc) {
        return "-";
      }
      return this.getDateAsString(
        job.ClosingDateUtc,
        job.ClosingDateUtcOffset,
        format
      );
    },
    getDateAsString(string, offset, format) {
      const dateString = string.replace(/^.+\(([0-9]+)\).+$/, "$1");
      if (!dateString) {
        return "-";
      }
      const parsed = new Date(parseInt(dateString));
      const luxon = DateTime.fromJSDate(parsed);
      const roundedOffset = Math.round(parseFloat(offset));
      return luxon.setZone("UTC+" + roundedOffset).toFormat(format);
    },
    getOpeningDate(job) {
      if (!job.OpeningDateUtc) {
        return "-";
      }
      return this.getDateAsString(
        job.OpeningDateUtc,
        job.OpeningDateUtcOffset,
        DateFormat.Written
      );
    },
    formatField(fieldValue) {
      return (fieldValue || "").replace(/^[^|]+\|/, "");
    },
    onHashChange() {
      const hash = window.location.hash;
      const regex = /^.+jobs\/([a-zA-Z0-9]+)[^a-zA-Z0-9]*$/;
      const matches = hash.match(regex);
      if (matches) {
        this.selectedJobId = matches[1];
      } else {
        this.selectedJobId = null;
      }
    },
  },
};
</script>

<style lang="scss"></style>
