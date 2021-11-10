<template>
<div>
    <!-- Place these wherever you'd like the widgets to appear. They can be anywhere, they don't have to be next to each other. -->
<div id="jobListing">Loading...</div>
</div>
</template>

<script>
export default {
    methods: {
        init() {
            window.PU.Widgets.jobMail(
                "jobListing",
                "https://careers.pageuppeople.com/1029/",
                "VAL",
                "en",
                {
                    filters: { 'Keyword': 'Keywords', 'Locations': 'Locations', 'Categories': 'Categories', 'WorkTypes': 'Work Types'},
                    subscribeButtonText: 'Subscribe'
                }
            );
        },
    },
    mounted() {
        this.init();
    },
}
</script>

<style lang="scss">
label {
    padding-left: 6px;
}

.field-wrapper {
    list-style-type: none;
    h4 {
        display: none !important;
    }
    ul {
        list-style-type: none;
        padding-left: 0;
    }
    padding-left: 0;

    
}

h3.field-title {
    margin-bottom: 1rem;
    margin-top: 2rem;
}

.job-mail-wrapper {
    display: flex;
    flex-wrap: wrap;
    .field-wrapper:nth-child(1), .field-wrapper:nth-child(2), .message-wrapper {
        flex-basis: 100%;
    }
    #PageUpPeopleLink {
        flex-basis: 100%;
        display: none;
    }

    .field-wrapper {
        margin-right: 2rem;
    }
}
</style>
