---
layout: documentation
title: How To Use Propel i18n Behavior With Symfony 1.4
---

# How To Use Propel i18n Behavior With Symfony 1.4 #

`Propel` i18n behavior is now fully integrated with `Symfony 1.4`.
 
All you have to do is to write your `schema.xml` with the i18n `<behavior>` tag, instead of using the old SfPropelBehaviorI18n style `<table is18n=true>` with a `culture` column.

First init a `Symfony` project with `Propel` as default ORM and let's start with this `schema.xml`  

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<database defaultIdMethod="native" name="propel">
  <table name="author">
    <column name="id" type="INTEGER" primaryKey="true" required="true"/>
    <column name="name" type="VARCHAR" size="256"/>
  </table>
  <table name="book">
    <behavior name="i18n">
      <parameter name="i18n_columns" value="title, description"/>
      <parameter name="locale_alias" value="culture"/>
    </behavior>
    <column name="id" type="INTEGER" primaryKey="true" required="true"/>
    <column name="author_id" type="INTEGER" required="true"/>
    <column name="ISBN" type="VARCHAR" size="13"/>
    <foreign-key foreignTable="author">
      <reference local="author_id" foreign="id"/>
    </foreign-key>
  </table>
  <table name="book_i18n">
    <column name="id" type="INTEGER" primaryKey="true" required="true"/>
    <column name="title" type="VARCHAR" size="45"/>
    <column name="description" type="VARCHAR" size="45"/>
    <foreign-key foreignTable="book">
      <reference local="id" foreign="id"/>
    </foreign-key>
  </table>
</database>
{% endhighlight %}

and those fixtures

{% highlight yml %}
Author:
  bach:
    id: 1
    name: Richard Bach

Book:
  livingston:
    id: 1
    author_id: bach
    ISBN: 0-380-01286-3     
  illusions:
    id: 2
    author_id: bach
    ISBN: 0-440-20488-7

BookI18n:
  livingston_fr:
    id: livingston
    locale: fr
    title: Jonathan Livingston le goéland
  livingston_en: 
    id: livingston
    locale: en
    title: Jonathan Livingston Seagull
  illusions_fr:
    id: illusions
    locale: fr
    title: Le Messie récalcitrant
  illusions_en: 
    id: illusions
    locale: en
    title: Jonathan Livingston Seagull
{% endhighlight %}

let's build this schema

{% highlight bash %}
php symfony propel:build --all --and-load --no-confirmation
{% endhighlight %}

## Simple Use Of embedI18n() ## 

create a book module

{% highlight bash %}
php symfony propel:generate-module main book Book
{% endhighlight %}

add i18N to book form `lib/form/BookForm.class.php`   

{% highlight php %}
<?php
class BookForm extends BaseBookForm
{
  public function configure()
  {
    $this->embedI18n(array('fr','en'));
  }
}
{% endhighlight %}

let's print the form with the i18n embedded form in `apps/main/modules/book/templates/_form.php`

{% highlight php %}
<?php use_stylesheets_for_form($form) ?>
<?php use_javascripts_for_form($form) ?>

<form action="<?php echo url_for('book/'.($form->getObject()->isNew() ? 'create' : 'update').(!$form->getObject()->isNew() ? '?id='.$form->getObject()->getId() : '')) ?>" method="post" <?php $form->isMultipart() and print 'enctype="multipart/form-data" ' ?>>
<?php if (!$form->getObject()->isNew()): ?>
<input type="hidden" name="sf_method" value="put" />
<?php endif; ?>
  <table>
    <tfoot>
      <tr>
        <td colspan="2">
          <?php echo $form->renderHiddenFields(false) ?>
          &nbsp;<a href="<?php echo url_for('book/index') ?>">Back to list</a>
          <?php if (!$form->getObject()->isNew()): ?>
            &nbsp;<?php echo link_to('Delete', 'book/delete?id='.$form->getObject()->getId(), array('method' => 'delete', 'confirm' => 'Are you sure?')) ?>
          <?php endif; ?>
          <input type="submit" value="Save" />
        </td>
      </tr>
    </tfoot>
    <tbody>
      <?php echo $form->renderGlobalErrors() ?>
      <?php echo $form ?>
    </tbody>
  </table>
</form>
{% endhighlight %}

## Use embedI18n() In An Embedded Form ##

create an author module

{% highlight php %}
php symfony propel:generate-module main author Author
{% endhighlight %}

embed book form in author `lib/form/AuthorForm.class.php`

{% highlight php %}
<?php
class AuthorForm extends BaseAuthorForm
{
  public function configure()
  {
    $this->embedRelation('Book');
  }
}
{% endhighlight %}

finally let's print the form with all his embedded forms in `apps/main/modules/templates/_form.php`

{% highlight php %}
<?php use_stylesheets_for_form($form) ?>
<?php use_javascripts_for_form($form) ?>

<form action="<?php echo url_for('author/'.($form->getObject()->isNew() ? 'create' : 'update').(!$form->getObject()->isNew() ? '?id='.$form->getObject()->getId() : '')) ?>" method="post" <?php $form->isMultipart() and print 'enctype="multipart/form-data" ' ?>>
<?php if (!$form->getObject()->isNew()): ?>
<input type="hidden" name="sf_method" value="put" />
<?php endif; ?>
  <table>
    <tfoot>
      <tr>
        <td colspan="2">
          <?php echo $form->renderHiddenFields(false) ?>
          &nbsp;<a href="<?php echo url_for('author/index') ?>">Back to list</a>
          <?php if (!$form->getObject()->isNew()): ?>
            &nbsp;<?php echo link_to('Delete', 'author/delete?id='.$form->getObject()->getId(), array('method' => 'delete', 'confirm' => 'Are you sure?')) ?>
          <?php endif; ?>
          <input type="submit" value="Save" />
        </td>
      </tr>
    </tfoot>
    <tbody>
      <?php echo $form->renderGlobalErrors() ?>
      <?php echo $form ?>
    </tbody>
  </table>
</form>
{% endhighlight %}