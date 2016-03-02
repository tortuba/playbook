# Zero Downtime Deployment with Rails & Postgres

This Cheatsheet explains when it is safe to run deployments without downtime, and how to turn unsafe deployments into a succession of safe deployments in order to achieve zero downtime.

## Adding columns
  * Safe for readonly models
  * Safe when there are no constraints on the column
  * If there's a constraints, use 2 deploys:
    1. Create the column without constraint but with a before_save helper to make sure the constraint is enforced (see below), update all existing
    2. Add the constraint

Ex for #1 helper:

    before_save :enforce_constraint

    def enforce_constraint
      self.admin ||= false
    end

Notes:
  * See github issue [#12330](https://github.com/rails/rails/issues/12330)
  * Model.reset_column_information is not enough because it only affects the process running the migration.

## Removing columns

Use https://github.com/pedro/moargration

Almost safe, but still [some errors with joins](https://github.com/pedro/moargration/issues/1).

## Renaming columns

Not safe

If used in a search expression:

1. Add a column with the desired name and change your model to write data to both (but don't change any queries yet)
2. Populate the new column with data from the previous one, and update queries to refer to the new column name
3. Delete the old column

Otherwise:

1. Add a column & patch attribute reader (see below), migrate the data
2. Drop the previous column & remove the patch

During #1, we need to read from both columns, adding a fallback to the accessor:

    def new_name
      super || attributes["old_name"]
    end


## Creating tables

Safe

## Removing tables

Safe

## Creating indexes

  * Safe only for readonly models
  * Otherwise make sure you create indexes concurrently (see below)


    class AddIndexToAsksActive < ActiveRecord::Migration
      disable_ddl_transaction!

      def change
        add_index :asks, :active, algorithm: :concurrently
      end
    end

## Removing indexes

Safe

---

**References**

  * [About ZDD](http://blog.codeship.com/zero-downtime-deployment/)
  * [Rails ZDD](http://blog.codeship.com/rails-migrations-zero-downtime/) or [original article](http://pedro.herokuapp.com/past/2011/7/13/rails_migrations_with_no_downtime/)
  * [Async index](https://robots.thoughtbot.com/how-to-create-postgres-indexes-concurrently-in)
