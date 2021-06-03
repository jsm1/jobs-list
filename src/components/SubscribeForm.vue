<template>
<div>
    <!-- Place these wherever you'd like the widgets to appear. They can be anywhere, they don't have to be next to each other. -->
<div id="jobSearch">Loading...</div>
<div id="jobListing">Loading...</div>
</div>
</template>

<script>
export default {
    methods: {
        init() {
            window.PU.Widgets.jobMail(
                "jobListing",
                "http://careers.pageuppeople.com/1029/",
                "VAL",
                "en",
                {
                    filters: { 'Keyword': 'Keywords', 'Locations': 'Where ya wanna go'},
                    subscribeButtonText: 'Farts'
                }
            );
        },
    },
    created() {
        window.PU.DomReady.add(() => {
            this.init();
        });
    },
}
</script>
