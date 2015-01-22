<!--
.. title: UITableView: Designing Headers and Footers in Interface Builder
.. slug: uitableview-designing-headers-and-footers-in-interface-builder
.. date: 2015-01-22 22:17:35 UTC+05:00
.. tags: UITableView, UIKit, iOS, Programming
.. link:
.. description:
.. type: text
-->

1. In the outline view, drag a view to the very top of table view (before the first prototype cell).

![new view added before first prototype cell](https://dl.dropboxusercontent.com/u/6845322/ishaq.pk/UITableView-HeaderFooter-01.png)

2. It would appear above the first prototype cell in the designer, Design it as you please, add controls, set constraints, etc.

![new header view designed](https://dl.dropboxusercontent.com/u/6845322/ishaq.pk/UITableView-HeaderFooter-02.png)

3. Once you are done with the design, drag it completely out of the view hierarchy.

![header view outside the view hierarchy](https://dl.dropboxusercontent.com/u/6845322/ishaq.pk/UITableView-HeaderFooter-03.png)

4. assign it to an outlet and return it from `viewForHeaderInSection` or `viewForFooterInSection` as appropriate.

5. Repeat to design header views for all sections

If you need to modify a particular header view, just drag it back inside the table view, modify and drag it out again.

If you are designing header view for the table instead of a section, you can leave it before the first prototype, it would work without any code. Same goes for table footer view, except that footer view has to be inserted *after* the last prototype cell.
