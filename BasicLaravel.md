## Praktisi Mengajar Universitas Widyatama - Basic Laravel & CRUD

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
    ...

    use App\Http\Controllers\StudentController;

    Route::resource('students', StudentController::class);

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
        return view('students.index', ['students' => $students]);
    }
    


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
            <td>{{ $value->grade }}</td>

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

We can now show all of our students on a page. There won’t be any that show up currently since we haven’t created any or seeded our database with students. Let’s move on to the form to create a student.


#### Controller Function create()

In this function, we will show the form for creating a new student. This form will be processed by the `store()` method.

**app/controllers/StudentController.php**

```php

<?php

    /**
        * Show the form for creating a new resource.
        *
        * @return Response
        */
    public function create()
    {
        // load the create form (app/views/students/create.blade.php)
        return view('students.create');
    }


```


#### The View app/views/students/create.blade.php

**app/views/students/create.blade.php**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Student App</title>
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

<h1>Create a student</h1>

<!-- if there are creation errors, they will show here -->
{{ HTML::ul($errors->all()) }}

{{ Form::open(array('url' => 'students')) }}

    <div class="form-group">
        {{ Form::label('name', 'Name') }}
        {{ Form::text('name', '', array('class' => 'form-control')) }}
    </div>

    <div class="form-group">
        {{ Form::label('email', 'Email') }}
        {{ Form::email('email', '', array('class' => 'form-control')) }}
    </div>

    <div class="form-group">
        {{ Form::label('student_level', 'student Level') }}
        {{ Form::select('student_level', array('0' => 'Select a Level', '1' => 'Grade 1', '2' => 'Grade 2', '3' => 'Grade 3'), '', array('class' => 'form-control')) }}
    </div>

    {{ Form::submit('Create the student!', array('class' => 'btn btn-primary')) }}

{{ Form::close() }}

</div>
</body>
</html>

```

We will add the errors section above to show validation errors when we try to `store()` the resource. <$>[note] Tip: When using `{{ Form::open() }}`, Laravel will automatically create a hidden input field with a token to protect from cross-site request forgeries. Read more at the Laravel docs. <$>

We now have the form, but we need to have it do something when it the submit button gets pressed. We set this form’s `action` to be a POST to example.com/students. The resource controller will handle this and automatically route the request to the `store()` method. Let’s handle that now.

### Storing a Resource store()

To process the form, we’ll want to validate the inputs, send back error messages if they exist, authenticate against the database, and store the resource if all is good. Let’s dive in.

#### Controller Function store()

**app/controllers/StudentController.php**
```php

<?php

    /**
        * Store a newly created resource in storage.
        *
        * @return Response
        */
    public function store()
    {
        // validate
        // read more on validation at http://laravel.com/docs/validation
        $rules = array(
            'name'       => 'required',
            'email'      => 'required|email',
            'student_level' => 'required|numeric'
        );
        $validator = Validator::make($request->all(), $rules);

        // process the login
        if ($validator->fails()) {
            return Redirect::to('students/create')
                ->withErrors($validator)
                ->withInput();
        } else {
            // store
            $student = new Student;
            $student->name       = $request->get('name');
            $student->email      = $request->get('email');
            $student->grade = $request->get('student_level');
            $student->save();

            // redirect
            Session::flash('message', 'Successfully created student!');
            return Redirect::to('students');
        }
    }


```

If there are errors processing the form, we will redirect them back to the create form with those errors. We will add them in so the user can understand what went wrong. They will show up in the errors section we setup earlier.

Now you should be able to create a student and have them show up on the main page! Navigate to example.com/students and there they are. All that’s left is showing a single student, updating, and deleting.


#### Controller Function show()

**app/controllers/StudentController.php**
```php

<?php



    /**
        * Display the specified resource.
        *
        * @param  int  $id
        * @return Response
        */
    public function show($id)
    {
        // get the student
        $student = Student::find($id);

        // show the view and pass the student to it
        return view('students.show', ['student' => $student]);
    }



```

#### The View app/views/students/show.blade.php

**app/views/students/show.blade.php**
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

<h1>Showing {{ $student->name }}</h1>

    <div class="jumbotron text-center">
        <h2>{{ $student->name }}</h2>
        <p>
            <strong>Email:</strong> {{ $student->email }}<br>
            <strong>Level:</strong> {{ $student->student_level }}
        </p>
    </div>

</div>
</body>
</html>

```

### Editing a Resource edit()

To edit a student, we need to pull them from the database, show the creation form, but populate it with the selected student’s info. To make life easier, we will use form model binding. This allows us to pull info from a model and bind it to the input fields in a form. Just makes it easier to populate our edit form and you can imagine that when these forms start getting rather large this will make life much easier.


#### Controller Function edit()

**app/controllers/StudentController.php**
```php
<?php



    /**
        * Show the form for editing the specified resource.
        *
        * @param  int  $id
        * @return Response
        */
    public function edit($id)
    {
        // get the student
        $student = Student::find($id);

        // show the edit form and pass the student
        return View::make('students.edit')
            ->with('student', $student);
    }


```

#### The View app/views/students/edit.blade.php

**app/views/students/edit.blade.php**
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

<h1>Edit {{ $student->name }}</h1>

<!-- if there are creation errors, they will show here -->
{{ HTML::ul($errors->all()) }}

{{ Form::model($student, array('route' => array('students.update', $student->id), 'method' => 'PUT')) }}

    <div class="form-group">
        {{ Form::label('name', 'Name') }}
        {{ Form::text('name', null, array('class' => 'form-control')) }}
    </div>

    <div class="form-group">
        {{ Form::label('email', 'Email') }}
        {{ Form::email('email', null, array('class' => 'form-control')) }}
    </div>

    <div class="form-group">
        {{ Form::label('student_level', 'student Level') }}
        {{ Form::select('student_level', array('0' => 'Select a Level', '1' => 'Grade 1', '2' => 'Grade 2', '3' => 'Grade 3'), null, array('class' => 'form-control')) }}
    </div>

    {{ Form::submit('Edit the student!', array('class' => 'btn btn-primary')) }}

{{ Form::close() }}

</div>
</body>
</html>

```

Note that we have to pass a method of PUT so that Laravel knows how to route to the controller correctly.

### Updating a Resource update()

This controller method will process the edit form. It is very similar to store(). We will validate, update, and redirect.

#### Controller Function update()

**app/controllers/StudentController.php**

```php
<?php



    /**
        * Update the specified resource in storage.
        *
        * @param  int  $id
        * @return Response
        */
    public function update($id)
    {
        // validate
        // read more on validation at http://laravel.com/docs/validation
        $rules = array(
            'name'       => 'required',
            'email'      => 'required|email',
            'student_level' => 'required|numeric'
        );
        $validator = Validator::make(Input::all(), $rules);

        // process the login
        if ($validator->fails()) {
            return Redirect::to('students/' . $id . '/edit')
                ->withErrors($validator)
                ->withInput(Input::except('password'));
        } else {
            // store
            $student = Student::find($id);
            $student->name       = Input::get('name');
            $student->email      = Input::get('email');
            $student->student_level = Input::get('student_level');
            $student->save();

            // redirect
            Session::flash('message', 'Successfully updated student!');
            return Redirect::to('students');
        }
    }


```


### Deleting a Resource destroy()

The workflow for this is that a user would go to view all the students, see a delete button, click it to delete. Since we never created a delete button in our `app/views/students/index.blade.php`, we will create that now. We will also add a notification section to show a success message.

We have to send the request to our application using the DELETE HTTP verb, so we will create a form to do that since a button won’t do.


>Alert: The DELETE HTTP verb is used when accessing the students.destroy route. Since you can’t just create a button or form with the method DELETE, we will have to spoof it by creating a hidden input field in our delete form.

#### The View app/views/students/index.blade.php

**app/views/students/index.blade.php**
```html

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
                {{ Form::open(array('url' => 'students/' . $value->id, 'class' => 'pull-right')) }}
                    {{ Form::hidden('_method', 'DELETE') }}
                    {{ Form::submit('Delete this student', array('class' => 'btn btn-warning')) }}
                {{ Form::close() }}

                <!-- show the student (uses the show method found at GET /students/{id} -->
                <a class="btn btn-small btn-success" href="{{ URL::to('students/' . $value->id) }}">Show this student</a>

                <!-- edit this student (uses the edit method found at GET /students/{id}/edit -->
                <a class="btn btn-small btn-info" href="{{ URL::to('students/' . $value->id . '/edit') }}">Edit this student</a>

            </td>
        </tr>
    @endforeach
    

```

Now when we click that form submit button, Laravel will use the students.destroy route and we can process that in our controller.

#### Controller Function destroy()

**app/controllers/StudentController.php**
```php

<?php



    /**
        * Remove the specified resource from storage.
        *
        * @param  int  $id
        * @return Response
        */
    public function destroy($id)
    {
        // delete
        $student = Student::find($id);
        $student->delete();

        // redirect
        Session::flash('message', 'Successfully deleted the student!');
        return Redirect::to('students');
    }


```

