# GranadaORM

## Documentation

Please note this documentation is not yet complete.
We're working on it!
Most of the functionality is based on Idiorm/Paris, so their documentation can be used as a starting basis.
For starters we'll have some more complex examples here, and as features are added.

If you want to contribute to the documentation, please feel free to submit a pull request!

# Querying

Note: in the examples below, it shows the query as executed on the SQL database.
Internally it does use placeholders so you only need to escape data if you are sending your data inline in a raw query.

## By primary key

To load a model by its primary key, use the `find_one()` function, specifying the primary key:

	<?php
	$item = User::find_one($id);
	// SELECT * FROM user WHERE id=$id;
	echo $item->name;

## Loading a single record

To load a single record of a model, set the query parameters then use `find_one()` function to limit to one record:

	<?php
	$item = User::find_one();
	// SELECT * FROM user LIMIT 1;
	echo $item->name;

## Loading multiple records

Get a number of records from the database, in an iterable list, by using the `find_many()` function.

	<?php
	$items = User::find_many();
	// SELECT * FROM user;
	foreach ($items as $item) {
		echo $item->name;
	}

## Limiting results

Specify the number of results you want to load using the `limit()` and `offset()` functions:

	<?php
	$items = User::offset(15)
			->limit(5)
			->find_many();
	// SELECT * FROM user LIMIT 5 OFFSET 15;

## Filtering results

Add comparative filters to the query by using various functions.
As you can see they are by default all reducing (uses AND).

	<?php
	$items = User::where('class', 'Test')
			->where_gt('age', 5)
			->where_lt('age', 10)
			->where_gte('friends', 2)
			->where_lte('friends', 4)
			->where_not_equal('enabled', 1)
			->where_like('first_name', '%red%')
			->where_not_like('first_name', '%blue%')
			->where_null('date_completed')
			->where_not_null('date_commenced')
			->where_raw('id IN (SELECT user_id FROM class_enrolment WHERE class_id=?)', $class_id)
			->find_many();
	// SELECT * FROM `user` WHERE
	// `class` = 'Test'
	// AND `age` > 5
	// AND `age` < 10
	// AND `friends` >= 2
	// AND `friends` <= 4
	// AND `enabled` != 1
	// AND `first_name` LIKE "%red%"
	// AND `first_name` NOT LIKE "%blue%"
	// AND `date_completed` IS NULL
	// AND `date_commenced` IS NOT NULL
	// AND `id` IN (SELECT user_id FROM class_enrolment WHERE class_id=$class_id);

## Using OR in filters

Since the default is to reduce results by using AND's, we use the `where_any_is()` function to add a group of filters that are OR'd together.

A simple OR, shown mixed with an AND filter:

	<?php
	$items = User::where_any_is(
        array(
            array('name' => 'Joe'),
            array('name' => 'Fred'),
		))
		->where('enabled', 1)
		->find_many();
        // SELECT * FROM `user` WHERE (`name` = 'Joe' OR `name` = 'Fred' ) AND `enabled` = 1

An OR, with a non-default operator

	<?php
	$items = User::where_any_is(
        array(
            array('name' => 'Joe'),
            array('name' => 'Fred'),
		), '!=')
		->where('enabled', 1)
		->find_many();
        // SELECT * FROM `user` WHERE (`name` != 'Joe' OR `name` != 'Fred' ) AND `enabled` = 1

Adding some AND comparisons inside the OR

	<?php
	$items = User::where_any_is(
        array(
            array('name' => 'Joe'),
            array('name' => 'Fred', 'age' => 20),
		)
		)->find_many();
        // SELECT * FROM `user` WHERE (( `name` = 'Joe' ) OR ( `name` = 'Fred' AND `age` = '20' ))

Overriding the comparison for one data type:

	<?php
	$items = User::where_any_is(
        array(
            array('name' => 'Joe', 'age' => 10),
            array('name' => 'Fred', 'age' => 20),
		), array('age' => '>')
		)->find_many();
        // SELECT * FROM `widget` WHERE (( `name` = 'Joe' AND `age` > '10' ) OR ( `name` = 'Fred' AND `age` > '20' ))

Overriding the comparison for all data types:

	<?php
	$items = User::where_any_is(
        array(
            array('score' => '5', 'age' => 10),
            array('score' => '15', 'age' => 20),
		), '>')->find_many();
        // SELECT * FROM `widget` WHERE (( `score` > '5' AND `age` > '10' ) OR ( `score` > '15' AND `age` > '20' ))

You can use NULL values in comparisons:

	<?php
	$items = User::where_any_is(
        array(
            array('name' => 'Joe', 'age' => NULL),
            array('name' => NULL, 'age' => 20),
		))->find_many();
	// SELECT * FROM `user` WHERE (( `name` = 'Joe' AND `age` IS NULL ) OR ( `name` IS NULL AND `age` = '20' ))

They also work with the `!=` operator:

	<?php
	$items = User::where_any_is(
        array(
            array('name' => 'Joe', 'age' => NULL),
            array('name' => NULL, 'age' => 20),
		), '!=')
		->find_many();
        // SELECT * FROM `widget` WHERE (( `name` != 'Joe' AND `age` IS NOT NULL ) OR ( `name` IS NOT NULL AND `age` != '20' ))

Pass an array to convert it into an IN or NOT IN (depending on the operator):

	<?php
	$items = User::where_any_is(
        array(
            array('name' => 'Joe', 'age' => array(18, 19)),
            array('name' => array('Bob', 'Jack'), 'age' => 20),
		), array( 'age' => '!=')
		)->find_many();
        // SELECT * FROM `widget` WHERE (( `name` = 'Joe' AND `age` NOT IN ('18', '19') ) OR ( `name` IN ('Bob', 'Jack') AND `age` != '20' ))