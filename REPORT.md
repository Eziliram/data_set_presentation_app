# Report

## Layout Design Of User Interface Components

The user interface consists of the following components:
- Sidebar: Collections Manager to view all saved collections
- Header: Search and Filter functionality to query user records
- Dashboard: A table to display user records and collections; table pagination and functionality to save to a new/existing collection

All components are fixed on the screen, except for the table component.

The table component displays 100 user records at a time where the user is able to scroll down to view records. The 
pagination is fixed at the bottom of the table which, if there are more than 100 records, allows the user to browse 
through the remainder of records.

The header, which contains the Search and Filter features, is fixed at the top of the page to be available to the user 
at all times for quick access to this functionality.

Collections are grouped in the Collections Manager sidebar component to allow for quick access to view, modify and/or delete collections. 

To avoid cluttering in the Collections Manager component, the component to save to a new/existing collection will
only become available after the user has selected at lease one user record.

Note: I have not used any UI Frameworks such as Vuetify/Bootstrap in order to showcase my CSS skills.

## Methods & Approach

- Drafted a layout design on paper
- Researched the user experience of the design by drafting user interactions to the design
- Researched and implemented the functionality of the drafted components
- Defined the final layout design on paper
- Styled components according to the final design
- Ran through user cases and fixed bugs
- Documentation and optimisation
- Published application on GitHub

## Problems Or Obstacles Encountered

I didn't really encounter any problems or obstacles. I was able to proceed with the task at a steady pace. I did however
had to put some thought into some components and functions.

#### Instance 1: What would be the best way to read and structure the data to be bound by the UI

As IE was not a concern for this application, I used jQuery (instead of pure JavaScript) to read the data and structured
the data in JSON format. Each line of data was marked by what it contained, so I checked for the first character, e.g. 
'A' or 'C' to determine in which array to store the data. Users were stored in an array together with an array of it's 
associated 'vroots' (from here on referred to as 'attributes'). Attributes were stored in a separate array to be used by
the User array to map each user's associated attribute IDs to their respective titles.

E.g.
```
attributes: [
    {
        id: '001',
        title: 'Attribute 1',
        url: '/url1'
    },
    {
        id: '002',
        title: 'Attribute 2',
        url: '/url2'
    }
]
```

```
users: [
    {
        id: '1',
        name: '1',
        attributeIds: ['001', '002']
    },
    {
        id: '2',
        name: 'Test',
        attributeIds: ['001']
    }
]
```

#### Instance 2: Applying certain actions/styles to a specific item in a list

All items are dynamically added to a list and in some instances actions needed to be applied to only the selected item.
E.g. Clicking on an item would result in all items in the list responding to the click action. This was easily solved by
targeting the index of the selected item. An example would be clicking on a user's name field to edit it. Another 
example is the 'delete' styling and functionality for Collections.

## Summary Of Time Spend On Task

- Functionality: 1.5 days
- Styling and UX: 1 day
- Documentation: 0.5 day
- Testing and optimisation: 

TOTAL: [x] days

## If I had more time, I would have:

- implemented field validations and error messages
- added an additional component to view the details of the Attributes, e.g. IDs and URLs
- implemented a feature to hide/show the Collections Manager to allow for more space on the dashboard to view the records
- persisted the state (if the page is refreshed, all data will be reset back to their defaults)
- moved sections out to their own components

## References

- https://yarnpkg.com/en/docs
- https://vuejs.org/v2/api/
- https://cli.vuejs.org/
- https://developer.mozilla.org/en-US/docs/Web/JavaScript
- https://api.jquery.com/
- http://stylus-lang.com/
- https://css-tricks.com/snippets/css/a-guide-to-flexbox/
- https://material.io/resources/icons/?style=baseline
- https://fonts.google.com/