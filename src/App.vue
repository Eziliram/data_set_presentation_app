<template>
    <div id="app">
        <div id="collections_container">
            <div id="collections_manager-title">Collections Manager</div>

            <div id="collections">
                <div v-for="(collection, index) in collections"
                     class="collection"
                     :class="{ 'selected-collection': selectedCollection === collection.id }"
                     @mouseover="setCurrentCollectionFocus(index)"
                     @mouseleave="currentCollectionFocus = null">
                    <div @click="viewCollection(collection)">{{ collection.name }}</div>
                    <div>
                        <img v-if="currentCollectionFocus === index" class="delete" src="./assets/images/delete.svg" @click="deleteCollection(collection)" alt="Delete"/>
                    </div>
                </div>

                <p v-if="collections.length === 0">You have no collections.</p>

<!--
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
-->
            </div>
        </div>

        <div id="dashboard">
            <div id="query_container">
                <img id="refresh" src="./assets/images/refresh.svg" @click="reset()" alt="Reset"/>

                <div id="search_container">
                    <label>
                        <input id="txt_search" type="text" placeholder="Search value"/>
                    </label>

                    <div class="btn-container">
                        <div id="btn_search" class="btn primary" @click="search()">Search</div>
<!--                        <div id="btn_clear" class="btn secondary" @click="clearSearch()">Clear</div>todo:should this be removed?-->
                    </div>
                </div>

                <div id="filter_container">
                    <label for="drp_filter">
                        <select id="drp_filter">
                            <option value="" disabled selected>Select filter</option>
                            <template v-for="attribute in attributes">
                                <option :value="attribute.id">{{ attribute.title }}</option>
                            </template>
                        </select>
                    </label>

                    <div class="btn-container">
                        <div id="btn_apply_filter" class="btn primary" @click="applyFilter()">Apply</div>
<!--                        <div id="btn_remove_filter" class="btn secondary" @click="clearFilter()">Clear</div>todo:should this be removed?-->
                    </div>
                </div>
            </div>

            <div id="table_container">
                <div id="table_title">{{ tableTitle === null ? 'All records' : 'Collection: ' + tableTitle }}</div>

                <div class="table-header data-row">
                    <div>
                        <label>
                            <!-- TODO: implement select all -->
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
                <div id="data_container">
                    <div v-if="index < numberOfRecordsPerPage"
                         v-for="(user, index) in displayedRecords"
                         class="data-row">
                        <div v-if="!isCollection">
                            <label>
                                <input :id="'chk_' + user.id" type="checkbox" @change="setSelectedRecord(user)" :checked="checked(user)"/>
                            </label>
                        </div>

                        <div v-else>
                            <img class="delete" src="./assets/images/delete.svg" @click="deleteRecord(index)" alt="Delete"/>
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
                </div>
            </div>

<!--            todo: set start = 1 and end = last page number -->
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
                ascending: true, // By default ascending

                currentCollectionFocus: null,
                selectedCollection: null,
                tableTitle: null,
                isCollection: false,
                collection: [],
                collections: [
                    { //TODO: Create a test collection?
                        id: 11,
                        name: 'Amind',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 112,
                        name: 'Groceriesfsgh',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 1122,
                        name: 'Chcikennds',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 11224,
                        name: '00001',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 112242,
                        name: 'Thinknings',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 1122425,
                        name: 'Random collection',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 11224253,
                        name: 'Insturiuomme',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 112242532,
                        name: 'Music',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 1122425327,
                        name: 'Pickadatetesdf',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 11224253272,
                        name: 'Restauransrntyt',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                    { //TODO: Create a test collection?
                        id: 109,
                        name: 'Bug-a-lug',
                        records: [
                            { id: 'test01', name: 'test01', attributeIds: [ '1069', '1275', '1154' ] },
                            { id: 'test02', name: 'test02', attributeIds: [ '1069', '1154' ] },
                            { id: 'test03', name: 'test03', attributeIds: [ '1154' ] },
                            { id: 'test04', name: 'test04', attributeIds: [ '1275', '1154', '1144', '1258' ] },
                        ]
                    },
                ],

                isResult: false,
                results: [],

                selectedRecords: [],
                selectedUserIndex: null, //todo: rename
                isNewCollection: true,

                currentPage: 0,
                numberOfRecordsPerPage: 100,

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
            setCurrentCollectionFocus(index) {
                this.currentCollectionFocus = index;
            },
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
                this.selectedCollection = null;
                this.tableTitle = null;
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

                this.clearFilter();

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

                this.clearSearch();

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
            viewCollection(collection) {
                this.selectedCollection = collection.id;
                this.tableTitle = collection.name;
                this.toggleCollection();
                collection.records.forEach(record => this.collection.push(record));
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
    primary = #4171A0
    secondary = #EFEFEF
    default = #2C3E50

    body
        margin 0

    #app
        font-family 'Roboto', sans-serif
        color default
        display flex
        flex-direction row
        #collections_container
            border-right 1px solid default
            width 15%
            max-height 100%
            #collections_manager-title
                background-color default
                color #FFF
                font-size 18px
                font-weight 500
                text-transform uppercase
                text-align center
                line-height 71px
                letter-spacing 1.5px
                height 71px
            #collections
                height calc(100vh - 92px)
                overflow auto
                font-size 17px
                .collection
                    display flex
                    flex-direction row
                    padding 0 5px 0 15px
                    &:hover
                        background-color #F4F6F7
                    .delete
                        height 20px
                        padding-top 5px
                        &:hover
                            cursor pointer
                    div
                        height 30px
                        line-height 30px
                        cursor pointer
                        &:first-child
                            width 90%
                        &:nth-child
                            width 10%
                .selected-collection
                    color primary
                    font-weight 600
                p
                    padding-left 10px
        #dashboard
            width 85%
            #query_container
                position fixed
                width 100%
                height 60px
                padding-bottom 10px
                display flex
                flex-direction row
                border-bottom 1px solid default
                #refresh
                    cursor pointer
                    border 2px solid transparent
                    height 24px
                    margin auto 10px 0
                    padding 1px
                    transition 200ms ease all
                    &:hover
                        transform rotate(360deg)
                        transition 100ms ease all
                #search_container
                    margin-right 20px
                #search_container
                #filter_container
                    height 50px
                    display flex
                    flex-direction row
                    label
                        align-self flex-end
                div
                    align-self flex-end
            #table_container
                position relative
                margin 71px 0 10px
                #table_title
                    font-size 17px
                    font-weight 600
                    height 50px
                    line-height 50px
                    padding-left 10px
                #data_container
                    height calc(100vh - 170px)
                    border-top 1px solid default
                    border-bottom 1px solid default
                    overflow-y scroll

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

    #pagination
        text-align center



    input[type=text]
    select
        font-size 14px
        border none
        border-bottom 2px solid primary
        width 250px
        margin-left 10px
        padding 10px 5px 5px 10px
        &:focus
            outline none !important


    .btn-container
        display flex
        flex-direction row

    .btn
        background-color transparent
        color primary
        font-family 'Roboto'
        font-size 14px
        font-weight 600
        text-transform uppercase
        text-align center
        cursor pointer
        border 1px solid transparent
        padding 7px 12px
        margin-left 10px
        height 15px
        width 70px

    .primary
        border 2px solid primary
        &:hover
            background-color primary
            color #FFF

    .secondary
        color default
        width 50px
        height 16px
        &:hover
            color #090C0F
</style>