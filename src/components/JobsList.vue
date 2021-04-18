<template>
    <div class="jobs-list">
        <div class="jobs-fields jobs-field-headers">
            <div class="job-field job-field--position">Position</div>
            <div class="job-field job-field--type">Type</div>
            <div class="job-field job-field--location">Location</div>
            <div class="job-field job-field--closing">Closing</div>
        </div>
        <div
            v-for="job in filteredJobs"
            :key="job.Id"
            class="jobs-fields">
            <div class="job-field job-field--position">
                <a :href="job.ApplyUrl" target="_blank">{{ job.Title }}</a>
                <p>{{ job.Summary }}</p>
            </div>
            <div class="job-field job-field--type">{{ job.workTypesList }}</div>
            <div class="job-field job-field--location">{{ job.locationsList }}</div>
            <div class="job-field job-field--closing">{{ job.closingDate }}</div>
        </div>
        <div class="job-filters">
            <div
                v-for="filter in filterOptions"
                :key="filter.label">
                <p>{{ filter.label }}:</p>
                <div
                    v-for="(transformedOption, option) in filter.options"
                    :key="option">
                    <label>
                        <input v-model="appliedFilters[filter.field]" :value="option" type="checkbox">
                        {{ transformedOption }}
                    </label>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
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
            appliedFilters: {
                WorkTypeList: [],
                LocationList: [],
                CategoryList: [],
            },
            filterFields: [
                {
                    label: 'Work types',
                    field: 'WorkTypeList',
                },
                {
                    label: 'Locations',
                    field: 'LocationList',
                    transformer: (value) => value.replace(/^[^|]+\|/, ''),
                },
                {
                    label: 'Categories',
                    field: 'CategoryList',
                    transformer: (value) => value.replace(/^[^|]+\|/, ''),
                },
            ],
        };
    },
    computed: {
        jobs() {
            return (this.data || []).map((job) => {
                const j = { ...job };
                j.workTypesList = j.WorkTypeList.join(', ');
                j.locationsList = j.LocationList.map((location) => location.replace(/^[^|]+\|/, '')).join(', ');
                j.closingDate = this.getClosingDate(j);
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
                    const jobIncludesValue = jobValueForFilter.some((v) => filterValue.includes(v));
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
            return Object.entries(this.appliedFilters)
                .filter(([, value]) => value.length > 0);
        },
    },
    created() {
        axios.get(this.url)
            .then((resp) => {
                this.data = resp.data;
            });
    },
    methods: {
        getClosingDate(job) {
            if (!job.ClosingDateUtc) {
                return '-';
            }
            const dateString = job.ClosingDateUtc.replace(/^.+\(([0-9]+)\).+$/, '$1');
            if (!dateString) {
                return '-';
            }
            const parsed = new Date(parseInt(dateString));
            const month = parsed.getMonth() + 1;
            const day = parsed.getDate();
            const year = parsed.getFullYear();
            return `${day}/${month}/${year}`;
        },
    },
}
</script>

<style lang="scss" scoped>
.jobs-fields {
    display: flex;
}

.job-field {
    flex: 1;
}

.job-field--position {
    flex-grow: 3;
}


</style>