<template>
    <div id="app">
        <!--<editor-fold desc="Collections Manager">-->
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
                        <img v-if="currentCollectionFocus === index"
                             class="delete"
                             src="./assets/images/delete.svg"
                             @click="deleteCollection(collection)"
                             alt="Delete"/>
                    </div>
                </div>

                <p v-if="collections.length === 0">You have no collections.</p>
            </div>
        </div>
        <!--</editor-fold>-->

        <!--<editor-fold desc="Dashboard">-->
        <div id="dashboard">
            <!--<editor-fold desc="Search & Filter Container">-->
            <div id="query_container">
                <img id="reset" src="./assets/images/reset.svg" @click="reset()" alt="Reset"/>

                <div id="search_container">
                    <label>
                        <input id="txt_search" type="text" placeholder="Search value"/>
                    </label>

                    <div class="btn-container">
                        <div id="btn_search" class="btn primary" @click="search()">Search</div>
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
                    </div>
                </div>
            </div>
            <!--</editor-fold>-->

            <!--<editor-fold desc="Table">-->
            <div id="table_container">
                <div id="table_title">
                    {{ tableTitle === null
                        ? 'All user records'
                        : isCollection
                            ? 'Collection: ' + tableTitle
                            : isResult
                                ? 'Results containing: \'' + tableTitle + '\''
                                : '' }}
                </div>

                <div id="table_header" class="data-row">
                    <div>
                        <label>
                            <input v-if="!isCollection" id="chk_select_all" type="checkbox" @change="selectAll()"/>
                        </label>
                    </div>

                    <div>ID</div>

                    <div class="data-row-user">
                        <div>NAME</div>
                        <img id="sort" :src="iconSort" @click="sort()" alt="Sort">
                    </div>

                    <div>ATTRIBUTES</div>
                </div>

                <!--<editor-fold desc="Table Rows">-->
                <div id="data_container" :class="{ 'show-collection-drawer': selectedRecords.length !== 0 }">
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

                        <div @mousedown="editUser(index)" @mouseup="focusEditUser(index)">
                            <span v-show="selectedUserIndex !== index">{{ user.name }}</span>

                            <label>
                                <input :id="'txt_' + index"
                                       v-show="selectedUserIndex === index"
                                       class="txt-data-row-user"
                                       type="text"
                                       v-model="user.name"
                                       @focusout="selectedUserIndex = null"/>
                            </label>
                        </div>

                        <div>
                            <span v-for="(attribute, index) in user.attributeIds">{{ mapAttributeId(attribute) }}<span v-if="user.attributeIds.length !== (index + 1)">, </span></span>
                        </div>
                    </div>

                    <p v-if="displayedRecords.length === 0">No records found.</p>
                </div>
                <!--</editor-fold>-->
            </div>
            <!--</editor-fold>-->

            <!--<editor-fold desc="Table Pagination">-->
            <div id="pagination">
                <span class="btn" @click="backToStart()">First</span>
                <span class="btn" @click="previous()">Prev</span>
                <span class="btn" @click="next()">Next</span>
                <span class="btn" @click="goToEnd()">Last</span>
                <span id="total_records">Total records: {{ totalRecords }}</span>
            </div>
            <!--</editor-fold>-->

            <!--<editor-fold desc="'Add New Or Add To Existing Collection' Component">-->
            <div id="add_modify_collection" v-if="selectedRecords.length !== 0">
                <div>
                    <template v-if="isNewCollection === null">
                        <span class="btn primary mr-5" @click="isNewCollection = true">ADD NEW</span>
                        <span v-if="collections.length !== 0"> OR
                            <span class="btn primary mr-5" @click="isNewCollection = false">ADD TO EXISTING</span>
                        </span>
                        COLLECTION?
                    </template>

                    <!--<editor-fold desc="Add New Collection">-->
                    <template v-if="isNewCollection">
                        <label>
                            <input id="txt_collection_name" type="text" placeholder="Collection name"/>
                        </label>

                        <span id="btn_create_new_collection" class="btn primary" @click="addToNewCollection()">Add</span>
                        <span class="btn secondary" @click="isNewCollection = null">Cancel</span>
                    </template>
                    <!--</editor-fold>-->

                    <!--<editor-fold desc="Add To Existing Collection">-->
                    <template v-if="isNewCollection !== null && !isNewCollection">
                        <template v-if="collections.length !== 0">
                            <label for="drp_collections">
                                <select id="drp_collections">
                                    <template v-for="(collection, index) in collections">
                                        <option :value="index">{{ collection.name }}</option>
                                    </template>
                                </select>
                            </label>

                            <span class="btn primary" @click="addToExistingCollection()">Add</span>
                            <span class="btn secondary" @click="isNewCollection = null">Cancel</span>
                        </template>

                        <template v-else>
                            <p>You have no saved collections.</p>
                        </template>
                    </template>
                    <!--</editor-fold>-->
                </div>
            </div>
            <!--</editor-fold>-->
        </div>
        <!--</editor-fold>-->
    </div>
</template>

<script>
    import $ from 'jquery'

    export default {
        name: 'App',

        data: () => {
            return {
                totalRecords: 0,

                ascending: true, // By default ascending

                currentCollectionFocus: null,
                selectedCollection: null,
                tableTitle: null,
                isCollection: false,
                collection: [],
                collections: [],

                isResult: false,
                results: [],

                selectedRecords: [],
                selectedUserIndex: null,
                isNewCollection: null,

                currentPage: 0,
                numberOfRecordsPerPage: 100,

                attributes: [],
                users: [],
                attributeIds: []
            }
        },

        mounted() {

            this.processData();
            // The state is not persisted, therefore stop the user before the page is refreshed to keep changes that were made.
            window.onbeforeunload = () => { return 'Changes you made may not be saved.' };
        },

        computed: {
            // The records that are currently displayed on the screen
            displayedRecords() {

                let records = this.isResult
                    ? this.results
                    : this.isCollection
                        ? this.collection
                        : this.users;

                this.totalRecords = records.length;

                return records.slice(this.currentPage, this.currentPage + this.numberOfRecordsPerPage);
            },
            iconSort() {
                return this.ascending ? require('./assets/images/up.svg') : require('./assets/images/down.svg');
            }
        },

        methods: {
            editUser(index) {
                this.selectedUserIndex = index;
            },
            focusEditUser(index) {
                document.getElementById('txt_' + index).focus();
            },
            setCurrentCollectionFocus(index) {
                this.currentCollectionFocus = index;
            },
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
                const records = this.isResult
                    ? this.results
                    : this.isCollection
                        ? this.collection
                        : this.users;
                if (records.length <= (this.currentPage + parseInt(this.numberOfRecordsPerPage))) return;
                this.currentPage += parseInt(this.numberOfRecordsPerPage);
            },
            goToEnd() {
                const records = this.isResult
                    ? this.results
                    : this.isCollection
                        ? this.collection
                        : this.users;
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
                if (!this.ascending) this.sort();
            },
            /**
             *
             */
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
                //todo: add to clearSomething
                this.toggleCollection(false);
                this.selectedCollection = null;
                this.selectedRecords = [];

                const searchValue = document.getElementById('txt_search').value;
                const regex = ''.replace(new RegExp('', 'g'), searchValue);

                this.tableTitle = searchValue;
                let searchResults = this.users.filter(user => {
                    if (user.name.match(searchValue) !== null) return user.name.match(searchValue);
                    let attributeResults = user.attributeIds.filter(id => {
                        return this.mapAttributeId(id).toLowerCase().match(regex.toLowerCase())
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
                //todo: add to clearSomething
                this.toggleCollection(false);
                this.selectedCollection = null;
                this.selectedRecords = [];

                const filter = document.getElementById('drp_filter');
                const selectedFilterOption = filter.value;
                let filteredResults = [];

                this.tableTitle = filter.options[filter.selectedIndex].text;
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
                if (document.getElementById('chk_select_all') !== null) document.getElementById('chk_select_all').checked = false;
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
            selectAll() {
                this.selectedRecords = document.getElementById('chk_select_all').checked
                    ? this.isResult ? this.results : this.users
                    : [];
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

                const name = document.getElementById('txt_collection_name').value;

                this.collections.push({ id: Date.now(), name, records: this.selectedRecords });
                this.clearCheckboxes();
                this.isNewCollection = null;
                document.getElementById('txt_collection_name').value = '';
            },
            /**
             *
             */
            addToExistingCollection() {

                let collectionIndex = document.getElementById('drp_collections').value;

                this.selectedRecords.forEach(selectedRecord => {
                    const result = this.collections[collectionIndex].records.find(record => { return selectedRecord.id === record.id; });
                    if (result === undefined) {
                        this.collections[collectionIndex].records.push(selectedRecord);
                        this.isNewCollection = null;
                        this.clearCheckboxes();
                    } else {
                        alert('Item already exists in collection.');
                    }
                });
            },
            viewCollection(collection) {
                this.toggleResults(false);
                this.clearSearch();
                this.clearFilter();
                this.selectedRecords = [];
                this.toggleCollection();
                this.selectedCollection = collection.id;
                this.tableTitle = collection.name;
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

<style src="./assets/css/app.styl" lang="stylus"/>