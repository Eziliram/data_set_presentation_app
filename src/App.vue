<template>
    <div id="app">
        <div>
            <label for="filter_by">Filter by attribute</label>

            <select id="filter_by">
                <option value="">Please select</option>
                <template v-for="attribute in attributes">
                    <option :value="attribute.id">{{ attribute.title }}</option>
                </template>
            </select>

            <button @click="applyFilter()">Apply Filter</button>
            <button @click="removeFilter()">Remove Filters</button>
        </div>
        
        <div id="data_container">
            <div class="header data-row">
                <div>
                    <label>
                        <input id="select_all" type="checkbox"/>
                    </label>
                </div>
                <div>ID</div>
                <div>USER</div>
                <div>ATTRIBUTES</div>
            </div>

            <div class="data-row" v-for="(user, index) in displayedRecords" v-if="index < numberOfRecordsPerPage">
                <div>
                    <label>
                        <input :id="'chk_' + user.id" type="checkbox"/>
                    </label>
                </div>
                <div>{{ user.id }}</div>
                <div @click="selectedUserIndex = index">
                    <span v-if="selectedUserIndex !== index">{{ user.name }}</span>
                    <label>
                        <input v-if="selectedUserIndex === index" type="text" v-model="user.name"/>
                    </label>
                </div>
                <div>
                    <span v-for="(attribute, index) in user.attributeIds">{{ attribute }}
                        <span v-if="user.attributeIds.length !== (index + 1)">| </span>
                    </span>
                </div>
            </div>

<!--            TODO: include pagination at top of page as well -->
            <div id="pagination">
                <button id="btn_back_to_start" @click="backToStart()">Back to start</button>
                <button id="btn_previous" @click="previous()">Previous</button>
                <button id="btn_next" @click="next()">Next</button>
                <button id="btn_go_to_end" @click="goToEnd()">Go to end</button>
            </div>
        </div>
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
                filter: false,
                filteredUsers: [],

                selectedUserIndex: null,

                currentPage: 0,
                numberOfRecordsPerPage: 50,

                attributes: [],
                users: [],
                attributeIds: []
            }
        },

        computed: {
            displayedRecords() {

                if (!this.filter) return this.users.slice(this.currentPage, this.currentPage + this.numberOfRecordsPerPage);
                this.filter = false;
                return this.filteredUsers;
            },

        },

        methods: {
            //<editor-fold desc="Data Processing">
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
                                    this.attributeIds = []; // Empty array for the next user
                                }
                                this.users.push(this.getUser(dataLineArray));
                                break;
                            case 'V': // Vote lines (attributes/vRoots for a user)
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
                // console.log(this.attributes)
                // console.log(this.users)
            },
            /**
             *
             * @param data
             * @returns {{id: *, title: *, url: *}}
             */
            getAttributes(data) {

                let attributes = [];

                data.forEach(attribute => attributes.push(this.cleanString(attribute)));

                // Ignore elements containing 'A' (0) and '1' (2)
                return {
                    id: attributes[1],
                    title: attributes[3],
                    url: attributes[4]
                };
            },
            /**
             * Constructs an object containing a user's data
             * @param data
             * @returns {{name: *, id: *}}
             */
            getUser(data) {

                let users = [];

                data.forEach(user => users.push(this.cleanString(user)));

                // Ignore element containing 'C' (0)
                return {
                    id: users[1],
                    name: users[2],
                    attributeIds: []
                }
            },
            /**
             * Compiles the group of attributes related to a user
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
            //</editor-fold>

            //<editor-fold desc="Pagination">
            //TODO: if user goes beyond next or more back than back
            backToStart() {
                this.currentPage = 0;
            },
            previous() {
                this.currentPage -= this.numberOfRecordsPerPage;
            },
            next() {
                this.currentPage += this.numberOfRecordsPerPage;
            },
            goToEnd() {
                this.currentPage = this.users.length -1;
            },
            //</editor-fold>

            //<editor-fold desc="Table Features">
            /**
             *
             */
            applyFilter() {

                const selectedFilterOption = document.getElementById('filter_by').value;
                let filteredUsers = [];

                this.filter = true;
                this.users.forEach(user => {
                    let result = user.attributeIds.filter((attr) => { return attr === selectedFilterOption });
                    if (result.length !== 0) filteredUsers.push(user);
                });

                filteredUsers.forEach(user => this.filteredUsers.push(user));
            },
            removeFilter() {
                this.filter = false;
                this.filteredUsers = [];
                document.getElementById('filter_by').value = '';
            },
            //</editor-fold>

            //<editor-fold desc="Utility Functions">
            cleanString(value) {
                return value.replace(/"/g, '');
            }
            //</editor-fold>
        }
    }
</script>

<style lang="stylus">
    #app
        font-family 'Roboto', Helvetica, Arial, sans-serif
        color #2C3E50
        margin-top 60px

    #filter_by
        option
            &:first-child
                color #8D9AA8
                font-style italic

    .header
        font-weight 600
        border-bottom 1px solid #2C3E50

    .data-row
        display flex
        flex-direction row
        div
            &:first-child
                width 30px
            &:nth-child(2)
                width 100px
            &:nth-child(3)
                width 100px
</style>
