## Basic Laravel & CRUD

Creating, reading, updating, and deleting resources is used in pretty much every application. Laravel helps make the process easy using resource controllers. Resource Controllers can make life much easier and takes advantage of some cool Laravel routing techniques. Today, we’ll go through the steps necessary to get a fully functioning CRUD application using resource controllers.

### Getting our Database Ready

#### Student Migration
We need to set up a quick database so we can do all of our CRUD functionality. In the command line in the root directory of our Laravel application, let’s create a migration.

```console
php artisan make:migration create_students_table --table=students --create
```

This will create our student migration in `app/database/migrations`. Open up that file and let’s add name, email, and student_level fields.


**app/database/migrations/####_##_##_######_create_students_table.php**
```php
<?php

use IlluminateDatabaseSchemaBlueprint;
use IlluminateDatabaseMigrationsMigration;

class CreatestudentTable extends Migration {

    /**
        * Run the migrations.
        *
        * @return void
        */
    public function up()
    {
        Schema::create('students', function(Blueprint $table)
        {
            $table->increments('id');

            $table->string('name', 255);
            $table->string('email', 255);
            $table->integer('grade');

            $table->timestamps();
        });
    }

    /**
        * Reverse the migrations.
        *
        * @return void
        */
    public function down()
    {
        Schema::drop('students');
    }

}
```

Now from the command line again, let’s run this migration. Make sure your database settings are good in `app/config/database`.php and then run:
```console
php artisan migrate
```
Our database now has a students table to house all of the students we CRUD (create, read, update, and delete). Read more about migrations at the [Laravel docs](https://laravel.com/docs/8.x/migrations).


### Eloquent Model for the students

Now that we have our database, let’s create a simple Eloquent model so that we can access the students in our database easily. You can read about Eloquent ORM and see how you can use it in your own applications.

In the app/models folder, let’s create a student.php model.

** app/models/Student.php
```php
<?php

    class Student extends Eloquent
    {

    }
```

