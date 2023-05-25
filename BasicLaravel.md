## Basic Laravel & CRUD

Creating, reading, updating, and deleting resources is used in pretty much every application. Laravel helps make the process easy using resource controllers. Resource Controllers can make life much easier and takes advantage of some cool Laravel routing techniques. Today, we’ll go through the steps necessary to get a fully functioning CRUD application using resource controllers.

### Getting our Database Ready

#### Student Migration
We need to set up a quick database so we can do all of our CRUD functionality. In the command line in the root directory of our Laravel application, let’s create a migration.

```console
php artisan make:migration create_students_table --table=students --create
```

This will create our shark migration in `app/database/migrations`. Open up that file and let’s add name, email, and shark_level fields.

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
