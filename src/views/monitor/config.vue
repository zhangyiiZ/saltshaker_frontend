<template>
    <div>
        <git-lab
                :apiService="apiService"
                :projectType="projectType"
                :slsURI = "slsURI"
                :projectShow="true"
                :productShow="true"
                ref="childrenMethods">
        </git-lab>
    </div>
</template>

<script>
    import GitLab from '../common-components/git-lab_config.vue';
    export default {
        components: {
            GitLab
        },
        data () {
            return {
                apiService: 'config',
                projectType: 'state_project',
                slsURI: 'execute/sls'
            };
        }
    };
</script>