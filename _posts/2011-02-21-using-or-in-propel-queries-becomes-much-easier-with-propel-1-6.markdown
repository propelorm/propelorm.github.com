---
layout: post
title: Using OR In Propel Queries Becomes Much Easier With Propel 1.6
published: true
---
<p>Combining two generated filters with a logical <code>OR</code> used to be impossible in Propel - the alternative was to use <code>orWhere()</code> or <code>combine()</code>, but that meant losing all the smart defaults of generated filters.</p>
<p>Propel 1.6 introduces a new method for Query objects: <code>_or()</code>. It just specifies that the next condition will be combined with a logical <code>OR</code> rather than an <code>AND</code>.</p>

```php
<?php
// Basic usage: _or() as a drop-in replacement for orWhere()
$books = BookQuery::create()
  ->where('Book.Title = ?', 'War And Peace')
  ->_or()
  ->where('Book.Title LIKE ?', 'War%')
  ->find();
// SELECT * FROM book
// WHERE book.TITLE = 'War And Peace' OR book.TITLE LIKE 'War%'

// _or() also works on generated filters:
$books = BookQuery::create()
  ->filterByTitle('War And Peace')
  ->_or()
  ->filterByTitle('War%')
  ->find();
// SELECT * FROM book
// WHERE book.TITLE = 'War And Peace' OR book.TITLE LIKE 'War%'

// _or() also works on embedded queries
$books = BookQuery::create()
  ->filterByTitle('War and Peace')
  ->_or()
  ->useAuthorQuery()
    ->filterByName('Leo Tolstoi')
  ->endUse()
  ->find();
// SELECT book.* from book
// INNER JOIN author ON book.AUTHOR_ID = author.ID
// WHERE book.TITLE = 'War and Peace' //    OR author.NAME = 'Leo Tolstoi'
```

<p>This new method is implemented in the <code>Criteria</code> class, so it also works for the old-style queries. And since <code>ModelCriteria::orWhere()</code> is a synonym for <code>-&gt;_or()-&gt;where()</code>, it is now deprecated.</p>
