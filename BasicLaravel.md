## Basic Laravel & CRUD

Creating, reading, updating, and deleting resources is used in pretty much every application. Laravel helps make the process easy using resource controllers. Resource Controllers can make life much easier and takes advantage of some cool Laravel routing techniques. Today, we’ll go through the steps necessary to get a fully functioning CRUD application using resource controllers.

### Getting our Database Ready

#### Student Migration
We need to set up a quick database so we can do all of our CRUD functionality. In the command line in the root directory of our Laravel application, let’s create a migration.

```bash
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
```bash
php artisan migrate
```
Our database now has a students table to house all of the students we CRUD (create, read, update, and delete). Read more about migrations at the [Laravel docs](https://laravel.com/docs/10.x/migrations).


### Eloquent Model for the students

Now that we have our database, let’s create a simple Eloquent model so that we can access the students in our database easily. You can read about [Eloquent ORM](https://laravel.com/docs/10.x/eloquent) and see how you can use it in your own applications.

In the app/models folder, let’s create a student.php model.

**app/models/Student.php**
```php
<?php

    class Student extends Eloquent
    {

    }
    
```

That’s it! Eloquent can handle the rest. By default, this model will link to our `students` table and we can access it later in our controllers.

### Creating the Controller

From the official Laravel docs, on [resource controllers](https://laravel.com/docs/10.x/controllers#resource-controllers), you can generate a resource controller using the artisan tool.

Let’s go ahead and do that. This is the easy part. From the command line in the root directory of your Laravel project, type:

`php artisan make:controller StudentController --resource` This will create our resource controller with all the methods we need.

**app/controllers/StudentController.php**

```php
<?php

class StudentController extends BaseController {

    /**
        * Display a listing of the resource.
        *
        * @return Response
        */
    public function index()
    {
        //
    }

    /**
        * Show the form for creating a new resource.
        *
        * @return Response
        */
    public function create()
    {
        //
    }

    /**
        * Store a newly created resource in storage.
        *
        * @return Response
        */
    public function store()
    {
        //
    }

    /**
        * Display the specified resource.
        *
        * @param  int  $id
        * @return Response
        */
    public function show($id)
    {
        //
    }

    /**
        * Show the form for editing the specified resource.
        *
        * @param  int  $id
        * @return Response
        */
    public function edit($id)
    {
        //
    }

    /**
        * Update the specified resource in storage.
        *
        * @param  int  $id
        * @return Response
        */
    public function update($id)
    {
        //
    }

    /**
        * Remove the specified resource from storage.
        *
        * @param  int  $id
        * @return Response
        */
    public function destroy($id)
    {
        //
    }

}

```

### Setting Up the Routes

Now that we have generated our controller, let’s make sure our application has the routes necessary to use it. This is the other easy part (they actually might all be easy parts). In your `routes.php` file, add this line:

**app/routes.php**
```php
<?php

    Route::resource('students', 'StudentController');

```

This will automatically assign many actions to that resource controller. Now if you, go to your browser and view your application at `example.com/students`, it will correspond to the proper method in your StudentController.


#### Actions Handled By the Controller

HTTP | Verb	Path (URL) | Action (Method) | Route Name
--- | --- | --- | ---
GET | /students | index | students.index
GET | /students/create | create | students.create
POST | /students | store | students.store
GET | /students/{id} | show | students.show
GET | /students/{id}/edit | edit | students.edit
PUT/PATCH | /students/{id} | update | students.update
DELETE | /students/{id} | destroy | students.destroy


Tip: From the command line, you can run `php artisan routes` to see all the routes associated with your application.

### The Views

Since only four of our routes are GET routes, we only need four views. In our `app/views` folder, let’s make those views now.


```bash
app
└───views
    └───students
        │    index.blade.php
        │    create.blade.php
        │    show.blade.php
        │    edit.blade.php

```

### Making It All Work Together

Now we have our migrations, database, and models, our controller and routes, and our views. Let’s make all these things work together to build our application. We are going to go through the methods created in the resource controller one by one and make it all work.

#### Controller Function index()

In this function, we will get all the students and pass them to the view.

**app/controllers/StudentController.php**
```php
<?php

...

    /**
        * Display a listing of the resource.
        *
        * @return Response
        */
    public function index()
    {
        // get all the students
        $students = Student::all();

        // load the view and pass the students
        return View::make('students.index')
            ->with('students', $students);
    }
    
...

```

#### The View app/views/students/index.blade.php

Now let’s create our view to loop over the students and display them in a table. We like using [Twitter Bootstrap](https://getbootstrap.com/) for our sites, so the table will use those classes.

```html
<!DOCTYPE html>
<html>
<head>
    <title>student App</title>
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap.min.css">
</head>
<body>
<div class="container">

<nav class="navbar navbar-inverse">
    <div class="navbar-header">
        <a class="navbar-brand" href="{{ URL::to('students') }}">student Alert</a>
    </div>
    <ul class="nav navbar-nav">
        <li><a href="{{ URL::to('students') }}">View All students</a></li>
        <li><a href="{{ URL::to('students/create') }}">Create a student</a>
    </ul>
</nav>

<h1>All the students</h1>

<!-- will be used to show any messages -->
@if (Session::has('message'))
    <div class="alert alert-info">{{ Session::get('message') }}</div>
@endif

<table class="table table-striped table-bordered">
    <thead>
        <tr>
            <td>ID</td>
            <td>Name</td>
            <td>Email</td>
            <td>Staudent Level</td>
            <td>Actions</td>
        </tr>
    </thead>
    <tbody>
    @foreach($students as $key => $value)
        <tr>
            <td>{{ $value->id }}</td>
            <td>{{ $value->name }}</td>
            <td>{{ $value->email }}</td>
            <td>{{ $value->student_level }}</td>

            <!-- we will also add show, edit, and delete buttons -->
            <td>

                <!-- delete the student (uses the destroy method DESTROY /students/{id} -->
                <!-- we will add this later since its a little more complicated than the other two buttons -->

                <!-- show the student (uses the show method found at GET /students/{id} -->
                <a class="btn btn-small btn-success" href="{{ URL::to('students/' . $value->id) }}">Show this student</a>

                <!-- edit this student (uses the edit method found at GET /students/{id}/edit -->
                <a class="btn btn-small btn-info" href="{{ URL::to('students/' . $value->id . '/edit') }}">Edit this student</a>

            </td>
        </tr>
    @endforeach
    </tbody>
</table>

</div>
</body>
</html>
```