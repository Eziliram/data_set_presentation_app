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
                attributes: [],
                users: [],
                attributeIds: []
            }
        },

        methods: {
            /**
             *
             */
            async processData() {
                // Get the data from the data file
                await $.get('anonymous-msweb.data', data => {

                    let dataLines = data.split('\n');

                    // Process Attributes and User data
                    dataLines.forEach(dataLine => {

                        let dataLineArray = dataLine.split(',');

                        switch (dataLine.charAt(0)) {
                            case 'A': // Attribute lines (vRoots)
                                this.attributes.push(this.getAttributes(dataLineArray));
                                break;
                            case 'C': // Case lines (users)
                                if (this.users.length > 0) {
                                    this.setAttributesForUser();
                                    this.attributeIds = [];
                                }
                                this.users.push(this.getUsers(dataLineArray));
                                break;
                            case 'V': // Vote lines (attributes/vRoots a user has visited)
                                this.setAttributeIdForUser(dataLineArray);
                                break;
                            case '': // End of file
                                this.setAttributesForUser();
                                break;
                            default:
                                break;
                        }
                    });
                });

                //todo: remove
                console.log(this.attributes)
                console.log(this.users)
            },
            /**
             *
             * @param data
             * @returns {{id: *, title: *, url: *}}
             */
            getAttributes(data) {

                let attributes = [];

                data.forEach(attribute => {
                    attributes.push(this.cleanString(attribute));
                });

                // Ignore elements containing 'A' and '1'
                return {
                    id: attributes[1],
                    title: attributes[3],
                    url: attributes[4]
                };
            },
            /**
             *
             * @param data
             * @returns {{name: *, id: *}}
             */
            getUsers(data) {

                let users = [];

                data.forEach(user => {
                    users.push(this.cleanString(user));
                });

                // Ignore element containing 'C'
                return {
                    id: users[1],
                    name: users[2],
                    attributeIds: []
                }
            },
            /**
             *
             * @param data
             */
            setAttributeIdForUser(data) {
                this.attributeIds.push(data[1])
            },
            /**
             *
             */
            setAttributesForUser() {
                this.users[this.users.length - 1].attributeIds = this.attributeIds;
            },
            cleanString(value) {
                return value.replace(/"/g, '');
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
