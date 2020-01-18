<template>
    <div id="app">
        <div id="collections_container">
            <h3>Collections</h3>

            <div v-for="collection in collections"
                 class="collection"
                 :class="{ 'selected-collection': selectedCollection === collection.id }">
                <div @click="viewCollection(collection.id, collection.records)">{{ collection.name }}</div>
                <img class="delete" src="./assets/delete.svg" @click="deleteCollection(collection)" alt="Delete">
            </div>

            <p v-if="collections.length === 0">You have no collections.</p>

            <div>
                <label>
                    <input id="chk_new_collection" type="checkbox" @change="isNewCollection = !isNewCollection" checked>
                    Save selected records to a new collection?
                </label>

                <template v-if="isNewCollection">
                    <label>
                        <input id="txt_collection_name" type="text" placeholder="Collection name"/>
                    </label>

                    <button id="btn_create_new_collection" @click="addToNewCollection()">Create new collection</button>
                </template>

                <template v-else>
                    <template v-if="collections.length !== 0">
                        <label for="drp_collections">Add to existing list</label>

                        <select id="drp_collections">
                            <template v-for="(collection, index) in collections">
                                <option :value="index">{{ collection.name }}</option>
                            </template>
                        </select>

                        <button @click="addToExistingCollection()">Add</button>
                    </template>

                    <template v-else>
                        <p>You have no saved collections.</p>
                    </template>
                </template>
            </div>
        </div>

<!--        TODO: Create a global reset/clear button -->

        <div id="dashboard">
            <div id="query_container">
                <div>
                    <button @click="reset()">Reset</button>
                </div>

                <div id="search_container">
                    <label>
                        <input id="txt_search" type="text" placeholder="Search value"/>
                    </label>

                    <button id="btn_search" @click="search()">Search</button>
                    <button id="btn_clear" @click="clearSearch()">Clear</button>
                </div>

                <div id="filter_container">
                    <label for="drp_filter"/>
                    <select id="drp_filter">
                        <option value="" disabled selected>Please select</option>
                        <template v-for="attribute in attributes">
                            <option :value="attribute.id">{{ attribute.title }}</option>
                        </template>
                    </select>

                    <button id="btn_apply_filter" @click="applyFilter()">Apply Filter</button>
                    <button id="btn_remove_filter" @click="clearFilter()">Clear</button>
                </div>

                <div>
                    <label>
                        Number of records displayed
                        <input type="text" v-model="numberOfRecordsPerPage"/>
                    </label>
                </div>
            </div>

            <div id="table-container">
                <div id="pagination">
                    <button id="btn_back_to_start" @click="backToStart()">Back to start</button>
                    <button id="btn_previous" @click="previous()">Previous</button>
                    <button id="btn_next" @click="next()">Next</button>
                    <button id="btn_go_to_end" @click="goToEnd()">Go to end</button>
                </div>

                <div id="data_container">
                    <div class="table-header data-row">
                        <div>
                            <label>
<!--                                TODO: implement select all -->
                                <input v-if="!isCollection" id="select_all" type="checkbox"/>
                            </label>
                        </div>
                        <div>ID</div>
                        <div>
                            <span>USER</span> |
                            <span id="sort" @click="sort()">sort</span>
                        </div>
                        <div>ATTRIBUTES</div>
                        <div v-if="isCollection"/>
                    </div>

                    <div v-if="index < numberOfRecordsPerPage"
                         v-for="(user, index) in displayedRecords"
                         class="data-row">
                        <div v-if="!isCollection">
                            <label>
                                <input :id="'chk_' + user.id" type="checkbox" @change="setSelectedRecord(user)" :checked="checked(user)"/>
                            </label>
                        </div>

                        <div v-else>
                            <img class="delete" src="./assets/delete.svg" @click="deleteRecord(index)" alt="Delete">
                        </div>

                        <div>{{ user.id }}</div>

                        <div @click="selectedUserIndex = index">
                            <span v-if="selectedUserIndex !== index">{{ user.name }}</span>

                            <label>
                                <input v-if="selectedUserIndex === index" type="text" v-model="user.name"/>
                            </label>
                        </div>

                        <div>
                            <span v-for="(attribute, index) in user.attributeIds">{{ mapAttributeId(attribute) }}
                                <span v-if="user.attributeIds.length !== (index + 1)">| </span>
                            </span>
                        </div>
                    </div>

                    <p v-if="displayedRecords.length === 0">No records found.</p>

                    <!-- TODO: add pagination to bottom of page as well (create a component) -->
                </div>
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
                ascending: true, // By default ascending

                selectedCollection: null,
                isCollection: false,
                collection: [],
                collections: [
                    {
                        id: 1,
                        name: 'Test',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    }
                ],

                isResult: false,
                results: [],

                selectedRecords: [],
                selectedUserIndex: null, //todo: rename
                isNewCollection: true,

                currentPage: 0,
                numberOfRecordsPerPage: 10, //todo: set to 50 when done

                attributes: [],
                users: [],
                attributeIds: []
            }
        },

        computed: {
            // The records that are currently displayed on the screen
            displayedRecords() {

                let records = this.isResult
                    ? this.results
                    : this.isCollection
                        ? this.collection
                        : this.users;

                return records.slice(this.currentPage, this.currentPage + this.numberOfRecordsPerPage);
            }
        },

        methods: {
            //<editor-fold desc="Data Processing">
            /**
             *
             */
            async processData() {
                // Get the data from the data file
                //todo: js way to get data?
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
            //TODO: if user goes beyond next or more back than back, disable buttons
            backToStart() {
                this.currentPage = 0;
            },
            previous() {
                if (this.currentPage !== 0) this.currentPage -= parseInt(this.numberOfRecordsPerPage);
            },
            next() {
                const records = this.isResult ? this.results : this.users;
                if (records.length <= (this.currentPage + parseInt(this.numberOfRecordsPerPage))) return;
                this.currentPage += parseInt(this.numberOfRecordsPerPage);
            },
            goToEnd() {
                const records = this.isResult ? this.results : this.users;
                if (records.length <= parseInt(this.numberOfRecordsPerPage)) return;
                this.currentPage = records.length - 1;
            },
            //</editor-fold>

            //<editor-fold desc="Table Features">
            /**
             *
             * @param id
             * @returns {T | string | *}
             */
            mapAttributeId(id) {
                return this.attributes.find(attribute => {
                    if (attribute.id === id) return attribute.title
                }).title;
            },
            reset() {
                this.currentPage = 0;
                this.clearSearch();
                this.clearFilter();
                this.clearCheckboxes();
                this.toggleCollection(false);
                this.selectedCollection = 0;
            },
            sort() {
                this.ascending = !this.ascending;

                let records = this.isResult
                    ? this.results
                    : this.isCollection
                        ? this.collection
                        : this.users;

                records.sort((a, b) => {
                    if (this.ascending ? a.name < b.name : a.name > b.name) return -1;
                    if (this.ascending ? a.name > b.name : a.name < b.name) return 2;
                    return 0;
                })
            },
            /**
             *
             */
            search() {
                //TODO: implement 'loading'
                const searchValue = document.getElementById('txt_search').value;
                let searchResults = this.users.filter(user => {
                    if (user.name.match(searchValue) !== null) return user.name.match(searchValue);
                    let attributeResults = user.attributeIds.filter(id => {
                        return this.mapAttributeId(id).toLowerCase().match(searchValue.toLowerCase())
                    });
                    if (attributeResults.length !== 0) return user;
                });

                this.toggleResults();
                searchResults.forEach(result => this.results.push(result));
            },
            clearSearch() {
                this.toggleResults(false);
                document.getElementById('txt_search').value = '';
            },
            /**
             *
             */
            applyFilter() {

                const selectedFilterOption = document.getElementById('drp_filter').value;
                let filteredResults = [];

                this.toggleResults();
                this.users.forEach(user => {
                    let result = user.attributeIds.filter(attr => { return attr === selectedFilterOption });
                    if (result.length !== 0) filteredResults.push(user);
                });

                filteredResults.forEach(result => this.results.push(result));
            },
            /**
             *
             */
            clearFilter() {
                this.toggleResults(false);
                document.getElementById('drp_filter').value = '';
            },
            clearCheckboxes() {
                this.selectedRecords = []; // Clears selected items
            },
            toggleResults(isResult = true) {
                this.results = [];
                this.currentPage = 0;
                this.isResult = isResult;
            },
            toggleCollection(isCollection = true) {
                this.collection = [];
                this.currentPage = 0;
                this.isCollection = isCollection;
            },
            checked(user) {
                return this.selectedRecords.find(record => { return record.id === user.id }) !== undefined;
            },
            /**
             * Add/removes the record that was selected/deselected to/from an array
             * todo: rename function name
             */
            setSelectedRecord(user) {
                document.getElementById('chk_' + user.id).checked
                    ? this.selectedRecords.push(user)
                    : this.selectedRecords.find((record, index) => {
                        if (record && record.id === user.id) this.selectedRecords.splice(index, 1);
                    });
            },
            deleteRecord(userIndex) {

                let collectionIndex = this.findCollectionIndex();

                this.collections[collectionIndex].records.find((record, index) => {
                    if (index === userIndex) this.collections[collectionIndex].records.splice(index, 1); // Delete record
                });

                this.collection = this.collections[collectionIndex].records; // Display updated records
            },
            /**
             *
             */
            addToNewCollection() {
                //todo: apply validation if field is null
                const name = document.getElementById('txt_collection_name').value;

                this.collections.push({ id: Date.now(), name, records: this.selectedRecords });
                this.clearCheckboxes();
                document.getElementById('txt_collection_name').value = '';
            },
            /**
             *
             */
            addToExistingCollection() {

                let collectionIndex = document.getElementById('drp_collections').value;

                this.selectedRecords.forEach(selectedRecord => {
                    const result = this.collections[collectionIndex].records.find(record => { return selectedRecord.id === record.id; });
                    if (result === undefined) this.collections[collectionIndex].records.push(selectedRecord);
                    else alert('Item already in collection'); // todo: handle this better?
                });

                this.clearCheckboxes();
            },
            viewCollection(id, records) {
                this.selectedCollection = id;
                this.toggleCollection();
                records.forEach(record => this.collection.push(record));
            },
            deleteCollection(selectedCollection) {
                this.reset();
                this.collections.find((collection, index) => {
                    if (selectedCollection.id === collection.id) this.collections.splice(index, 1); // Delete collection
                })
            },
            //</editor-fold>

            //<editor-fold desc="Utility Functions">
            //todo: this function is only used in one place?
            findCollectionIndex() {
                return this.collections.findIndex(collection => {
                    return collection.id === this.selectedCollection;
                });
            },
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
        display flex
        flex-direction row
        #collections_container
            height 100vh
            width 15%
            border-right 1px solid #2C3E50
            .collection
                display flex
                flex-direction row
                &:hover
                    background-color gold
                div
                    cursor pointer
                    width 100%
        #dashboard
            width 85%
            #query_container
                height 80px
                margin-bottom 10px
                display flex
                flex-direction row
                border-bottom 1px solid #2C3E50
                div
                    align-self flex-end
                    margin 10px
            #table-container
                margin 10px

    #drp_filter
        option
            &:first-child
                color #D3D3D3

    #sort
        cursor pointer
        &:hover
            color dodgerblue

    .table-header
        font-weight 600
        border-bottom 1px solid #2C3E50

    .data-row
        padding-top 1px
        display flex
        flex-direction row
        div
            &:first-child
                width 40px
            &:nth-child(2) // ID
                width 100px
            &:nth-child(3) // User
                width 100px
            &:nth-child(4) // Attribute
                width 1200px

    .delete
        height 20px
        &:hover
            cursor pointer

    .selected-collection
        color red
        font-weight 600
</style>