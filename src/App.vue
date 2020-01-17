<template>
    <div id="app">

    </div>
</template>

<script>
    import $ from 'jquery'

    export default {
        name: 'App',

        mounted() {
            this.processData();
        },

        data: () => {
            return {

            }
        },

        methods: {
            /**
             *
             */
            async processData() {

                let attributes = [];
                let users = [];

                // Get the data from the data file
                await $.get('anonymous-msweb.data', data => {

                    let dataLines = data.split('\n');

                    dataLines.forEach(dataLine => {
                        if (dataLine.charAt(0) === 'A') attributes.push(this.getAttributes(dataLine.split(',')));
                        // TODO: store users
                    });
                });

                console.log(attributes)
            },
            /**
             *
             * @param data
             * @returns {{id: *, title: *, url: *}}
             */
            getAttributes(data) {

                let attributes = [];

                data.forEach(attribute => {
                    // Ignore the elements containing 'A' and '1'
                    if (attribute === 'A' || attribute === '1') return; //todo: filter this in a better way
                    attributes.push(attribute.replace(/"/g, ''));
                });

                return {
                    id: attributes[0],
                    title: attributes[1],
                    url: attributes[2]
                };
            },
            getUsers(data) {

            }
        }
    }
</script>

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }
</style>
