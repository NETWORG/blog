---
author: Ondrej Juda
title: Selection and filtering in Fluent UI DetailsList
date: '2023-02-02T10:00:00+0100'
categories:
  - Fluent UI
  - DetailsList
  - ReactJS
tags:
  - Fluent UI
  - DetailsList
  - ReactJS
---

Recently I got my hands on Fluent UI component called [DetailsList](https://developer.microsoft.com/en-us/fluentui#/controls/web/detailslist). It is pretty handy component to show tabular data. There are many options how you can cusomize rendering of the table, rows and even its cells.

![DetailsList](/uploads/2023/02/fluentui-detailslist-1.png)

What I needed to acomplish was to add text input above the table. User would write text into the field, the table would filter acordingly. Nothing complicated or what I thought at first.

Component proved me othervise after I tried to implement it. I wanted to get inspired in one of [examples](https://developer.microsoft.com/en-us/fluentui#/controls/web/detailslist/compact), but even though the filtering is there, the table doesn't keep previously selected rows on its own. I hope you will agree that it is not good ux. Imagine you select few rows, change filter to look for some specific row, and after you select another one and click a button to trigger some action, it will happen only for the displayed row, not the previously selected.

I started to look for some inspiration and after a while I managed to implement my own solution. If you want to jump into the code, feel free to click this link - [codesandbox.io](https://codesandbox.io/s/fluent-ui-detailslist-preserve-selection-when-filtering-hptj03), I will continue to explain it below.

To preserve selected items when they aren't displayed because of filter, you will need Selection imported from @fluentui/react and useRef hook from react library.

Selection is stored in state variable using useState hook. The useRef hook stores selected keys inside of selectedKeys ref variable.

You can notice two functions defined inside of Selection object - onSelectionChanged and onItemsChanged. Those functions are what makes this code work. OnSelectionChanged triggers when you click on a row and change Selection. OnItemsChanged triggers when items in Selection changes.

Inside of the first function we want to compare previously selected keys inside of ref selectedKeys and currently selected keys inside of Selection. Outcome of this compare is saved back to the selectedKeys ref.

OnItemsChanged is used to "reselect" rows displayed in the table. It iterates through the Selection and sets row selected if the row stored in selectedKeys.

The cycle works like this:

1. User selects a row.
2. OnSelectionChanged is triggered and row is stored in selectedKeys.
3. User inputs filter.
4. Table rows are filtered. Previously selected row is not displayed, but its key is stored in variable.
5. OnItemsChanged triggers because Selection got new array of items. Nothing happens since selected row is not present.
6. User select second row.
7. OnSelectionChanged is triggered. SelectedKeys variable fills with previously selected row and the new one.
8. User removes filter.
9. Table displays all rows. Both selected rows are displayed.
10. OnItemsChanged is triggered, iterates through Selection items and sets two rows as selected based on selectedKeys variable value.


![DetailsList selected](/uploads/2023/02/fluentui-detailslist-2.png)

![DetailsList selected and filtered](/uploads/2023/02/fluentui-detailslist-3.png)

You can freelely filter through table and keep previously selected rows stored inside of a variable for later usage thanks to this implementation.