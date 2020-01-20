<template>
    <div id="app">
        <!--<editor-fold desc="Collections Manager">-->
        <div id="collections_container">
            <div id="collections_manager-title">Collections Manager</div>

            <div id="collections">
                <div v-for="(collection, index) in collections"
                     class="collection"
                     :class="{ 'selected-collection': selectedCollection === collection.id }"
                     @mouseover="selectedCollectionFocus = index"
                     @mouseleave="selectedCollectionFocus = null">
                    <div @click="viewCollection(collection)">{{ collection.name }}</div>

                    <div>
                        <img v-if="selectedCollectionFocus === index"
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
                            <input v-if="!isCollection" id="chk_select_all" type="checkbox" @change="selectAll()" :checked="allChecked()"/>
                        </label>
                    </div>

                    <div>ID</div>

                    <div class="data-row-user">
                        <div>NAME</div>
                        <img id="sort" :src="iconSortOrder" @click="sort()" alt="Sort">
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
                                <input :id="'chk_' + user.id" type="checkbox" @change="setSelectedRecord(user)" :checked="selectedRecordChecked(user)"/>
                            </label>
                        </div>

                        <div v-else>
                            <img class="delete" src="./assets/images/delete.svg" @click="deleteRecord(index)" alt="Delete"/>
                        </div>

                        <div>{{ user.id }}</div>

                        <div @mousedown="editUser(index)" @mouseup="setEditUserFocus(index)">
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

                        <span id="btn_create_new_collection" class="btn primary add" @click="addToNewCollection()">Add</span>
                        <span class="btn secondary clear" @click="isNewCollection = null">Cancel</span>
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

                            <span class="btn primary add" @click="addToExistingCollection()">Add</span>
                            <span class="btn secondary clear" @click="isNewCollection = null">Cancel</span>
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
                numberOfRecordsPerPage: 100,
                currentPage: 0,

                ascending: true, // By default ascending

                tableTitle: null,

                attributes: [],

                users: [],
                attributeIds: [],

                isNewCollection: null, // Shows/hides 'Add New Or Add To Existing Collection' component
                selectedCollection: null, // Set active class for the selected collection
                selectedCollectionFocus: null, // Show/hide 'delete' function for the collection currently hovered over

                isCollection: false, // True if a user is viewing results from a collection
                collections: [], // Contains all saved collection
                collection: [], // Contains the content of the selected collection

                isResult: false, // True if a user is viewing results from a Search or Filter
                results: [], // Contains the results of a Search or Filter

                selectedRecords: [],

                selectedUserIndex: null // Index of the user selected to edit name
            }
        },

        mounted() {
            this.processData();
            // The state is not persisted, therefore stop the user before the page is refreshed to keep changes that were made.
            window.onbeforeunload = () => { return 'Changes you made may not be saved.' };
        },

        computed: {
            displayedRecords() {

                let records = this.getRecords();

                this.totalRecords = records.length;
                // Get the first 100 records
                return records.slice(this.currentPage, this.currentPage + this.numberOfRecordsPerPage);
            },
            // Return up or down arrow image based on the state of the current sort order
            iconSortOrder() {
                return this.ascending ? require('./assets/images/up.svg') : require('./assets/images/down.svg');
            }
        },

        methods: {
            //<editor-fold desc="Data Processing">
            /**
             * Gets the data from the data file and process it into a defined structure.
             */
            async processData() {
                // Get the data from the data file
                await $.get('anonymous-msweb.data', data => {

                    let dataLines = data.split('\n');

                    // Process Attributes (vroots) and User data (Case and vote lines)
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
                                // Assign the last batch of attributes for the last user added
                                this.setAttributesForUser();
                                break;
                            default:
                                break;
                        }
                    });
                });
            },
            /**
             * Construct an object containing an attribute's data
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
             * Constructs an object containing a user's data.
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
             * Compiles the group of attributes related to a user.
             * @param data
             */
            setAttributeIdForUser(data) {
                this.attributeIds.push(data[1])
            },
            /**
             * Sets the attribute IDs related to the last user that was added.
             */
            setAttributesForUser() {
                this.users[this.users.length - 1].attributeIds = this.attributeIds;
            },
            //</editor-fold>

            //<editor-fold desc="Search & Filter">
            /**
             * Resets all relevant fields and the table to it's initial state.
             */
            reset() {

                this.currentPage = 0;
                this.tableTitle = null;

                this.clearSearch();
                this.clearFilter();
                this.clearCheckboxes();

                this.selectedCollection = null;
                this.toggleCollection(false);

                if (!this.ascending) this.sort(); // Sorts table contents to it's default order
            },
            /**
             * Display all records that match the search criteria.
             */
            search() {

                // Reset view
                this.clearFilter();
                this.clearCollection();

                const searchValue = document.getElementById('txt_search').value;
                const searchValueRegex = ''.replace(new RegExp('', 'g'), searchValue);

                if (searchValue === '') return alert('Enter a search value.');

                this.tableTitle = searchValue;

                // Filter all records that match the search value
                let searchResults = this.users.filter(user => {

                    // Return users that match the search value
                    if (user.name.match(searchValue) !== null) return user.name.match(searchValue);

                    // Return attributes that match the search value
                    let attributeResults = user.attributeIds.filter(id => {
                        return this.mapAttributeId(id).toLowerCase().match(searchValueRegex.toLowerCase())
                    });

                    if (attributeResults.length !== 0) return user; //todo: is this needed? users are already returned as well as attributes
                });

                this.toggleResults(); // Enable result view
                searchResults.forEach(result => this.results.push(result));
            },
            /*
            * Clears Search related fields.
            */
            clearSearch() {
                this.toggleResults(false);
                document.getElementById('txt_search').value = '';
            },
            /**
             * Display all records that contains the selected filter.
             */
            applyFilter() {

                // Reset view
                this.clearSearch();
                this.clearCollection();

                const filter = document.getElementById('drp_filter');
                const selectedFilter = filter.options[filter.selectedIndex].text;
                let filteredResults = [];

                if (selectedFilter === 'Select filter') return alert('Select a filter option.');

                this.tableTitle = selectedFilter;

                this.users.forEach(user => {
                    let result = user.attributeIds.filter(attr => { return attr === filter.value });
                    if (result.length !== 0) filteredResults.push(user);
                });

                this.toggleResults(); // Enable result view
                filteredResults.forEach(result => this.results.push(result));
            },
            /**
             * Clears Filter related fields.
             */
            clearFilter() {
                this.toggleResults(false);
                document.getElementById('drp_filter').value = '';
            },
            //</editor-fold>

            //<editor-fold desc="Data Table">
            /**
             * Sorts the displayed records by the user name field.
             */
            sort() {

                let records = this.getRecords();
                this.ascending = !this.ascending;

                records.sort((a, b) => {
                    if (this.ascending ? a.name < b.name : a.name > b.name) return -1;
                    if (this.ascending ? a.name > b.name : a.name < b.name) return 2;
                    return 0;
                });
            },
            /**
             * Sets the index of the selected record to enable the user to edit a user's name field.
             */
            editUser(index) {
                this.selectedUserIndex = index;
            },
            /**
             * Forces focus to the selected user field.
             */
            setEditUserFocus(index) {
                document.getElementById('txt_' + index).focus();
            },
            /**
             *
             */
            allChecked() {
                if (this.selectedRecords.length === 0) return false;
                return this.selectedRecords.length === this.totalRecords;
            },
            /**
             * Selects all records from the current list.
             */
            selectAll() {

                let records = document.getElementById('chk_select_all').checked
                    ? this.isResult ? JSON.stringify(this.results) : JSON.stringify(this.users)
                    : [];

                records.length !== 0 ? this.selectedRecords = JSON.parse(records) : [];
            },
            /**
             * Binds the checkbox state to it's records.
             */
            selectedRecordChecked(user) {
                return this.selectedRecords.find(record => { return record.id === user.id }) !== undefined;
            },
            /**
             * Add/removes the record that was selected/deselected to/from the list of selected records.
             */
            setSelectedRecord(user) {
                document.getElementById('chk_' + user.id).checked
                    ? this.selectedRecords.push(user) // Adds record
                    : this.selectedRecords.find((record, index) => {
                        if (record && record.id === user.id) this.selectedRecords.splice(index, 1); // Removes record
                    });
            },
            /**
             * Clears all checkboxes
             */
            clearCheckboxes() {
                this.selectedRecords = []; // Clears selected items
                if (document.getElementById('chk_select_all') !== null) document.getElementById('chk_select_all').checked = false;
            },
            /**
             * Enables/disables 'Result' view
             */
            toggleResults(isResult = true) {
                this.results = [];
                this.currentPage = 0;
                this.isResult = isResult;
            },
            //</editor-fold>

            //<editor-fold desc="Pagination">
            backToStart() {
                this.currentPage = 0;
            },
            previous() {
                if (this.currentPage !== 0) this.currentPage -= parseInt(this.numberOfRecordsPerPage);
            },
            next() {
                const records = this.getRecords();
                // Do not execute if there are no further records to view
                if (records.length <= (this.currentPage + parseInt(this.numberOfRecordsPerPage))) return;
                this.currentPage += parseInt(this.numberOfRecordsPerPage);
            },
            goToEnd() {
                const records = this.getRecords();
                // Do not execute if there are less than 100 records
                if (records.length <= parseInt(this.numberOfRecordsPerPage)) return;
                this.currentPage = records.length - 1;
            },
            //</editor-fold>

            //<editor-fold desc="Collections Manager">
            /**
             * Adds selected records to a new collection.
             */
            addToNewCollection() {

                const name = document.getElementById('txt_collection_name').value;

                if (name === '') return alert('Enter a collection name.');

                // Use Date.now() to set a unique ID for the collection
                this.collections.push({ id: Date.now(), name, records: this.selectedRecords });
                document.getElementById('txt_collection_name').value = ''; // Empty collection name field
                this.closeCollectionDrawer();
            },
            /**
             * Adds selected records to a specified collection.
             */
            addToExistingCollection() {

                let collectionIndex = document.getElementById('drp_collections').value;

                this.selectedRecords.forEach(selectedRecord => {

                    const result = this.collections[collectionIndex].records.find(record => { return selectedRecord.id === record.id; });

                    if (result === undefined) {
                        this.collections[collectionIndex].records.push(selectedRecord);
                        this.closeCollectionDrawer();
                    } else {
                        alert('Item already exists in collection.');
                    }
                });
            },
            /**
             * Displays the records of the selected collection.
             */
            viewCollection(collection) {

                this.tableTitle = collection.name;
                this.selectedCollection = collection.id;
                this.selectedRecords = []; // Clear any selected item

                // Reset Search and Filter
                this.clearSearch();
                this.clearFilter();

                this.toggleResults(false); // Disable 'Results' view
                this.toggleCollection(); // Enable 'Collection' view

                collection.records.forEach(record => this.collection.push(record));
            },
            /**
             * Deletes a record from a selected collection.
             */
            deleteRecord(recordIndex) {

                // Return the index of the selected collection
                let collectionIndex = this.collections.findIndex(collection => {
                    return collection.id === this.selectedCollection;
                });

                this.collections[collectionIndex].records.find((record, index) => {
                    if (index === recordIndex) this.collections[collectionIndex].records.splice(index, 1); // Delete record
                });

                this.collection = this.collections[collectionIndex].records; // Display updated records
            },
            /**
             * Deletes the specified collection.
             */
            deleteCollection(selectedCollection) {

                this.reset(); // Reset view

                this.collections.find((collection, index) => {
                    if (selectedCollection && selectedCollection.id === collection.id) this.collections.splice(index, 1); // Delete collection
                });
            },
            /**
             * Close the 'drawer' where the 'Add New Or Add To Existing Collection' component lies.
             */
            closeCollectionDrawer() {
                this.isNewCollection = null;
                this.clearCheckboxes();
            },
            /**
             * Clears components related to collection.
             */
            clearCollection() {
                this.toggleCollection(false);
                this.selectedCollection = null;
                this.selectedRecords = [];
            },
            /**
             * Enables/disables 'Collection' view.
             */
            toggleCollection(isCollection = true) {
                this.collection = [];
                this.currentPage = 0;
                this.isCollection = isCollection;
            },
            //</editor-fold>

            /**
             * Returns the records related the the current view.
             */
            getRecords() {
                return this.isResult
                    ? this.results
                    : this.isCollection
                        ? this.collection
                        : this.users;
            },
            /**
             * Maps the attribute ID to the attribute title.
             * @param id
             * @returns string
             */
            mapAttributeId(id) {
                return this.attributes.find(attribute => { if (attribute.id === id) return attribute.title }).title;
            },
            /**
             * Function to clean the string value.
             * @param value
             * @returns string
             */
            cleanString(value) {
                return value.replace(/"/g, '');
            }
        }
    }
</script>

<style src="./assets/css/app.styl" lang="stylus"/>